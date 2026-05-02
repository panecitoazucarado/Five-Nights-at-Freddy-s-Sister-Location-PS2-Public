# Five Nights at Freddy's: Sister Location PS2

Port/prototipo de *Five Nights at Freddy's: Sister Location* para PlayStation 2 hecho sobre AthenaEnv. El proyecto funciona como una aplicacion JavaScript: `athena.elf` inicia Athena, `athena.ini` apunta al script principal y los archivos JS cargan escenas, imagenes, audio, textos, entradas del control y transiciones.

Este README esta escrito para entender el proyecto y para facilitar su futura edicion desde un motor visual como Athena Studios Visual. La idea principal es separar el juego en piezas editables: escenas, assets, sonidos, textos, botones, triggers, timelines y transiciones.

## Resumen Rapido

- Runtime: AthenaEnv para PS2.
- Lenguaje: JavaScript modular con `import` / `export`.
- Resolucion usada por el codigo: `640x448`.
- Script inicial: `PS2DATA/DATA/SCRIPTS/main.js`.
- Gestor central: `SceneManager`.
- Render por frame: `renderScreen(() => { ... })`, que usa `Screen.display`.
- Imagenes: `ImageManager`.
- Efectos cortos: `SfxManager`.
- Musica/voces/audio largo: `StreamManager`.
- Idiomas: `LanguageSystem`, `languages.json` y archivos `.cfg`.
- Flujo actual: logos, idioma, warning, intro, menu, Night 1, Night 2 y base de Night 5.

## Estado Actual

Implementado o parcialmente implementado:

- Logos iniciales.
- Selector de idioma con banderas.
- Warning localizado.
- Introduccion y title screen.
- Night 1 casi completa.
- Night 2 parcialmente implementada.
- Night 5 con elevador, ducto, parts and services y scooping room base.
- Sistema de carga/liberacion de imagenes y sonidos.
- Sistema basico de localizacion.
- Workflow para crear ISO al hacer push a `main`.

Pendiente o incompleto:

- `night3.js` esta vacio.
- `night4.js` esta vacio.
- `CONFIGS/configs.js` esta vacio.
- Guardado, pausa, loading screen, logros, subtitulos y extras siguen como tareas.
- Algunos idiomas estan declarados pero no tienen archivo `.cfg`.
- Algunas rutas tienen diferencias de mayusculas/minusculas que conviene revisar para Linux y PS2.

## Arranque Desde Athena

Athena lee `athena.ini`:

```ini
boot_logo = true
dark_mode = true
default_script = "PS2DATA/DATA/SCRIPTS/main.js"

audsrv = true
```

Significado:

- `boot_logo`: muestra el logo de arranque del runtime.
- `dark_mode`: activa el modo oscuro de Athena.
- `default_script`: define el JS que Athena ejecuta primero.
- `audsrv`: habilita el sistema de audio requerido por `Sound.Sfx` y `Sound.Stream`.

Para Athena Studios Visual, este archivo puede tratarse como el manifiesto de arranque del proyecto.

## Estructura General

```text
.
|-- athena.elf
|-- athena.ini
|-- list.txt
|-- README.md
|-- .github/workflows/build-iso.yml
|-- PS2DATA/
|   |-- DATA/
|   |   |-- ASSETS/
|   |   |   |-- FONTS/
|   |   |   |-- LOGO/
|   |   |   |-- SOUND/
|   |   |   `-- SPRITES/
|   |   |-- ISOF/
|   |   |-- LANGUAGES/
|   |   |-- SCRIPTS/
|   |   `-- languages.json
|   `-- MODULES/
`-- flags/
```

Carpetas importantes:

- `PS2DATA/DATA/SCRIPTS`: codigo JS del juego.
- `PS2DATA/DATA/ASSETS/SPRITES`: imagenes por escena.
- `PS2DATA/DATA/ASSETS/SOUND`: efectos, voces y musica.
- `PS2DATA/DATA/ASSETS/FONTS`: fuentes usadas por menu y textos.
- `PS2DATA/DATA/LANGUAGES`: traducciones.
- `flags`: selector de idioma, banderas, fuente FNAF, jumpscare y audio del selector.
- `PS2DATA/DATA/ISOF`: archivos copiados a la raiz de la ISO.

## Flujo de Ejecucion Completo

El juego se mueve por escenas. Cada escena es una funcion que crea recursos, declara estado local y registra un render/update con `renderScreen`.

Flujo inicial:

```text
athena.elf
-> athena.ini
-> PS2DATA/DATA/SCRIPTS/main.js
-> setupGlobals()
-> SceneManager.load(MenuManager.logoUpdate)
-> MenuManager.logoUpdate
-> MenuManager.logo2Update
-> MenuManager.logo3Update
-> MenuManager.languageSelect
-> MenuManager.warningscreen
-> MenuManager.introAndTitle
-> Night1Scene.elevatorScene
```

Flujo jugable principal actual:

```text
Night1Scene.elevatorScene
-> Night1Scene.ventcrawl1
-> Night1Scene.mainhub
-> Night1Scene.ventcrawl2
-> Night1Scene.babyroom
-> Night1Scene.ventcrawl3
-> Night1Scene.shiftcomplete
-> Night1Scene.tvshowepisode1
-> Night2Scene.elevatorScene
```

Night 2 continua con:

```text
Night2Scene.elevatorScene
-> Night2Scene.ventcrawl1
-> Night2Scene.mainhub
-> Night2Scene.ventcrawl2
-> Night2Scene.babyroom
-> Night2Scene.underdesk
```

Night 5 tiene base para:

```text
Night5Scene.elevatorScene
-> Night5Scene.ventcrawl1
-> Night5Scene.baby
-> Night5Scene.scoopingroom
```

## Modelo Mental Para Athena Studios Visual

El codigo actual puede mapearse a un editor visual con estos conceptos:

- Proyecto: `athena.ini` mas carpeta `PS2DATA`.
- Escena: una funcion como `Night1Scene.elevatorScene()`.
- Nodo Sprite: una instancia de `ImageManager`.
- Nodo Audio SFX: una instancia de `SfxManager`.
- Nodo Audio Stream: una instancia de `StreamManager`.
- Timeline o Animation Strip: un array de `ImageManager` con frames numerados.
- Trigger Area: objeto `{ x, y, width, height, sound }` o `{ x, y, width, height }`.
- Cursor/Player Pointer: variables `rectX`, `rectY`, `rectWidth`, `rectHeight`.
- Camera2D: variables `cameraX`, `cameraY`, `targetCameraX`, `targetCameraY`.
- State Machine: strings como `introPhase`, `tabletState`, `gameSequence`, `sequenceState`, `gameState`.
- Transition: llamada `SceneManager.load(OtraEscena.metodo)`.
- Localized Text: `LanguageSystem.getText("clave")`.
- Localized Audio: `LanguageSystem.getLocalizedStream(path)` o `getLocalizedSfx(path)`.

## Codigo de Entrada: `main.js`

Archivo:

```text
PS2DATA/DATA/SCRIPTS/main.js
```

Contenido:

```js
import { setupGlobals } from "./CONFIGS/scenes.js";

setupGlobals();

SceneManager.load(MenuManager.logoUpdate);
```

Explicacion linea por linea:

- Importa `setupGlobals` desde `CONFIGS/scenes.js`.
- Ejecuta `setupGlobals()`, que importa managers, escenas y fuentes, y las coloca en `globalThis`.
- Llama `SceneManager.load(MenuManager.logoUpdate)`.
- `SceneManager.load` limpia recursos anteriores, guarda la escena activa y ejecuta la funcion.
- La primera escena real es `MenuManager.logoUpdate`.

Por que el proyecto usa globals:

- Muchas escenas llaman directamente a `SceneManager`, `Night1Scene`, `MenuManager`, `font`, `CONSOLA`, etc.
- En vez de importar todo en cada archivo, `setupGlobals()` centraliza esas referencias.
- Para un motor visual, `setupGlobals()` seria equivalente al registro de clases/servicios globales del proyecto.

## Registro Global: `CONFIGS/scenes.js`

Este archivo conecta los modulos:

```text
PS2DATA/DATA/SCRIPTS/CONFIGS/scenes.js
```

Importa:

- `SceneManager`, `ImageManager`, `SfxManager`, `StreamManager`.
- `Night1Scene`, `Night2Scene`, `Night5Scene`.
- `MenuManager`.
- `renderScreen`.
- `LanguageSystem`.

Luego registra todo:

```js
globalThis.ImageManager = ImageManager;
globalThis.StreamManager = StreamManager;
globalThis.SfxManager = SfxManager;
globalThis.SceneManager = SceneManager;
globalThis.renderScreen = renderScreen;
globalThis.LanguageSystem = LanguageSystem;
```

Tambien registra escenas y fuentes.

Para Athena Studios Visual:

- Este archivo es el lugar ideal para registrar nuevas escenas.
- Si se crea `Night3Scene`, debe importarse aqui y agregarse a `globalThis`.
- Si se agregan managers nuevos, tambien deben registrarse aqui o importarse directamente en cada escena.

## Render Loop: `CONFIGS/render.js`

`renderScreen(callback)` envuelve `Screen.display`.

Patron:

```js
renderScreen(() => {
    // update inputs
    // update timers/state
    // draw background
    // draw sprites
    // draw overlay/cursor/fade
});
```

Este callback es el update/draw de la escena. Todo lo que cambia por frame vive dentro de ese bloque.

El archivo tambien incluye `DebugMemory()`, que calcula RAM usada/libre, pero `debugMemoryEnabled` esta en `false`. Si se activa, el texto se dibuja encima de la escena.

Para un motor visual, `renderScreen` representa el "Frame Update" de una escena.

## Managers Principales

Archivo:

```text
PS2DATA/DATA/SCRIPTS/UTILS/scenemanager.js
```

### `ImageManager`

Responsabilidad:

- Cargar imagenes.
- Guardar `width`, `height`, `color`.
- Dibujar.
- Liberar VRAM/RAM.
- Registrarse en `SceneManager`.

Uso comun:

```js
const cursorImage = new ImageManager("PS2DATA/DATA/ASSETS/SPRITES/ELEVATOR/cursor.png");
cursorImage.draw(x, y);
```

Propiedades editables visualmente:

- `path`
- `x`
- `y`
- `width`
- `height`
- `color`
- `filter`

Detalle importante:

- El constructor carga la imagen inmediatamente.
- El `SceneManager` la rastrea para liberarla cuando se cambia de escena.

### `SfxManager`

Responsabilidad:

- Cargar efectos cortos con `Sound.Sfx`.
- Reproducirlos en canal libre o canal definido.
- Controlar volumen, pan, pitch y loop.
- Liberar el SFX.

Uso comun:

```js
const beep = new SfxManager("PS2DATA/DATA/ASSETS/SOUND/SFX/beep.adp");
beep.play();
```

Propiedades editables:

- `path`
- `volume`
- `pan`
- `pitch`
- `loop`

Ideal para:

- Beeps.
- Botones.
- Shock.
- Buzz.
- Puertas.
- Golpes cortos.

### `StreamManager`

Responsabilidad:

- Cargar audio largo con `Sound.Stream`.
- Reproducir voces, musica o loops largos.
- Pausar, rebobinar, ajustar posicion y volumen.
- Limitar memoria de streams.

Uso comun:

```js
const introAudio = new StreamManager("PS2DATA/DATA/ASSETS/SOUND/STREAM/intro.wav");
introAudio.play();
```

Detalle critico:

```js
static MAX_STREAMS = 1;
```

Solo se permite un stream cargado al mismo tiempo. Si se carga otro, el mas viejo puede liberarse. Esto ayuda a PS2, pero significa que no conviene superponer varias voces/musicas largas como streams.

Ideal para:

- Voces de HandUnit.
- Musica de fondo.
- Episodios de TV.
- Ambientes largos.

### `SceneManager`

Responsabilidad:

- Saber que escena esta activa.
- Rastrear imagenes y sonidos cargados.
- Limpiar recursos al cambiar de escena.
- Ejecutar la nueva escena.

Metodo mas importante:

```js
SceneManager.load(Night1Scene.ventcrawl1);
```

Que hace:

1. Marca `sceneLoading = true`.
2. Llama `clear()`.
3. Libera imagenes y sonidos rastreados.
4. Ejecuta `std.gc()` si existe.
5. Guarda `currentScene = sceneFunction`.
6. Ejecuta la funcion de escena.
7. Marca `sceneLoading = false`.

Para Athena Studios Visual, `SceneManager.load` es el nodo "Cambiar escena".

## Sistema de Idiomas

Archivos:

```text
PS2DATA/DATA/languages.json
PS2DATA/DATA/LANGUAGES/english.cfg
PS2DATA/DATA/LANGUAGES/portuguese.cfg
```

`LanguageSystem.init()`:

- Lee `languages.json`.
- Guarda lista de idiomas.
- Selecciona `defaultLanguage`.
- Carga el archivo `.cfg` del idioma por defecto.

Formato de traduccion:

```cfg
menu_new_game=N E W   G A M E
menu_continue=C O N T I N U E
menu_extras=E X T R A S
```

Uso:

```js
const text = LanguageSystem.getText("menu_new_game");
CONSOLA.print(x, y, text);
```

Audio localizado:

```js
const handunit01c = LanguageSystem.getLocalizedStream(
    "PS2DATA/DATA/ASSETS/SOUND/NIGHT1/handunit01c.wav"
);
```

Como construye rutas localizadas de audio:

```text
PS2DATA/DATA/ASSETS/SOUND/NIGHT1/handunit01c.wav
-> PS2DATA/DATA/ASSETS/SOUND/NIGHT1/EN/handunit01c.wav
```

Nota importante:

- En disco existen carpetas `en` y `pt_br` en minusculas.
- El codigo usa `lang.toUpperCase()` para construir `EN` o `PT_BR`.
- En sistemas case-sensitive eso puede fallar. Conviene normalizar rutas o carpetas.

Idiomas declarados:

- `pt_br`: existe `portuguese.cfg`.
- `en`: existe `english.cfg`.
- `fr`, `de`, `es`, `it`: declarados, pero faltan sus `.cfg`.

## Menu y Pantallas Iniciales

Archivo:

```text
PS2DATA/DATA/SCRIPTS/UI/ui.js
```

Clase:

```js
class MenuManagerClass
```

### Constructor

```js
constructor() {
    LanguageSystem.init();
}
```

Inicializa idiomas al crear `MenuManager`.

### `logoUpdate()`

Carga:

- `eclipsesound.wav`
- `eclipse.png`
- `eclipsetext.png`

Funcionamiento:

- Hace fade in del logo.
- Reduce el logo hasta un tamano inicial.
- Lo mueve hacia la izquierda.
- Muestra el texto.
- Despues de unos segundos hace fade out.
- Cambia a `MenuManager.logo2Update`.

Estados principales:

- `phase = 1`: aparece y se reduce.
- `phase = 2`: pausa breve.
- `phase = 3`: movimiento hacia posicion final y texto.

### `logo2Update()`

Muestra `eod.png` con fade in, espera y fade out. Luego carga `logo3Update`.

### `logo3Update()`

Muestra `warning.png` con fade in/out. Luego carga `languageSelect`.

### `languageSelect()`

Carga:

- Lista de idiomas desde `LanguageSystem.getLanguages()`.
- Fondos y musica de `flags`.
- 24 frames de jumpscare.
- 4 frames de estatica.
- Banderas desde `flagPath`.

Controles:

- `LEFT`: idioma anterior.
- `RIGHT`: idioma siguiente.
- `CROSS` o `CIRCLE`: confirmar.

Funcionamiento:

- Muestra bandera central con pulso.
- Muestra bandera anterior/siguiente como laterales.
- Dibuja nombre del idioma.
- Dibuja si tiene doblaje o no.
- Al confirmar, llama `LanguageSystem.setLanguage(selectedLang.id)`.
- Pausa musica, reproduce scream, muestra jumpscare y estatica.
- Luego carga `warningscreen`.

### `warningscreen()`

Usa:

- `GENERAL/LINE/1..6.png`.
- Textos `warning_title`, `warning_line1`, `warning_line2`.

Funcionamiento:

- Dibuja lineas animadas de fondo.
- Dibuja texto localizado.
- Hace fade in, espera 5 segundos, fade out.
- Carga `introAndTitle`.

### `introAndTitle()`

Tiene dos partes: introduccion y title screen.

Intro:

- Carga `INTRODUCTION/foot.png`, `body.png`, `head.png`, `face.png`.
- Reproduce `intro.wav`.
- Mueve una camara vertical sobre el cuerpo.
- Activa la cara cuando llega a cierta posicion.
- Cuando termina, hace pantalla negra y pasa al title.

Title:

- Reproduce `gradual.wav`.
- Carga grupos de frames `TITLE_SCREEN/AM/1..4`.
- Dibuja glitches y lineas.
- Muestra opciones:
  - `menu_new_game`
  - `menu_continue`
  - `menu_extras`

Controles:

- `UP` / `DOWN`: cambiar opcion.
- `CROSS` en `NEW GAME`: fade out y carga `Night1Scene.elevatorScene`.

## Patron de Escena Jugable

La mayoria de escenas siguen este patron:

```js
metodoDeEscena() {
    // 1. Estado del cursor/player
    let rectX = 500;
    let rectY = 263;
    const rectWidth = 10;
    const rectHeight = 10;

    // 2. Estado de camara/mundo
    const maxWidth = 1000;
    const maxHeight = 526;
    let cameraX = 0;
    let cameraY = 0;

    // 3. Control
    const pad = Pads.get(0);

    // 4. Assets
    const cursorImage = new ImageManager("...");
    const audio = new StreamManager("...");

    // 5. Estado de gameplay
    let gameState = "fadein";
    let fadeAlpha = 255;

    // 6. Funciones auxiliares
    function updateSomething(deltaTime) {}
    function drawSomething() {}

    // 7. Loop
    renderScreen(() => {
        const currentTime = Date.now();
        const deltaTime = (currentTime - lastTime) / 1000;
        lastTime = currentTime;

        pad.update();

        // update
        // draw
        // transition
    });
}
```

Para un editor visual, esto se puede convertir en:

- Scene node.
- Variables editables.
- Asset list.
- Input graph.
- Update graph.
- Draw order.
- Transition nodes.

## Sistema de Cursor y Camara

Muchas escenas usan un "cursor jugador" definido por:

```js
let rectX = 500;
let rectY = 263;
const rectWidth = 10;
const rectHeight = 10;
const speed = 300;
```

Ese cursor se mueve con D-Pad y analogico izquierdo:

```js
if (pad.pressed(Pads.LEFT)) rectX -= moveDistance;
if (pad.pressed(Pads.RIGHT)) rectX += moveDistance;
if (pad.pressed(Pads.UP)) rectY -= moveDistance;
if (pad.pressed(Pads.DOWN)) rectY += moveDistance;
```

La camara sigue al cursor:

```js
targetCameraX = rectX + rectWidth / 2 - screenWidth / 2;
targetCameraY = rectY + rectHeight / 2 - screenHeight / 2;
cameraX += (targetCameraX - cameraX) * cameraLerpSpeed;
cameraY += (targetCameraY - cameraY) * cameraLerpSpeed;
```

Dibujo de sprites con mundo/camara:

```js
sprite.draw(worldX - cameraX, worldY - cameraY);
```

Esto es clave para Athena Studios Visual:

- El mundo puede ser mas grande que la pantalla.
- Las posiciones de botones y triggers estan en coordenadas de mundo.
- Los sprites se dibujan restando la camara.
- El cursor se dibuja en pantalla con `rectX - cameraX`, `rectY - cameraY`.

## Colisiones y Botones

El proyecto usa rectangulos simples:

```js
const button = {
    x: 247,
    y: 355,
    width: 22,
    height: 12
};
```

Funcion comun:

```js
function isClickingButton(cursorX, cursorY, cursorWidth, cursorHeight, button, cameraX, cameraY) {
    const buttonScreenX = button.x - cameraX;
    const buttonScreenY = button.y - cameraY;
    const cursorScreenX = cursorX - cameraX;
    const cursorScreenY = cursorY - cameraY;

    return cursorScreenX < buttonScreenX + button.width &&
        cursorScreenX + cursorWidth > buttonScreenX &&
        cursorScreenY < buttonScreenY + button.height &&
        cursorScreenY + cursorHeight > buttonScreenY;
}
```

Para motor visual:

- Crear componente `Hotspot`.
- Propiedades: `x`, `y`, `width`, `height`, `enabled`, `onPress`.
- Al presionar `CROSS`, evaluar colision con cursor.
- Si coincide, ejecutar accion: sonido, cambiar estado, iniciar fade, abrir vent, etc.

## Night 1 en Detalle

Archivo:

```text
PS2DATA/DATA/SCRIPTS/SCENES/night1.js
```

Clase:

```js
class Night1SceneClass
```

El constructor llama `LanguageSystem.init()`.

### `elevatorScene()`

Escena inicial jugable de Night 1.

Assets principales:

- `ELEVATOR/ELEVATORSEQUENCE/1/1..10.png`
- `ELEVATOR/ELEVATORSEQUENCE/2/1..10.png`
- `ELEVATOR/ELEVATORCO/1/1..11.png`
- `ELEVATOR/ELEVATORCO/2/1..11.png`
- `ELEVATOR/TABLET/introduction/1..10.png`
- `ELEVATOR/TABLET/Midle/1..11.png`
- `ELEVATOR/TABLET/closing/1..10.png`
- `ELEVATOR/nolight1.png`, `nolight2.png`, `lights.png`, `condemned.png`
- `GENERAL/overlay.png`, `GENERAL/mous.png`, cursores.

Audio:

- Daughter line localizada.
- HandUnit localizado.
- `fanv4.adp`
- `elevatorrideloop.adp`
- `jingle1v2dpart1.adp`
- `jingle1v2dpart2.adp`
- efectos de keypad, beep, clank.

Estados importantes:

- `introPhase`: `daughterLine`, `blackScreen`, `fadeIn`, `normal`.
- `tabletState`: `hidden`, `introduction`, `middle`, `error`, `eggs`, `closing`.
- `pointCount`: cantidad de clicks correctos en tablet.
- `endingFinished`: termina la interaccion de tablet.
- `finalNoLightPhase`: fase final sin luz.
- `elevatorCOSequencePlaying`: reproduce secuencia final.
- `fadeOutStarted`: transicion al ducto.

Funcionamiento:

1. Pantalla negra mientras suena DaughterLine.
2. Espera breve.
3. Fade in y audio de HandUnit.
4. Se activa tablet.
5. El jugador debe hacer click varias veces dentro del area de puntos.
6. Se muestra error.
7. Se muestra texto `eggs_benedict`.
8. Se cierra la tablet.
9. Se activa fase de temblor, luces y jingle.
10. Luego aparece fase sin luz.
11. El jugador interactua con boton de elevador.
12. Se reproduce secuencia `ELEVATORCO`.
13. Fade out hacia `Night1Scene.ventcrawl1`.

Variables visualmente editables:

- `speed`
- `maxWidth`, `maxHeight`
- `cameraSpeed`
- `shakeIntensity`, `shakeSpeed`
- `tabletX`, `tabletY`
- `pointAreaOffsetX`, `pointAreaOffsetY`, `pointAreaWidth`, `pointAreaHeight`
- `maxPoints`
- posiciones de `soundAreas`
- timings de fade, jingle, no-light y condemned.

### `ventcrawl1()`, `ventcrawl2()`, `ventcrawl3()`

Las escenas de ducto comparten patron.

Assets:

- `VENTCRAWL/SEQUENCE/1/1..16.png`
- `VENTCRAWL/SEQUENCE/2/1..16.png`
- `GENERAL/ventcrawlcontrols.png`

Audio:

- `circusgaleryvent.adp`
- `metal_duct_fast.wav`
- `metal_duct_slow.wav`
- `scaryamb.adp`

Estados:

- `gameState`: `fadein`, `playing`, `fadeout`.
- `progress`: avance del ducto.
- `maxProgress`: avance requerido para salir.
- `isAnimating1`, `isAnimating2`: animacion activa si se presiona `UP`.

Controles:

- `UP`: avanzar.
- `L1`: correr, acelera frames.
- Analogico derecho: mirar/mover camara.

Funcionamiento:

1. Fade in.
2. Si el jugador mantiene `UP`, aumenta `progress` y se animan los frames.
3. Si mantiene `L1`, cambia `frameDelay` a uno mas rapido.
4. Al llegar a `maxProgress`, fade out.
5. Carga la siguiente escena.

Transiciones:

- `ventcrawl1` -> `mainhub`
- `ventcrawl2` -> `babyroom`
- `ventcrawl3` -> `shiftcomplete`

### `mainhub()`

Escena central de control de animatronicos.

Assets:

- `MAINHUB/mainhubsequence/1/1..16.png`
- `MAINHUB/mainhubsequence/2/1..16.png`
- `MAINHUB/Shock/left/*`
- `MAINHUB/Shock/right/*`
- `MAINHUB/Shock/ballora/1..31.png`
- `MAINHUB/Shock/foxy/1..31.png`
- `MAINHUB/middle/1..6.png`
- `MAINHUB/left/1.png`
- `MAINHUB/right/1.png`
- `GENERAL/overlay.png`

Audio:

- HandUnit localizado `Handunit04b`, `handunit05`, `Handunit06`, `handunit07`, `handunit08`, `handunit09`, `handunit10`.
- `elbuzz.adp`
- `energy.adp`
- `circus_vent.adp`
- sonidos de keypad, denied, faces, ennard nose.

State machine principal:

```text
initial
-> leftBuzzOnly
-> leftBuzzPressed
-> leftShockOnly
-> leftShockPressed
-> leftBuzzOnly2
-> leftBuzzPressed2
-> waitingForFinalAudio
-> rightBuzzOnly1
-> rightBuzzPressed1
-> rightShockOnly1
-> rightShockPressed1
-> rightBuzzOnly2
-> rightBuzzPressed2
-> rightShockOnly2
-> rightShockPressed2
-> rightBuzzOnly3
-> rightBuzzPressed3
-> completed
```

Estados de panel:

- `leftShockState`: `off`, `on`, `buzz`, `shock`.
- `rightShockState`: `off`, `on`, `buzz`, `shock`.

Funciones internas importantes:

- `updateMiddleVent(deltaTime)`: anima apertura del ducto central.
- `isClickingButton(...)`: colision entre cursor y botones.
- `updateFlashEffect(deltaTime)`: parpadeos durante shock.
- `updateAnimatronics(deltaTime)`: frames de Ballora/Foxy.
- `updateShockState(side, deltaTime)`: avanza estados despues de buzz/shock.
- `updateButtonStates()`: habilita que botones se pueden presionar.
- `drawShockImage(side, x, y)`: dibuja off/on/buzz/shock.
- `freeAllAudio()`: libera streams y sfx antes de transicion.
- `playAudio(audio)`: reproduce una voz y bloquea input hasta terminar.
- `handleAudioFinished()`: mueve la secuencia despues de cada voz.

Funcionamiento:

1. Reproduce audio inicial.
2. Habilita panel izquierdo.
3. El jugador debe alternar buzz y shock segun la secuencia.
4. Despues pasa al panel derecho.
5. Al final abre el ducto central.
6. Interactuar con el ducto abierto dispara fade out.
7. Carga `Night1Scene.ventcrawl2`.

Para editor visual:

- Esta escena es una state machine de tutorial.
- Cada paso tiene: audio, botones habilitados, accion esperada, resultado.
- Conviene modelarla como una tabla editable y no como ifs fijos.

### `babyroom()`

Escena de Circus Baby.

Assets:

- `BABY_ROOM/sequence/1/1..11.png`
- `BABY_ROOM/sequence/2/1..11.png`
- `BABY_ROOM/off.png`
- `BABY_ROOM/on.png`
- `BABY_ROOM/buzz.png`
- `BABY_ROOM/shock1.png`
- `BABY_ROOM/shock2.png`
- `BABY_ROOM/buzzframe1.png`
- `BABY_ROOM/buzzframe2.png`
- `BABY_ROOM/exit.png`

Audio:

- `handunit11.wav`
- `handunit12.wav`
- `handunit13.wav`
- `handunit14.wav`
- `elbuzz.adp`
- `energy.adp`

State machine:

```text
FADE_IN
-> PLAYING_HANDUNIT11
-> WAIT_BUZZ
-> PLAYING_HANDUNIT12
-> WAIT_SHOCK
-> WAIT_BUZZ_2
-> PLAYING_HANDUNIT13
-> WAIT_SHOCK_2
-> WAIT_BUZZ_3
-> PLAYING_HANDUNIT14
-> COMPLETED
```

Controles:

- Mover cursor con D-Pad/analogico.
- `CROSS` sobre boton buzz o shock.
- `TRIANGLE` cuando aparece salida.

Funcionamiento:

1. Fade in.
2. Reproduce voz.
3. Habilita buzz o shock segun `sequenceState`.
4. Al completar la secuencia, muestra salida.
5. `TRIANGLE` inicia fade out.
6. Carga `Night1Scene.ventcrawl3`.

### `shiftcomplete()`

Pantalla de fin de noche.

Assets:

- `SHIFTCOMPLETE/s/1..31.PNG`
- `SHIFTCOMPLETE/pop1/1..31.PNG`
- `SHIFTCOMPLETE/enjoy.png`

Audio:

- `jingle_4b.wav`
- `Pop.adp`

Funcionamiento:

- Reproduce jingle.
- Dibuja animacion central.
- Genera fuegos artificiales con clase interna `Particle`.
- Dibuja texto `shiftcomplete_message`.
- Cuando termina el audio, hace fade out.
- Carga `Night1Scene.tvshowepisode1`.

### `tvshowepisode1`

Escena del episodio de TV.

Assets:

- `TVSHOW/1/1..4.png`
- `TVSHOW/2/1..4.png`
- `TVSHOW/EPISODE1/TV_000.PNG` hasta `TV_771.PNG`
- `TVSHOW/static_1..4.png`
- `TVSHOW/balde.png`
- `TVSHOW/pipoca.png`
- `TVSHOW/eat.png`

Audio:

- `episode1.wav` localizado.

Detalle de memoria:

- No carga los 772 frames de TV al inicio.
- Usa `maxLoadedFrames = 50`.
- Carga frames bajo demanda.
- Libera frames antiguos con `free()`.

Controles:

- `SQUARE`: lanzar popcorn.

Funcionamiento:

1. Reproduce episodio.
2. Muestra frames de TV bajo demanda.
3. Permite minijuego visual de popcorn.
4. Al terminar frames, muestra estatica.
5. Fade out.
6. Carga `Night2Scene.elevatorScene`.

## Night 2 en Detalle

Archivo:

```text
PS2DATA/DATA/SCRIPTS/SCENES/night2.js
```

`Night2Scene` reutiliza mucho del patron de Night 1:

- Elevador con tablet.
- Ductos.
- Main hub.
- Baby room.
- Underdesk.

Diferencias:

- Usa sprites `night2.png`.
- Usa audios `Night 2-*.wav`, `Angsty*.wav`, `DaughterLine_2.wav`.
- Tiene escena `underdesk()` con mecanica propia.

### `underdesk()`

Assets:

- `UNDERDESK/bg1.png`, `bg2.png`
- `UNDERDESK/metal1.png`, `metal2.png`
- `UNDERDESK/metalsheet1.png`, `metalsheet2.png`
- `UNDERDESK/grabhere.png`
- `UNDERDESK/spot.png`, `spotlight.png`, `spotlightdown.png`
- `UNDERDESK/background.png`
- `UNDERDESK/bidy/1..6.png`

Audio:

- `bidybab01.wav`

Funcionamiento resumido:

- Muestra escena debajo del escritorio.
- Usa cursor y zonas de arrastre.
- Permite resistir/arrastrar una puerta o sheet con `CROSS`.
- Actualiza una secuencia de Bidybab.
- Usa camara/mundo y D-Pad como otras escenas.

Para editor visual:

- Esta escena necesita nodos de arrastre.
- Propiedades clave: area draggable, fuerza/resistencia, progreso, animacion Bidybab, estado de puerta.

## Night 5 en Detalle

Archivo:

```text
PS2DATA/DATA/SCRIPTS/SCENES/night5.js
```

Contiene:

- `elevatorScene()`
- `ventcrawl1()`
- `baby()`
- `scoopingroom()`

### `elevatorScene()`

Similar a los elevadores anteriores, con:

- `night5.png`
- `exoticbutters.png`
- `HandUnit5_*.wav`
- `DaughterLine_5.ogg`

### `baby()`

Escena de Parts and Services 2.

Assets usados:

- `PARTSANDSERVICES2/parts/...`
- `PARTSANDSERVICES2/PARTS/spot/...`
- `PARTSANDSERVICES2/key.png`
- `UNDERDESK/spotlight.png`

Mecanica:

- Cursor.
- Spotlight.
- Area interactiva.
- Secuencias `sequence1` y `sequence2`.

### `scoopingroom()`

Assets:

- `SCOOPINGROOM/SEQUENCE/...`
- cursor de elevador.

Mecanica:

- Cursor con camara.
- Secuencia visual base.

Advertencia:

- Algunas rutas en Night 5 parecen no coincidir exactamente con los nombres reales del disco, por ejemplo `SOUNDS` frente a `SOUND`.
- Conviene auditar rutas antes de usar Night 5 en build final.

## Assets y Convenciones de Nombres

### Imagenes

Usadas con:

```js
new ImageManager("ruta/al/archivo.png")
```

Convenciones actuales:

- Secuencias por carpetas numeradas: `1/1.png`, `1/2.png`, etc.
- Algunas extensiones estan en `.PNG` mayuscula.
- Algunas carpetas tienen mayusculas internas, como `TABLET/Midle`.

Importante:

- Linux y algunos entornos son case-sensitive.
- `image.png` y `image.PNG` no son lo mismo.
- `SOUND/STREAM` y `SOUND/Stream` no son lo mismo.

### Audio

Tipos:

- `.adp`: efectos o audio optimizado para PS2.
- `.wav`: voces, musica, efectos largos.
- `.ogg` y `.mp3`: algunos streams.

Uso recomendado:

- `SfxManager` para efectos cortos y repetibles.
- `StreamManager` para audio largo.
- `LanguageSystem.getLocalizedStream` para voces dependientes de idioma.

## Build de ISO

Workflow:

```text
.github/workflows/build-iso.yml
```

Se ejecuta en push a `main`.

Pasos:

1. Checkout.
2. Instala `genisoimage`.
3. Crea `iso_temp`.
4. Copia archivos raiz, excepto markdown, gitignore, yml/yaml y `FNAF5.elf`.
5. Copia carpetas excepto `.git`, `.github` e `iso_temp`.
6. Copia `PS2DATA/DATA/ISOF/*` a la raiz de `iso_temp`.
7. Genera `FNAF5PS2.iso`.
8. Sube artifact `fnaf5-ps2-iso`.

Volumen:

```text
FNAF5PS2
```

## Guia Para Editar a Mano o Desde Motor Visual

### Agregar una escena nueva

1. Crear clase o metodo en `SCENES/nightX.js`.
2. Registrar la escena en `CONFIGS/scenes.js` si es una clase nueva.
3. Cargar assets al inicio del metodo con managers.
4. Crear variables de estado.
5. Crear funciones auxiliares de update/draw.
6. Usar `renderScreen`.
7. Cambiar de escena con `SceneManager.load(...)`.

Plantilla:

```js
myScene() {
    const pad = Pads.get(0);
    let lastTime = Date.now();
    let fadeAlpha = 255;

    const bg = new ImageManager("PS2DATA/DATA/ASSETS/SPRITES/MY_SCENE/bg.png");
    const music = new StreamManager("PS2DATA/DATA/ASSETS/SOUND/STREAM/music.wav");
    let started = false;

    renderScreen(() => {
        const now = Date.now();
        const deltaTime = (now - lastTime) / 1000;
        lastTime = now;

        pad.update();

        if (!started) {
            music.play();
            started = true;
        }

        bg.draw(0, 0);

        if (fadeAlpha > 0) {
            fadeAlpha -= 120 * deltaTime;
            Draw.rect(0, 0, 640, 448, Color.new(0, 0, 0, fadeAlpha));
        }
    });
}
```

### Crear una secuencia animada

```js
const frames = [];
for (let i = 1; i <= 16; i++) {
    frames.push(new ImageManager(`PS2DATA/DATA/ASSETS/SPRITES/SCENE/ANIM/${i}.png`));
}

let currentFrame = 0;
let frameTimer = 0;
const frameDelay = 50;

frameTimer += deltaTime * 1000;
if (frameTimer >= frameDelay) {
    frameTimer = 0;
    currentFrame = (currentFrame + 1) % frames.length;
}

frames[currentFrame].draw(x, y);
```

En editor visual:

- Asset folder.
- Start index.
- End index.
- Frame delay.
- Loop true/false.
- Draw position.

### Crear un boton/hotspot

```js
const button = { x: 100, y: 120, width: 40, height: 20 };

if (pad.justPressed(Pads.CROSS) &&
    isClickingButton(rectX, rectY, rectWidth, rectHeight, button, cameraX, cameraY)) {
    // accion
}
```

En editor visual:

- Rectangulo interactivo.
- Tecla requerida.
- Condicion de estado.
- Accion: reproducir sonido, cambiar estado, iniciar timeline, cargar escena.

### Agregar texto traducible

1. Agregar clave en todos los `.cfg`.
2. Usar `LanguageSystem.getText("clave")`.
3. Dibujar con la fuente elegida.

Ejemplo:

```js
const text = LanguageSystem.getText("look_around");
NudMotoyaMaruW55.print(36, 416, text);
```

### Agregar audio localizado

1. Poner audio base o localizado en carpeta correcta.
2. Usar `LanguageSystem.getLocalizedStream(...)`.
3. Asegurar que carpetas de idioma coincidan con el codigo.

Ejemplo:

```js
const voice = LanguageSystem.getLocalizedStream("PS2DATA/DATA/ASSETS/SOUND/NIGHT1/handunit11.wav");
voice.play();
```

## Reglas Practicas Para No Romper el Proyecto

- Cambiar de escena siempre con `SceneManager.load`.
- No cargar muchos `StreamManager` a la vez.
- Liberar manualmente audio pesado si una escena lo controla fuera del cambio normal.
- Mantener nombres de rutas exactamente iguales.
- No usar rutas con mayusculas distintas a las del archivo real.
- Para secuencias grandes, cargar bajo demanda como `tvshowepisode1`.
- Mantener `deltaTime` para movimiento y timers, no depender solo de frames.
- Usar `LanguageSystem.getText` para todo texto visible.
- Usar coordenadas de mundo para botones si la camara se mueve.
- Restar `cameraX` y `cameraY` al dibujar sprites del mundo.

## Problemas Conocidos a Revisar

- `LanguageSystem.buildLocalizedPath` convierte idiomas a mayusculas, pero existen carpetas `en` y `pt_br` en minusculas.
- `night2.js` y `night5.js` tienen rutas con `Stream` en vez de `STREAM` en algunos lugares.
- `night5.js` tiene una ruta `SOUNDS` en vez de `SOUND`.
- `night3.js`, `night4.js` y `CONFIGS/configs.js` estan vacios.
- `DebugManager` no esta conectado al render principal.
- El repositorio incluye `.DS_Store`; son archivos de macOS y no son necesarios para el juego.
- Algunas escenas duplican bastante codigo de cursor/camara; convendria extraer helpers si se va a mantener a largo plazo.

## Mapa Para Convertir a Athena Studios Visual

Entidades recomendadas:

```text
Project
|-- BootConfig
|-- GlobalRegistry
|-- SceneManager
|-- LanguageDatabase
|-- AssetLibrary
|-- Scene
|   |-- Variables
|   |-- Sprites
|   |-- Audio
|   |-- Hotspots
|   |-- Timelines
|   |-- StateMachine
|   |-- InputHandlers
|   |-- DrawOrder
|   `-- Transitions
```

Propiedades que conviene exponer por escena:

- `worldWidth`
- `worldHeight`
- `screenWidth`
- `screenHeight`
- `cursorStartX`
- `cursorStartY`
- `cursorSpeed`
- `cameraLerpSpeed`
- `fadeInDuration`
- `fadeOutDuration`
- `frameDelay`
- `runningFrameDelay`
- `audioPath`
- `nextScene`

Acciones visuales basicas:

- `PlaySfx`
- `PlayStream`
- `PauseStream`
- `SetState`
- `IncrementCounter`
- `StartFade`
- `OpenVent`
- `EnableHotspot`
- `DisableHotspot`
- `LoadScene`
- `DrawSprite`
- `DrawFrameSequence`
- `DrawText`

## Licencia

Ver `LICENSE`.
