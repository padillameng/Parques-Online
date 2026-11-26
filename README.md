# Parqués Vial — Multijugador online con GitHub Pages + Firebase

Juego educativo de seguridad vial, basado en 32 preguntas y decisiones de la charla. Permite proyectar un tablero y conectar equipos desde **cualquier navegador con internet**.

## Arquitectura (sin servidor en tu PC)

- **GitHub Pages** hospeda la interfaz: `index.html`, `boot.js`, `app.js` y `questions.js`; los estilos y elementos HTML de la pantalla se reconstruyen desde el cargador ofuscado.
- **Firebase Authentication (anónima)** identifica cada navegador invitado sin pedirle usuario ni contraseña. El jugador escribe un nombre y elige equipo.
- **Firebase Realtime Database** sincroniza sala, turnos, respuestas y tablero en tiempo real.
- La pestaña del **presentador debe permanecer abierta** para lanzar el dado y puntuar respuestas recibidas: el algoritmo anfitrión vive en su navegador.
- El juego se sirve por internet. Los jugadores **no requieren LAN, un ejecutable ni instalar nada**.

> **Importante:** Un repositorio GitHub Pages de solo HTML no proporciona por sí mismo una base de datos multijugador. No es posible tener sincronización compartida real y segura entre navegadores con archivos estáticos sin algún servicio externo. Firebase presta ese servicio sin montar un servidor local. Puede haber límites/costos conforme a su plan.

## Configuración inicial (una sola vez)

1. Crea un proyecto en [Firebase Console](https://console.firebase.google.com/).
2. En **Authentication > Sign-in method**, habilita **Anonymous**.
3. En **Build > Realtime Database**, crea una base de datos y copia su URL. Elige reglas bloqueadas en la creación: publicarás las reglas del archivo adjunto.
4. En **Realtime Database > Rules**, pega el contenido completo de `database.rules.json` y pulsa **Publish**. Nunca uses reglas globales públicas que permitan leer y escribir todo.
5. En **Project settings > General > Your apps**, registra una **Web App** y copia su `firebaseConfig`.
6. Edita `firebase-config.js` reemplazando los cinco valores de ejemplo (`apiKey`, `authDomain`, `databaseURL`, `projectId`, `appId`). La `databaseURL` debe ser la URL completa de tu instancia de Realtime Database.
7. En **Authentication > Settings > Authorized domains** comprueba que el dominio donde correrá la página esté autorizado (normalmente `<usuario>.github.io`). En algunos proyectos tendrás que agregarlo manualmente.
8. Crea un repositorio público en GitHub, sube **los archivos de esta carpeta en la raíz** (no el ZIP).
9. En el repositorio, abre **Settings > Pages > Build and deployment > Deploy from a branch**. Elige **main** y **/(root)**, guarda los cambios.
10. Abre `https://<usuario>.github.io/<repositorio>/`. Si ves la interfaz del juego sin aviso de configuración, está listo.

## Cómo jugar — v2.5 (podio, historial de respuestas y emojis de carreras)

1. El presentador crea una sala nueva **v2.5** y comparte el QR. Se admiten **hasta 3 integrantes por equipo**, 4 colores y 12 personas como máximo.
2. Cada jugador elige **nombre, equipo y un emoji de carreras** entre 10 opciones (corredores, ciclistas, vehículos y otros símbolos de competición). Su emoji se muestra junto al nombre al finalizar la partida.
3. El primero de los tres integrantes del equipo activo que lance el dado representa a su equipo. Si ninguno lo hace antes de 10 segundos, el presentador realiza el lanzamiento automáticamente.
4. El equipo activo tiene prioridad exclusiva para responder durante el plazo configurable (1 a 10 segundos, 2 por defecto); después los demás colores pueden robar. La primera selección A/B/C/D recibida en Firebase por cada equipo es la única que se considera en esa pregunta.
5. Si un equipo se equivoca, queda fuera de esa pregunta pero los demás siguen respondiendo. No se muestra la alternativa correcta hasta que alguien acierte. Un robo acertado muestra **«TURNO ROBADO»** a los miembros del equipo que tenía el turno.
6. Un acierto avanza exactamente el valor del dado, sin bonificaciones. Una pregunta dura 20 segundos; si todos fallan o nadie acierta dentro del límite, nadie avanza. El árbitro pasa automáticamente al siguiente turno.
7. **Cada respuesta evaluada, correcta o incorrecta, queda guardada en el historial de la sala** bajo `answerLog/<turnId>/<team>`, con número de turno, ronda, equipo al que pertenecía el dado, equipo que contestó, nombre, emoji y UID del representante, pregunta, alternativa, resultado, fecha, robo y avance efectivo.
8. Al alcanzarse la meta (18, 28 o 40 casillas) aparece una **pantalla final** con el **equipo campeón en el podio** y las posiciones de los demás, la puntuación acumulada, el número de respuestas correctas e incorrectas, el porcentaje de efectividad por equipo y una **tabla detallada de los representantes de cada equipo por turno**. Incluye a quienes respondieron mal antes de un robo correcto.
9. **Reiniciar** comienza una partida desde cero, con los mismos jugadores y su selección de emoji, y limpia `answerLog`. El historial del juego anterior no se conserva después del reinicio; documenta resultados antes de iniciar otra partida si necesitas mantenerlos.
10. La pestaña del presentador debe permanecer abierta para arbitrar turnos, registrar resultados y ejecutar el dado automático.

> **Activación requerida:** actualiza y publica `database.rules.json` en **Firebase Console → Realtime Database → Rules → Publish**. Las reglas de v2.5 exigen un emoji del catálogo para la inscripción y `rulesVersion: "podio-historial-6"`. Después crea una sala nueva; el código GitHub no aplica automáticamente cambios a las reglas del proyecto Firebase.

## Características y limitaciones

- Máximo **cuatro colores/equipos**, con **tres participantes por color** (doce personas en total). Cada persona elige un emoji de carreras al registrarse.
- La sala cierra nuevas inscripciones al comenzar la partida.
- La ventana exclusiva para responder, de **1 a 10 segundos**, corresponde a todos los miembros del equipo activo. Si nadie lanza el dado en 10 segundos, se realiza un lanzamiento automático. Cada equipo tiene un solo intento de respuesta por pregunta y Firebase conserva el primer envío.
- El presentador es la autoridad de juego. Si cierra su pestaña, el tablero persiste pero el flujo necesita que reabra la sala desde **el mismo navegador y perfil**, con la misma sesión anónima (si pierde esa identidad, necesita una sala nueva).
- El acceso es **por invitación mediante código**, no cuentas personales ni claves; un usuario que conozca el código puede leer la partida si inicia sesión de invitado.
- **No es antifraude**: las respuestas correctas están incluidas en el JavaScript público, porque un sitio estático sin servidor propio no puede mantener secretos. Es una dinámica de capacitación, no una competición con premios de valor.
- No persigue ni almacena correos ni contraseñas; guarda nombres de participante, emoji, ID anónimo, respuestas del equipo e información de la partida en Firebase. No ingreses datos personales sensibles. Los datos quedan en la base de Firebase hasta eliminarlos manualmente o configurar una política de borrado.
- **PPE-709 — Manual Movilidad + Segura** se menciona como referencia interna, **sin atribuirle requisitos concretos** pues su contenido no fue suministrado.
- Para el QR se usa la biblioteca externa `qrcodejs` mediante CDN; si la biblioteca no carga, los participantes pueden copiar o abrir el enlace manualmente.
- Si habilitas reglas, API key o dominios de forma diferente, debes validar el entorno en tu proyecto Firebase.

## Seguridad y publicación

`firebase-config.js` contiene identificadores **públicos de cliente** (no contraseñas). No agregues credenciales de administrador, tokens privados ni archivos JSON de cuentas de servicio al repositorio. Las reglas de `database.rules.json` permiten al invitado registrar un intento para su propio equipo cuando corresponde, bloquean los intentos duplicados y verifican los 2 segundos de prioridad con la hora del servidor. Solo el anfitrión puede editar el estado general del tablero.

**Atención:** la creación de salas permite usuarios anónimos autenticados. Para un sitio público muy utilizado, considera controles de abuso, cuota, limpieza automática de salas antiguas y Firebase App Check. Revisa costos y límites vigentes antes de invitar a un grupo numeroso.

## Archivos

```
index.html           Entrada HTML mínima al juego
demo.html            Entrada HTML mínima a la demo
demo.js              Simulación ofuscada por etapas sin Firebase
boot.js              Arranque, HTML y estilos codificados del juego y de la demo
app.js               Módulo distribuido ofuscado del juego
questions.js         Banco de 32 preguntas codificado
firebase-config.js   Identificadores públicos del SDK Firebase (codificados)
database.rules.json  Reglas de acceso que debes copiar a Firebase
README.md            Esta guía
```

**Nota:** Conservar esta configuración es condición para que el proyecto funcione en GitHub Pages. Sin Firebase, se muestra una pantalla de instrucciones en lugar de una partida falsa.
## Versión y caché

- El nuevo arranque ofuscado utiliza **v3.0.0** para invalidación de caché de `boot.js`, `app.js` y `demo.js`. Las reglas de las partidas siguen siendo `podio-historial-6`.
- Las nuevas salas se identifican por `rulesVersion: "podio-historial-6"`. Las partidas anteriores no disponen de la auditoría final y requieren crear otra sala.
- El historial de respuestas no sustituye a un registro certificado: la app es educativa, Firebase sincroniza el primer envío de cada equipo y la latencia de red puede influir en el orden de recepción.


## Identidad visual de seguridad vial — v2.6

- El circuito tiene **asfalto, marcas de carril, pasos de cebra, semáforos, señales PARE, zona escolar, obras, ciclovía y advertencias**.
- Los cuatro recuadros de equipo se presentan como estacionamientos de vehículos de colores y la meta está identificada por una bandera de llegada.
- La interfaz de la portada, el registro, las preguntas, las opciones de respuesta, los temporizadores y la tabla/podio final comparten colores y señales propias de la seguridad vial.
- La señalización colocada en determinadas casillas es **exclusivamente ilustrativa**: no incorpora penalizaciones, cambios de turno, bonificaciones ni reglas nuevas. Las 52 casillas existentes, la puntuación exacta del dado, el robo de turnos, los tres cupos por equipo y el historial se mantienen sin cambios.
- Se conserva `rulesVersion: "podio-historial-6"` y la estructura de Firebase. **Esta actualización visual no requiere publicar reglas nuevas en Firebase ni crear otra sala**; basta con recargar la página en todos los dispositivos.
- Las pantallas se adaptan a teléfonos y a la proyección del tablero en escritorio.

## Diseño plano y rectangular — v2.7

- Se retiraron **degradados, difuminados, brillos, sombras, transparencias decorativas, animaciones y esquinas redondeadas** del estilo visual. Se usan superficies de color sólido y bordes rectos.
- Los botones, tarjetas, tableros, contadores, fichas, señales, indicadores, paneles y podio siguen la misma composición plana y sobria.
- Se mantienen colores distintivos por equipo, contrastes de seguridad vial y pasos de cebra identificables sin degradados; los colores de las bases y del marcador siguen vinculados a cada equipo.
- Esta edición es únicamente de **CSS/interfaz** y no modifica `rulesVersion: "podio-historial-6"`, la puntuación, las preguntas, Firebase ni el avance de las fichas. No requiere una nueva sala ni publicar reglas distintas.

## Demo guiada sin Firebase — v2.8

- Acceso directo: `https://padillameng.github.io/Parques-Online/demo.html`. También hay un botón **«🎮 Probar demo»** en la cabecera y una tarjeta en la portada.
- **No requiere jugadores conectados, sala, QR, Firebase Authentication ni Realtime Database**. Abre una simulación que utiliza el catálogo de preguntas local `questions.js`, los colores y las 52 casillas del circuito real.
- Un tutorial interactivo muestra **siete etapas**: registro simulado de hasta tres integrantes por color; lanzamiento manual o automático a los 10 s; prioridad exclusiva configurable (1–10 s, dos por defecto); intento incorrecto; robo por otro equipo; resultado y rotación automática de turno; podio, score de equipo, representantes y efectividad.
- Alterna entre **vista del presentador** y **vista del jugador** para comprender qué acciones estarán disponibles en cada dispositivo. Puedes seleccionar el integrante que representa a un equipo, elegir A–D y comprobar que se bloquean respuestas repetidas por color.
- Para permitir leer las explicaciones, **los relojes de la demo comienzan cuando el visitante pulsa el botón correspondiente**, o se pueden simular los tiempos agotados de inmediato. **En el juego productivo los relojes empiezan y terminan automáticamente**.
- Dado y registros de la demo son **ficticios**, no constituyen partidas ni se sincronizan o guardan. Las muestras del podio simulan el resto de la partida y no deben interpretarse como historial real.
- Esta mejora no cambia `rulesVersion: "podio-historial-6"`, reglas de Firebase, lógica de puntaje, orden de equipos ni condiciones de victoria. Tampoco requiere salas nuevas.

## Favicon y ofuscación de distribución — v2.9

- Se agregó `favicon.svg`, un **semáforo rectangular de diseño plano** en azul marino, rojo, amarillo y verde. Se enlaza desde `index.html` y `demo.html`.
- Los módulos `app.js` y `demo.js` se publican con el texto original transformado a datos codificados/ofuscados; al cargarse en el navegador se reconstruyen como módulos ES de forma dinámica. `questions.js` exporta el catálogo decodificado mediante `JSON.parse` sin exponer las preguntas como texto directo en el archivo publicado.
- Se conservan las importaciones de Firebase, la autenticación anónima, las reglas de seguridad y las 32 preguntas. `firebase-config.js` contiene **identificadores públicos del SDK**, ahora representados de forma codificada sin alterar sus valores; no debe contener secretos.
- **Limitación importante:** esta ofuscación solamente dificulta la lectura casual del código entregado al navegador. **No es cifrado ni protección real de secretos o propiedad intelectual**: cualquier usuario avanzado puede decodificarlo y el historial de Git del repositorio público conserva revisiones anteriores legibles. Si se requiere ocultar el código fuente verdaderamente, hay que mantenerlo en un repositorio privado y publicar solo el resultado de una compilación; las operaciones sensibles deben residir en un backend protegido.
- Para futuras modificaciones, no edites directamente los datos ofuscados. Recupera una revisión anterior de código legible desde el historial de Git y vuelve a producir los archivos de distribución. No se generan mapas de código fuente en esta versión.
- La actualización **no cambia el contrato de Firebase ni requiere crear salas nuevas**. La demo sigue funcionando sin autenticación.

## Ofuscación integral de los archivos publicados — v3.0.0

- `index.html` y `demo.html` son documentos mínimos de arranque. La composición visual original de las páginas se reconstruye desde `boot.js`.
- `boot.js` contiene la estructura HTML y los estilos del juego y la demostración como datos codificados, decodificados solamente al cargar la página. Los antiguos `style.css` y `demo.css` en texto legible ya no están presentes en la rama publicada; las hojas de estilo se inyectan en un elemento `<style>`.
- `app.js`, `demo.js`, `questions.js` y `firebase-config.js` entregan contenido transformado sin publicar directamente los métodos, las preguntas o los identificadores de configuración en forma de texto legible en esos archivos.
- Se requiere JavaScript activado y compatibilidad con módulos ES, Blob, TextDecoder y atob (navegadores web modernos). La página original también requería módulos ES y JavaScript.
- La ofuscación **no es protección criptográfica**: el navegador puede reconstruir el código y un tercero también; el historial público del repositorio conserva versiones anteriores que eran legibles. Para impedir que el código original siga público hay que cambiar el modelo de distribución: repositorio fuente privado, rama o repositorio de publicación con artefactos compilados; backend propio para operaciones realmente confidenciales.
- Las reglas de Firebase en `database.rules.json` siguen disponibles para mantenimiento y no deben tratarse como secretos; la seguridad depende de que las reglas se apliquen realmente en Firebase.
