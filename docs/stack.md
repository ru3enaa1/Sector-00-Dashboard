# El stack de SECTOR 00 — qué usa, por qué, y en qué se diferencia de PFM

Documento de referencia. Explica cada pieza del stack, qué gana y qué cuesta la
versión concreta que está instalada, y por qué se eligió frente a lo que hace el
mismo trabajo en **PFM** (Java 17 · Spring Boot 3.5.16 · MySQL 8 · Thymeleaf).

No es una lista de ventajas. Donde el stack de PFM es mejor, lo dice.

---

## 0. La decisión que manda sobre todas las demás

Antes de comparar nada hay que fijar la diferencia real entre los dos proyectos,
porque **casi todo lo que sigue se deduce de ella**:

| | PFM | SECTOR 00 |
|---|---|---|
| Quién lo usa | Varias personas, con cuenta y contraseña | **Una persona** |
| Dónde corre | Un servidor, accesible por red | **Un portátil**, `localhost` |
| Cuándo | Siempre encendido | Cuando lo abres |
| Qué protege | Datos de terceros, obligaciones fiscales | Tu propia watchlist |

Comparar los dos stacks sin esto es comparar un camión con una moto. **PFM tiene
que resolver problemas que en SECTOR 00 no existen** — concurrencia, sesiones,
autenticación, despliegue, copias en caliente — y esos problemas son justamente
los que justifican Spring.

Al revés también: SECTOR 00 tiene un requisito que PFM no tiene —
**arrancar en dos segundos y no depender de nada instalado** — y eso es lo que
descarta Spring aquí.

---

## 1. Lenguaje y ejecución

### Node 24 + TypeScript 7.0.2

**Qué hace.** TypeScript añade tipos a JavaScript; se comprueban al compilar y
desaparecen al ejecutar. Node ejecuta el resultado.

**El pro que importa aquí: un solo lenguaje de punta a punta.** El mismo archivo
de contratos lo lee el servidor y el navegador. En PFM hay Java en el servidor y
JavaScript en el navegador, y todo lo que cruza esa frontera se escribe dos
veces.

**Los contras, y son reales:**

- **TypeScript 7 es muy nuevo y ya nos mordió.** Eliminó `baseUrl` de
  `tsconfig.json`. Las rutas de `paths` tuvieron que reescribirse con `./`, y el
  error no era obvio. Está apuntado en `CLAUDE.md` §6 porque volvería a pasar.
- **Los tipos no existen en ejecución.** Esta es la diferencia conceptual más
  grande con Java y hay que tenerla clara para explicarla:

  ```ts
  // TypeScript: esto compila y revienta en ejecución si el JSON no encaja
  const p = await respuesta.json() as Proyecto;
  ```

  En Java, `ObjectMapper.readValue(json, Proyecto.class)` falla ahí mismo porque
  la clase existe en tiempo de ejecución. En TypeScript, `as` es una promesa que
  le haces al compilador y nadie comprueba. **Por eso este proyecto valida todo
  lo que entra con Zod** (§4): no es paranoia, es tapar el agujero que Java no
  tiene.
- **Node no tiene `BigDecimal`.** Ver §6.

---

## 2. La base de datos — el pro que ya habías visto

Tu observación: *«el manejo, estructura y administración de la base de datos sin
necesidad de ser algo completamente aparte y tener el conector desde el framework
como el Spring de Java»*. Es correcta y conviene desarrollarla, porque es el
punto más fuerte de esta comparación.

### Lo que hay en cada proyecto

| | PFM | SECTOR 00 |
|---|---|---|
| Motor | MySQL 8, **proceso aparte** | SQLite, **un archivo** |
| Conector | `mysql-connector-j` por JDBC | `better-sqlite3` 13.0.3, en el proceso |
| ORM | Spring Data JPA (Hibernate) | Drizzle ORM 0.45.2 |
| Para instalar | MySQL Server, usuario, contraseña, puerto 3306 | nada |
| Para respaldar | `mysqldump` | copiar un archivo |
| Para mirar dentro | Workbench / cliente | `pnpm db:studio` o cualquier visor |

### Por qué esto cambia tanto

**En PFM la base de datos es un servicio.** Está encendida aunque la aplicación
no lo esté, tiene su propio usuario y su propia contraseña, escucha en un puerto,
y si no está arriba la aplicación no arranca. Eso es *correcto* para PFM: varias
personas escriben a la vez y hace falta un árbitro.

**En SECTOR 00 la base de datos es un archivo: `data/sector00.db`.** No hay
servicio, no hay puerto, no hay credenciales que guardar. `better-sqlite3` abre
el archivo dentro del mismo proceso de Node — la consulta no viaja por un socket,
es una llamada a función.

```ts
// src/server/db.ts — la conexión entera
const sqlite = new Database(rutaArchivo);
sqlite.pragma("journal_mode = WAL");
sqlite.pragma("foreign_keys = ON");
```

Compáralo con lo que PFM necesita en `application.properties`: URL, usuario,
contraseña, dialecto, pool de conexiones. Aquí no hay `.env` para la base de
datos porque no hay nada que configurar.

### El contra, y es un techo de verdad

- **SQLite admite un solo escritor a la vez.** Con WAL las lecturas no se
  bloquean, pero dos escrituras simultáneas se serializan. Para una persona es
  irrelevante; para PFM sería inaceptable.
- **`better-sqlite3` es SÍNCRONO.** Cada consulta **bloquea el bucle de eventos**
  de Node mientras dura. En un servidor con cien peticiones a la vez esto sería
  un error grave de arquitectura. Aquí es una ventaja: sin `async`, las
  transacciones se escriben como código normal.

  ```ts
  // src/server/modules/pista/consultas.ts — la transaccion no es async
  return db.transaction((tx) => {
    const [fila] = tx.insert(proyecto).values({...}).returning({ id: proyecto.id }).all();
    tx.insert(mint).values({ proyectoId: fila.id, ... }).run();
    return fila.id;
  });
  ```

- **Es un módulo nativo**: se compila al instalar. De ahí la trampa de
  `CLAUDE.md` §6 — `pnpm install` falla desde Git Bash — y de que renombrar la
  carpeta rompa `node_modules`.

### Drizzle 0.45.2 frente a JPA / Hibernate

Aquí la comparación es más pareja de lo que parece, y hay un punto donde Drizzle
gana claramente y otro donde pierde.

**Dónde gana Drizzle: las migraciones son SQL que puedes leer.**

```sql
-- drizzle/0005_magical_rogue.sql, generado por `pnpm db:generate`
ALTER TABLE `proyecto` ADD `arte_publicado` integer DEFAULT false NOT NULL;
```

Una línea, versionada en git, revisable antes de aplicarla. El equivalente típico
en Spring es `spring.jpa.hibernate.ddl-auto=update`, que decide solo qué cambiar y
**no te enseña el SQL**. (PFM puede usar Flyway o Liquibase para lo mismo, pero
hay que añadirlos; en Drizzle es el camino por defecto.)

**Dónde gana Drizzle: las consultas son SQL con tipos, no un lenguaje aparte.**

```ts
db.select({ id: proyecto.id, nombre: proyecto.nombre })
  .from(proyecto)
  .innerJoin(mint, eq(mint.proyectoId, proyecto.id))
  .where(and(eq(proyecto.tipo, "MINT"), inArray(proyecto.estado, ESTADOS_EN_PISTA)))
  .all();
```

Si te equivocas en un nombre de columna, **no compila**. En JPQL —
`@Query("SELECT p FROM Proyecto p WHERE p.tipo = :tipo")` — el error sale al
arrancar, no al compilar, porque la consulta es una cadena de texto.

**Dónde pierde Drizzle: no tiene relaciones automáticas.** Un `@OneToMany` de JPA
te trae las piezas de un proyecto sin escribir el join. Drizzle no: lo escribes
tú. Es más código, y a cambio no existe el problema del *N+1* silencioso ni el de
las sesiones perezosas, que en JPA son dos de los fallos de rendimiento más
comunes.

**Dónde pierde Drizzle de verdad: la versión.** `0.45.2` es **pre-1.0**. La API
puede romper entre versiones menores, y ya causó un problema documentado en
`CLAUDE.md` §6: la documentación oficial tiene dos canales y el que sale primero
al buscar (`sqlite-new`) es de libSQL y del canal v1.0 RC, que **no** es el
nuestro. Spring Data JPA lleva más de una década estable. Eso es una ventaja
seria de PFM y no se puede maquillar.

---

## 3. El servidor: Fastify 5.12.3 frente a Spring Boot 3.5.16

**Qué hace Fastify.** Escucha en un puerto y enruta peticiones HTTP. Eso es todo.

```ts
app.get("/api/pista/acierto", async () => aciertoPorTier());
```

**Qué hace Spring Boot.** Enruta HTTP **y además** inyección de dependencias,
gestión del ciclo de vida de los beans, transacciones declarativas
(`@Transactional`), seguridad, configuración por perfiles, actuator, arranque de
plantillas, pool de conexiones…

### La comparación honesta

| | Spring Boot | Fastify |
|---|---|---|
| Arranque en frío | ~4–8 s | ~0,3 s |
| Inyección de dependencias | Sí, el contenedor | **No**, importas la función |
| Transacciones | `@Transactional` | `db.transaction(...)` a mano |
| Seguridad | Spring Security | **No hay** — no hace falta |
| Configuración | `application.properties` + perfiles | `--env-file`, y casi nada que configurar |
| Curva | Alta: hay que entender el contenedor | Baja: es una función que devuelve JSON |

**El contra de Fastify es real y no es "menos características":** es que todo lo
que Spring te da hecho, aquí o lo escribes o no lo tienes. En SECTOR 00 no lo
tenemos **porque no hace falta** — no hay usuarios, no hay sesiones, no hay
permisos. El día que hiciera falta (la «base espacial» del README), esa carencia
pasaría a ser un problema de verdad, y ahí Spring volvería a ser la respuesta
correcta.

**El pro que sí se nota a diario:** el arranque. `tsx watch` recompila y reinicia
el servidor en menos de un segundo al guardar. En PFM, cambiar una plantilla de
Thymeleaf **obliga a reiniciar la aplicación** — está anotado en las reglas del
proyecto, con la advertencia de que un Ctrl+F5 del navegador no basta.

---

## 4. Validación: Zod 4.5.4 — el pro más grande del stack

Éste es, junto con la base de datos, el argumento más fuerte a favor de este
stack. Y es el que mejor funciona en un pitch, porque se explica en dos frases.

### El problema en PFM

Un formulario se valida **dos veces, en dos lenguajes**:

```java
// Servidor: Java, con Bean Validation
public class TransaccionDTO {
    @NotBlank(message = "El concepto es obligatorio")
    private String concepto;

    @DecimalMin(value = "0.01", message = "El importe debe ser positivo")
    private BigDecimal importe;
}
```

```javascript
// Navegador: JavaScript, a mano, y otra vez
if (!concepto.trim()) mostrarError("El concepto es obligatorio");
if (importe <= 0) mostrarError("El importe debe ser positivo");
```

**Dos definiciones de lo mismo.** A los tres meses una cambia y la otra no, y el
fallo se nota cuando el servidor rechaza algo que el formulario dio por bueno.

### Cómo lo resuelve SECTOR 00

Una definición. La misma.

```ts
// src/shared/contratos/index.ts
export const apuntarInstantanea = z.object({
  seguidores: z.coerce
    .number({ error: () => t("How many followers", "Cuántos seguidores") })
    .int(t("A whole number", "Un número entero"))
    .min(0, t("It cannot be negative", "No puede ser negativo")),
});
```

```ts
// El servidor la usa como frontera
const revisado = esquemaInstantanea.safeParse(peticion.body);
if (!revisado.success) return respuesta.code(400).send({ ... });
```

```ts
// El navegador la usa para avisar antes de mandar
const formulario = useForm({ resolver: zodResolver(editarMint) });
```

**El mismo objeto. Un solo sitio donde cambiar una regla.** Y el tipo de
TypeScript se *deriva* del esquema con `z.infer`, así que tampoco hay que
mantener una interfaz aparte.

Es lo que Spring **no puede hacer**, y no por falta de calidad: las anotaciones de
Bean Validation viven en clases Java, y el navegador no ejecuta Java.

**El contra:** Zod valida en ejecución, así que cuesta ciclos en cada petición —
irrelevante aquí. Y **Zod 4 cambió la API respecto a la 3**: `z.string().email()`
pasó a `z.email()`, los mensajes de error se declaran distinto. Casi todo lo que
encuentres escrito en internet es de Zod 3 y no compila.

---

## 5. La interfaz: React 19.2.8 + Vite 8.2.2 frente a Thymeleaf

### La diferencia de fondo

**Thymeleaf renderiza en el servidor.** Pides una URL, el servidor devuelve HTML
ya montado. Sencillo, rápido de escribir, y **el estado vive en el servidor**.

**React renderiza en el navegador.** El servidor manda JSON y el navegador pinta.
Más complejo, y **el estado vive en el navegador**.

### Por qué React aquí

No por moda. Por esto:

- **El hangar es una escena interactiva**, no una página. Ocho islas
  seleccionables con el teclado, un mecánico que camina hasta la que eliges, una
  carta que gira tres veces en el aire al completar una ficha. Eso en Thymeleaf
  serían cientos de líneas de JavaScript suelto colgando de un HTML generado.
- **No hay recargas.** Abrir una isla, cerrarla, cambiar un tier — nada de eso
  pide una página nueva.

### Los contras

- **Una tabla cuesta más código.** Compara:

  ```html
  <!-- Thymeleaf: cuatro lineas -->
  <tr th:each="m : ${mints}">
    <td th:text="${m.nombre}"></td>
    <td th:text="${#numbers.formatDecimal(m.precio, 1, 'POINT', 2, 'COMMA')}"></td>
  </tr>
  ```

  ```tsx
  // React: mas ceremonia para lo mismo
  {mints.map((m) => (
    <tr key={m.id}>
      <td>{m.nombre}</td>
      <td>{m.precioMint === null ? <Guion /> : m.precioMint}</td>
    </tr>
  ))}
  ```

  Para una aplicación que es **sólo** tablas y formularios, Thymeleaf gana. PFM
  es en buena parte eso, y por eso la elección de PFM también es correcta.
- **Vite 8 y Vitest 5 son muy nuevos.** Menos respuestas en internet, plugins que
  aún no se han puesto al día.
- **Hay dos procesos**, no uno: el 5353 sirve la interfaz y el 5354 la API. En
  desarrollo hace falta un proxy para que el navegador vea un solo origen. Spring
  sirve las dos cosas desde el mismo puerto y eso es más simple.

  > Trampa comprobada, en `CLAUDE.md` §6: **Vite escucha en `::1` y Fastify en
  > `127.0.0.1`.** `localhost` resuelve a uno u otro según el sistema. En el
  > navegador no se nota porque todo pasa por el proxy; probando a mano, sí.

---

## 6. El dinero: `dnum` 2.17.0 — donde Java gana

**En Java esto ya viene resuelto:** `BigDecimal` está en la biblioteca estándar
desde 1997, es lo que usa PFM y es correcto.

**JavaScript no tiene equivalente.** Sus números son coma flotante de 64 bits, y
`0.1 + 0.2` da `0.30000000000000004`. En una cartera eso es un error contable que
además no avisa.

La solución aquí: los decimales **se guardan como texto** y se operan con `dnum`,
que por dentro usa `BigInt`.

```ts
// src/shared/contratos/index.ts
const compra = dn.from(pieza.precioPagado, DECIMALES);  // "0.05"
const venta  = dn.from(pieza.precioVenta,  DECIMALES);  // "0.4"
const diferencia = dn.sub(venta, compra);
const positivo = dn.gte(diferencia, dn.from(0, DECIMALES));
```

**El contra es doble y hay que decirlo:**

1. Es una **dependencia de terceros** para algo que Java trae de serie.
2. **No hay nada que te obligue a usarla.** Un `+` normal entre dos números
   compila igual de bien. En Java, sumar dos `BigDecimal` con `+` ni siquiera es
   sintaxis válida — el compilador te protege. Aquí la protección es una regla
   escrita en `CLAUDE.md` y la disciplina de aplicarla.

Éste es el punto donde el stack de PFM es objetivamente más seguro.

---

## 7. Datos en el navegador: TanStack Query 5.102.8

**En PFM no existe este problema.** El servidor renderiza el HTML con los datos
ya dentro; no hay nada que sincronizar.

**En React sí:** los datos del servidor están en el navegador, y hay que decidir
cuándo están viejos. TanStack Query es una caché con invalidación por clave.

```ts
export const clavePista = {
  todo: ["pista"] as const,
  lista: (vista: VistaPista) => ["pista", "lista", vista] as const,
  acierto: ["pista", "acierto"] as const,
};
```

Las claves son arrays para poder **invalidar por prefijo**: al guardar una
evaluación se invalida `["pista"]` entero y se refrescan la tabla, las cuentas y
la tasa de acierto de una vez. Es la pieza que hace que la interfaz no se quede
mostrando un número viejo.

**Es complejidad que Thymeleaf no necesita.** El precio de renderizar en el
navegador.

---

## 8. Herramientas

| Trabajo | PFM | SECTOR 00 | Nota |
|---|---|---|---|
| Dependencias | Maven | pnpm 12 | pnpm bloquea los scripts de instalación por defecto; se autorizan uno a uno en `pnpm-workspace.yaml`. Maven no tiene esa protección. |
| Formato y lint | Checkstyle / plugins | **Biome 2.5.12** | Uno solo hace lo de ESLint + Prettier, escrito en Rust. Formatea el repositorio entero en ~300 ms. Contra: `biome.json` **no admite comentarios**, y si no parsea cae a su configuración por defecto y reformatea todo — comprobado el 2026-09-11. |
| Pruebas | JUnit + Spring Boot Test | **Vitest 5** | 180 pruebas en ~3 s. `@SpringBootTest` levanta un contexto entero y tarda mucho más. |
| Compilación | `mvn package` → JAR | `vite build` → estáticos | |
| Imágenes | — | `sharp` 0.35.4 | Otro módulo nativo. |

---

## 9. Resumen para explicarlo en voz alta

Si hay que defender el stack en tres frases:

1. **Un solo esquema de validación para las dos orillas.** Zod valida el
   formulario en el navegador y el cuerpo de la petición en el servidor con el
   mismo objeto. Spring no puede: sus anotaciones viven en Java y el navegador no
   ejecuta Java.
2. **La base de datos es un archivo.** Nada que instalar, nada que configurar,
   respaldo = copiar. A cambio: un solo escritor, y por eso sólo vale para un
   tablero personal.
3. **Arranca en menos de un segundo y recarga sin reiniciar.** En un proyecto que
   se toca a diario, eso es la diferencia entre probar una idea y no probarla.

Y si preguntan por lo que **no** es mejor, hay tres respuestas honestas:

- **Drizzle es pre-1.0.** Spring Data JPA lleva una década estable.
- **`BigDecimal` protege; `dnum` sólo ayuda.** En Java el compilador te impide
  sumar dinero mal. Aquí eso es una regla escrita, no una barrera.
- **No hay contenedor de dependencias ni seguridad.** El día que esto tenga
  usuarios, Spring vuelve a ser la respuesta correcta.

---

## 10. Versiones instaladas

| Paquete | Versión | Papel |
|---|---|---|
| react / react-dom | 19.2.8 | Interfaz |
| typescript | 7.0.2 | Tipos |
| vite | 8.2.2 | Servidor de desarrollo y empaquetado |
| fastify | 5.12.3 | Servidor HTTP |
| drizzle-orm | 0.45.2 | ORM |
| better-sqlite3 | 13.0.3 | Conector SQLite |
| zod | 4.5.4 | Validación compartida |
| @tanstack/react-query | 5.102.8 | Caché de datos del servidor |
| react-hook-form | 7.87.0 | Formularios |
| dnum | 2.17.0 | Decimales exactos |
| @biomejs/biome | 2.5.12 | Formato y lint |
| vitest | 5.0.0 | Pruebas |
| sharp | 0.35.4 | Arte |

Comparación: PFM corre sobre **Java 17** y **Spring Boot 3.5.16**, con
`spring-boot-starter-web`, `-data-jpa`, `-thymeleaf`, `-validation`, `-security`
y `-mail`, MySQL por `mysql-connector-j`, y PDFBox 3.0.8 + POI 5.5.1 para
exportar.

---

*Las reglas vivas del proyecto están en `CLAUDE.md`, en la raíz. Si algo de aquí
se contradice con ella, manda ella.*
