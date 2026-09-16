<div align="center">

# SECTOR 00

**A crypto operations board that looks like a game and works like a tool.**

Mints, whitelists, wallets, airdrops and secondary plays — tracked on a hand-painted
pixel-art space station, one island per job. It runs entirely on your own machine.
Nothing leaves it.

[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-7-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Fastify](https://img.shields.io/badge/Fastify-5-000000?logo=fastify&logoColor=white)](https://fastify.dev/)
[![SQLite](https://img.shields.io/badge/SQLite-local-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Pruebas](https://img.shields.io/badge/pruebas-118-21E6C1)](#estado)
[![Local](https://img.shields.io/badge/datos-solo%20en%20tu%20disco-FFB13D)](#lo-que-no-se-negocia)
[![Licencia](https://img.shields.io/badge/licencia-propietaria-FF6A3D)](#licencia)

<br>

<!-- FALTA LA CAPTURA 01-hangar.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/01-hangar.png" width="900"
     alt="El hangar de SECTOR 00: ocho islas flotantes en una estación espacial en pixel art">
-->

</div>

---

> **Este repositorio es la vitrina del producto.** El código fuente vive en un repositorio
> privado; aquí están la descripción y las capturas.
>
> **Español más abajo.** La sección en español no es una traducción — entra más a fondo.

---

## What it replaces

A spreadsheet. The one where you track which mint is tomorrow, which wallet holds the
whitelist, what you paid and whether it was worth it. It works until it doesn't: a
spreadsheet can't tell you that a whitelist expires in four hours, can't hold two
whitelists of the same project on the same wallet, and can't stop you from typing
`0.1 + 0.2` and getting `0.30000000000000004` in a column that represents money.

But a spreadsheet is only half of what it replaces. **The other half is the noise.**

A Discord alpha group drops thirty projects a week. Maybe three are for you. The rest are
somebody else's taste, somebody else's thesis, somebody else's bag — and they arrive in the
same feed, at the same volume, with the same urgency. The filter that decides which of the
thirty is worth your wallet is yours, and it lives in your head, where it does you no good
at 2am.

SECTOR 00 is where that filter becomes a thing you can see. A project gets a **tier** the
moment you write it down — COOK, POTENTIAL, DEGEN, WATCH — and from then on the board shows
your list, ranked by your judgement, not the channel's. Something that turned out to be
nothing gets discarded **with the reason attached**, so six months later you know whether
you were right. The noise stays in Discord. What crosses over is what you decided was worth
it.

So: a spreadsheet rebuilt as an instrument, and a feed rebuilt as a shortlist.

## Why it feels like a game

Because the thing that kills a watchlist isn't forgetting to write in it. It's writing half
of it and never coming back.

So the form is a **collectible card that evolves as you fill it**. Two fields and the
project exists — that's the baby stage. Past halfway it evolves. Fill the whole sheet and
the card spins three times in the air, lands, and shows the épico art. You see one of those
and you know what it cost.

None of that is decoration bolted on. It is the answer to a real problem: incomplete
records.

<!-- FALTA LA CAPTURA 03-carta-epica.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/03-carta-epica.png" width="820"
     alt="La carta de colección al completar una ficha entera, en su arte épico">
<sub><b>La carta épica</b> · sale sola al completar la ficha entera. No hay botón que la invoque.</sub>
-->

## How it works

**The hangar** is the home screen — eight floating islands, one per job, navigable with the
mouse or the arrow keys. A mechanic walks over to whichever island you select and tells you
what it's for. Light pulses travel along the walkways toward whatever needs attention.

**Three doors for the same data**, because the cost of entry decides whether the data
exists at all:

| Door | When | Cost |
|---|---|---|
| **Console** (`Ctrl+K`) | A mint drops in Discord at 2am | One line: `Abstract Cats @abscats` → Enter |
| **The card wizard** | You sit down with time | One field at a time, each one explained |
| **Edit** | A detail changed | Every field at once, no ceremony |

<!-- FALTA LA CAPTURA 02-consola.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/02-consola.png" width="820"
     alt="La consola de Ctrl+K: una línea de texto crea un proyecto entero">
<sub><b>La consola</b> · el 70 % del uso real. Una línea, Enter, y el proyecto existe.</sub>
-->

### The eight islands

| | | |
|---|---|---|
| **02 · RACE STRIP** | Mints, in two views | **ON TRACK** is what is still ahead of you; **ARCHIVE** is what already closed. The line between them is the state, never the date: a mint whose day passed but that you never resolved is not finished, it is *pending*, and hiding it by age would bury a decision. |
| **03 · KEY VAULT** | Whitelists | A wallet can hold several keys for the same project; each phase can cost a different price. Addresses are stored twice: as typed, and normalised for comparison. |
| **04 · SIGNAL ARRAY** | Other people's wallets | **Public addresses and labels only.** Each one carries a required *why you follow it* and a manual hit/miss counter — there is no API that tells you whether someone was right. Wallets that overlap with your own list sort first. |
| **05 · FARM RIG** | The long shift | Measured in months and consistency. The whole island hangs on the **consistency light**: green while you keep your interval, amber the moment you slip, red at double. What is overdue goes on top. It is the only pressure the system applies, and it is enough. |
| **06 · WORKSHOP** | Links, by drawer | Answers one question: *where was that bridge?* Pinned ones sit in a quick-access bar. A link tied to a live airdrop lights up **⛏ FARMING**. |
| **07 · SALVAGE MARKET** | Secondary plays | Two questions no other island asks, both required: *why it interests you* and *until when*. The deadline does not close the position for you — it puts it at the top with the danger stripe lit, so you close it. |
| **08 · SHOWROOM / SALVAGE** | Pieces, held and sold | The result card, with its two frames and four cells: BUY · SELL · PnL · TIME. You never pick the frame — the number picks it. |
| **01 · HANGAR** | The board itself | What needs doing today, and what is heating up. |

**All eight are built.**

## Non-negotiable

**No private keys. No seed phrases. Ever.** Public addresses and labels only. The
application does not sign and does not execute transactions. There is no wallet connection,
because there is nothing to connect.

Money is never stored as a float. Decimals are text, compared as integers. Timestamps are
UTC milliseconds; local time exists only at the moment of painting them on screen.

Your data is a single SQLite file on your disk. No account, no cloud, no telemetry.

---

# SECTOR 00 — en español

**Un tablero de operación cripto que parece un juego y funciona como una herramienta.**

## Qué resuelve

Una hoja de cálculo. Esa donde sigues qué mint es mañana, qué wallet tiene la whitelist,
cuánto pagaste y si valió la pena. Funciona hasta que deja de funcionar: una hoja no te
avisa de que una whitelist caduca en cuatro horas, no sabe sostener dos whitelists del
mismo proyecto en la misma wallet, y no te impide escribir `0.1 + 0.2` y obtener
`0.30000000000000004` en una columna que representa dinero.

Pero la hoja de cálculo es solo la mitad de lo que reemplaza. **La otra mitad es el ruido.**

Un grupo alpha de Discord suelta treinta proyectos por semana. Quizá tres son para ti. El
resto es el gusto de otro, la tesis de otro, la bolsa de otro — y llegan en el mismo canal,
al mismo volumen, con la misma urgencia. El filtro que decide cuál de los treinta merece tu
wallet es tuyo, y vive en tu cabeza, donde no te sirve de nada a las dos de la mañana.

SECTOR 00 es donde ese filtro se vuelve algo que se ve. Un proyecto recibe un **tier** en el
momento en que lo anotas — COOK, POTENTIAL, DEGEN, WATCH — y a partir de ahí el tablero
muestra tu lista, ordenada por tu criterio y no por el del canal. Lo que resultó no ser nada
se descarta **con el motivo pegado**, para que seis meses después sepas si tenías razón. El
ruido se queda en Discord. Lo que cruza es lo que tú decidiste que valía.

## La idea de fondo: uno a uno, no comunitario

Los tableros de alpha son comunitarios: muchos ojos, una lista. Éste es lo contrario. **Una
sola persona, su propio criterio, su propia máquina.** No hay cuentas, no hay canal, no hay
nadie más escribiendo en tu lista.

Eso no es una limitación técnica que se arregle luego: es de dónde sale el valor. Una
watchlist compartida termina midiendo lo que le gusta al grupo. Ésta mide lo que aciertas
tú — y por eso puede decirte, con el tiempo, si tu tier vale algo o si estás adivinando.

## Por qué tiene cartas

Porque lo que mata una watchlist no es olvidarse de escribir en ella. Es escribir la mitad y
no volver nunca.

Así que la ficha es una **carta de colección que evoluciona según la llenas**. Dos campos y
el proyecto existe — ésa es la etapa bebé. Pasada la mitad, evoluciona. Llena la ficha
entera y la carta gira tres veces en el aire, aterriza y enseña el arte épico. Ves una de
ésas y sabes lo que costó.

Nada de eso es adorno atornillado encima. Es la respuesta a un problema real: los registros
a medias.

## El hangar

<!-- FALTA LA CAPTURA 12-placa-operador.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/12-placa-operador.png" width="820"
     alt="La placa del operador, con el cronómetro del turno y el acceso al cassette">
<sub><b>La placa del operador</b> · tu nombre, el cronómetro del turno y el testigo del servidor. Detrás del cronómetro vive el cassette, la mascota que sube de etapa con los días que abres el tablero.</sub>
-->

Ocho islas flotantes, una por oficio, navegables con el ratón o con las flechas. Un mecánico
camina hasta la isla que seleccionas y te cuenta para qué sirve. Pulsos de luz recorren las
pasarelas hacia lo que necesita atención.

## Las ocho islas

<!-- FALTA LA CAPTURA 04-race-strip.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/04-race-strip.png" width="900"
     alt="RACE STRIP en la vista EN PISTA: los mints que todavía están por delante">
<sub><b>02 · RACE STRIP</b> · lo vivo y lo archivado, y la línea entre los dos es el estado, nunca la fecha.</sub>
-->

<!-- FALTA LA CAPTURA 07-farm-rig.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/07-farm-rig.png" width="900"
     alt="FARM RIG con el semáforo de constancia: verde, ámbar y rojo según el intervalo">
<sub><b>05 · FARM RIG</b> · el semáforo de constancia. Verde mientras cumples tu intervalo, ámbar en cuanto resbalas, rojo al doble. Es la única presión que ejerce el sistema, y es suficiente.</sub>
-->

<!-- FALTA LA CAPTURA 08-showroom.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/08-showroom.png" width="900"
     alt="SHOWROOM con la tarjeta de resultado: COMPRA, VENTA, PnL y TIEMPO">
<sub><b>08 · SHOWROOM</b> · la tarjeta de resultado, con sus dos marcos y sus cuatro casillas. El marco no se elige: lo elige el número.</sub>
-->

<!-- FALTA LA CAPTURA 10-workshop.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/10-workshop.png" width="900"
     alt="WORKSHOP: los enlaces colgados de la pared por cajón, con el distintivo FARMING">
<sub><b>06 · WORKSHOP</b> · los enlaces por cajón, incluido el de las <i>faucets</i>. Un enlace atado a un airdrop vivo enciende <b>⛏ FARMING</b>.</sub>
-->

<!-- FALTA LA CAPTURA 11-signal-array.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/11-signal-array.png" width="900"
     alt="SIGNAL ARRAY: wallets ajenas con su motivo y su contador de aciertos">
<sub><b>04 · SIGNAL ARRAY</b> · <b>solo direcciones públicas y etiquetas.</b> Cada antena lleva obligatorio el <i>por qué la sigues</i> y un contador de aciertos a mano: no existe la API que te diga si alguien tuvo razón.</sub>
-->

<!-- FALTA LA CAPTURA 09-salvage-market.png — quitar estas dos lineas para que aparezca
<img src="docs/capturas/09-salvage-market.png" width="900"
     alt="SALVAGE MARKET: plays secundarias con su motivo, su fecha límite y la franja de peligro">
<sub><b>07 · SALVAGE MARKET</b> · dos preguntas que ninguna otra isla hace, y las dos obligatorias: <i>por qué te interesa</i> y <i>hasta cuándo</i>. La fecha no cierra la posición por ti: la sube arriba con la franja de peligro encendida, para que la cierres tú.</sub>
-->

## Tres puertas para el mismo dato

El coste de entrada decide si el dato llega a existir.

| Puerta | Cuándo | Coste |
|---|---|---|
| **La consola** (`Ctrl+K`) | Cae un mint en Discord a las 2am | Una línea: `Abstract Cats @abscats` → Enter |
| **El asistente** | Te sientas con tiempo | Campo a campo, cada uno explicado |
| **La edición** | Cambió un detalle | Todos los campos a la vez, sin ceremonia |

Las tres escriben en las mismas columnas. No hay un camino de primera y otro de segunda.

## Lo que no se negocia

**Nunca se almacenan, piden ni muestran claves privadas ni frases semilla.** Solo
direcciones públicas y etiquetas. La aplicación **no firma ni ejecuta transacciones**, y no
hay conexión de wallet porque no hay nada que conectar.

- **Ningún importe en coma flotante.** Los decimales se guardan como texto. Un `0.1 + 0.2`
  en una cartera es un error contable que además no avisa.
- **Los instantes son enteros UTC en milisegundos.** La hora local existe solo al pintarla.
- **Las direcciones van en dos columnas:** una conserva las mayúsculas —que en una EVM son la
  suma de verificación y en Solana son parte de la dirección— y la otra, en minúsculas, es la
  única con la que se busca y se compara.
- **Nada se borra.** Lo descartado cambia de estado y guarda el motivo. Un descarte sin
  motivo, seis meses después, vale lo mismo que no haber anotado nada.

  Con **dos excepciones, y están razonadas**: los enlaces del taller y las wallets del radar.
  Un bridge que cerró no tiene historia que contar. El caso que lo deja claro es el de las
  *faucets*: una cadena sale a mainnet y su grifo de testnet deja de existir — no es que el
  enlace se estropeara, es que el proyecto cambió de etapa. Que `DELETE` exista exactamente
  dos veces en ocho islas es la señal de que la regla sigue viva.

## Decisiones que vale la pena conocer

**Dos capas, una sola verdad: el lienzo dibuja el mundo, el DOM dibuja el instrumento.**
El fondo, el mecánico y los efectos son imágenes. Las tablas, las cifras, los formularios y
los rótulos son DOM de verdad: texto seleccionable y accesible. **Ningún número se pinta
dentro de una imagen, nunca.** Las dos capas ni siquiera comparten sistema de coordenadas —
el mundo se ancla al dibujo, el instrumento se ancla a la ventana.

**Un esquema de validación, no dos.** El mismo esquema de Zod valida el formulario en el
navegador y el cuerpo de la petición en el servidor. Una sola definición de qué es válido, y
no dos que se separan a los tres meses.

**Bilingüe, y ninguno de los dos es traducción del otro.** Inglés por defecto, español si se
le pide. La jerga se queda en inglés en los dos idiomas — *mint*, *whitelist*, *floor*,
*airdrop*, *tier* — porque es de donde viene el vocabulario. Los dos textos se escriben uno
al lado del otro, en el sitio donde se usan: así no existe la clave huérfana ni la traducción
que se queda vieja.

**Un enum tiene dos vidas: el valor que se guarda y la palabra que se lee.** En la base sigue
diciendo `VIGILANDO` aunque la pantalla diga otra cosa, porque traducir el valor guardado
obligaría a migrar la base cada vez que se ajuste una palabra.

## Stack

| | |
|---|---|
| **Interfaz** | React 19 · TypeScript 7 · Vite 8 · TanStack Query y Table · React Hook Form |
| **Servidor** | Fastify 5 · Drizzle ORM · SQLite (`better-sqlite3`, en modo WAL) |
| **Compartido** | Zod 4 — los mismos contratos en los dos lados · `dnum` para los decimales |
| **Calidad** | Vitest · Biome |
| **Arte** | Pixel art propio, exportado a WebP por un guion con `sharp` |

Monolito modular con un solo `package.json` y rebanada vertical por funcionalidad: por cada
módulo del servidor existe uno de la interfaz, con el mismo nombre, siempre.

### Dimensiones

| | |
|---|---|
| Islas construidas | 8 de 8 |
| Tablas | 19 |
| Pruebas | 118, en verde |
| Dependencias en tiempo de ejecución | Las justas — sin framework de UI de terceros |
| Servicios externos | Ninguno |

## Estado

**Funcionando y en uso diario.** Las ocho islas están construidas, las tres puertas de
entrada escriben en las mismas columnas, y el tablero reemplazó a la hoja de cálculo.

**En camino**

- **El tacómetro de intensidad** — un 0 a 100 que no se escribe a mano, por seis
  componentes explícitos, con un límite deliberado: no más de cinco proyectos en zona roja a
  la vez. La atención es un recurso finito y ésta es la mecánica que lo hace visible.
- **Instantáneas de seguidores** — velocidad y aceleración, no volumen. Un proyecto que pasó
  de 2.000 a 9.000 en cinco días vale más que uno estancado en 40.000.
- **La tasa de acierto por tier** — el tablero comparando tu criterio de entonces contra el
  resultado real. Si tus COOK aciertan el 30 % y tus DEGEN el 28 %, tu tier no está aportando
  información.
- **El Destilador** — pegar un hilo o un litepaper y recibir siempre la misma estructura de
  siete campos. Se guarda el original completo, se marca como generado y es editable.
- **La base espacial** — si esto llega a varias manos, cada tablero sigue siendo privado,
  pero cada uno puede *enviar* lo que considere top a un centro común. **Cada isla es un
  router; la base espacial es internet.** Lo que se comparte se decide isla por isla, y
  nunca por defecto.

---

## Licencia

**Software propietario. Todos los derechos reservados.**

Copyright © 2026 Rubén Agudelo Alzate.

Este repositorio se publica con fines de **consulta y evaluación**. No se concede licencia
para usar, copiar, modificar, distribuir ni explotar comercialmente el software ni ninguna
parte de él. El texto completo y vinculante está en [`LICENSE`](LICENSE).

Las bibliotecas de terceros conservan sus propias licencias, y ninguna de las condiciones de
arriba pretende alterarlas.

---

<div align="center">
<sub>Hecho en Colombia.</sub>
</div>
