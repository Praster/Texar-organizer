# Texar Organizer
<img width="256" height="256" alt="icon" src="https://github.com/user-attachments/assets/a3501767-12f9-41e9-a65d-e866e421ad67" />

> Organizador personal de **horario, notas y calendario** con los **festivos de España por provincia**. Sin cuenta, sin anuncios, sin suscripción: tus datos se quedan en tu dispositivo.

Hecho por **joselitox**.

Disponible para **Windows** (Electron) y **Android** (Capacitor), a partir de **un único código** HTML + CSS + JavaScript sin frameworks.

---

## Índice

1. [Características](#características)
2. [Tecnologías](#tecnologías)
3. [Cómo funciona por dentro](#cómo-funciona-por-dentro)
4. [Estructura del repositorio](#estructura-del-repositorio)
5. [Compilar la app](#compilar-la-app)
6. [Copia de seguridad](#copia-de-seguridad)
7. [Privacidad](#privacidad)
8. [Limitaciones conocidas](#limitaciones-conocidas)
9. [Créditos y licencia](#créditos-y-licencia)

---

## Características

### Horario
- **Semana completa de 7 días** (lunes a domingo) con navegación por semanas: ‹ semana anterior, Hoy, siguiente ›.
- Cada actividad tiene título, nota opcional, horas de inicio y fin, categoría y recordatorio.
- **Repetición flexible**: una actividad puede repetirse **cada semana** o ser **solo de una semana**.
  - Al editar una que se repite, eliges entre cambiar **todas** o **solo esta semana**.
  - Al borrar una que se repite, eliges entre **solo esta semana** o **esta y las siguientes** (las semanas pasadas se conservan).
- **Copiar semana anterior**: copia a la semana actual las actividades sueltas de la anterior.
- Marcar actividades como **hechas** (se guarda por semana y se reinicia sola cada lunes).
- Aviso cuando dos actividades se solapan.
- **Categorías propias** con nombre y color a elegir, y filtros por categoría.
- Tarjeta **Ahora / Siguiente** en el panel lateral con el tiempo que falta.

### Notas
- Crea todas las notas que quieras, con buscador.
- **Editor con formato**: negrita, cursiva, subrayado, tachado, fuente, tamaño, color del texto, resaltado, listas con viñetas y numeradas, alineación, deshacer / rehacer y quitar formato.
- Se guardan solas al escribir. Al pegar, el texto entra como texto plano.

### Calendario
- Vista mensual con todos los días.
- **Festivos de España en rojo**, con selector de **provincia** (incluye las islas de Canarias) para ver también los de tu comunidad autónoma.
- **Festivos propios** (por ejemplo, los 2 locales de tu municipio), con opción de repetirse cada año.
- **Eventos con fecha concreta** (exámenes, entregas, citas) con **cuenta atrás** ("mañana", "en 12 días").
- Los puntos de colores de cada día muestran las actividades de esa semana.

### Estadísticas
- Porcentaje de actividades hechas esta semana.
- **Racha**: semanas seguidas cumpliendo al menos el 80 %.
- Horas por categoría e historial de las últimas 8 semanas.

### Personalización
- **Tema claro, oscuro o automático** (sigue al sistema). La barra lateral también cambia con el tema.
- **5 colores de acento**.
- **Fondos personalizados**: galería con 10 imágenes para ordenador y 5 para móvil, intensidad ajustable y opción de **añadir tus propias imágenes**.
- Diseño **responsive**: panel lateral en ordenador y barra de navegación inferior en móvil.

### Datos
- **Copia de seguridad** (exportar / importar en JSON).
- **Exportar a calendario (.ics)** para Google Calendar, Apple Calendar u Outlook.

### Avisos (notificaciones)
- Notificaciones nativas del sistema en **Windows, Android**. Ver [Notificaciones](#notificaciones).

---

## Tecnologías

| Parte | Tecnología |
|---|---|
| Interfaz y lógica | **HTML + CSS + JavaScript** puro (sin frameworks ni paso de compilación) |
| App de escritorio (Windows) | **[Electron](https://www.electronjs.org/)** 33 + **electron-builder** 25 (instalador NSIS) |
| App móvil (Android) | **[Capacitor](https://capacitorjs.com/)** 7 |
| Avisos en móvil | `@capacitor/local-notifications` |
| Barra de estado (Android) | `@capacitor/status-bar` |
| Iconos móviles | `@capacitor/assets` |
| Tipografía | Bricolage Grotesque (Google Fonts, con tipografías del sistema como alternativa) |
| Festivos 2026 | Datos oficiales del **BOE** incluidos en la app |
| Festivos de otros años | API pública **[Nager.Date](https://date.nager.at/)** |
| Almacenamiento | `localStorage` e `IndexedDB` (todo local) |

---

## Cómo funciona por dentro

### Un solo código para todas las plataformas
Toda la aplicación vive en **un único archivo `index.html`** (HTML, CSS y JavaScript juntos, con las imágenes de fondo incluidas en formato WebP codificado en base64). Ese mismo archivo se copia a:

- `texar-organizer-desktop/index.html` → lo carga Electron.
- `texar-organizer-mobile/www/index.html` → lo empaqueta Capacitor.

La app detecta dónde se está ejecutando y activa la parte que corresponde:

| Entorno | Cómo se detecta | Qué activa |
|---|---|---|
| Windows (Electron) | `window.desktop` (lo expone `preload.js`) | Notificaciones nativas de Windows y descarga de festivos desde el proceso principal |
| Android / iOS (Capacitor) | `Capacitor.isNativePlatform()` | Avisos programados con `LocalNotifications`, ajuste de barras del sistema |
| Navegador | ninguna de las anteriores | API `Notification` del navegador y avisos dentro de la página |

### Modelo de datos
Cada actividad del horario es un objeto con estos campos:

```json
{
  "id": "k3j9x",
  "title": "Programación",
  "day": 3,
  "start": "17:00",
  "end": "18:30",
  "rem": 10,
  "cat": "1",
  "note": "Capítulo 4",
  "done": false,
  "wk": "",
  "from": "2026-10-12",
  "until": "",
  "skip": []
}
```

| Campo | Significado |
|---|---|
| `day` | Día de la semana, de 0 (lunes) a 6 (domingo) |
| `rem` | Minutos de antelación del aviso; `-1` = sin aviso, `0` = a la hora exacta |
| `wk` | Si no está vacío, la actividad es **solo de esa semana** (lunes en formato `AAAA-MM-DD`) |
| `from` / `until` | Una actividad que se repite aparece desde la semana `from` hasta antes de la semana `until` |
| `skip` | Semanas concretas en las que una actividad repetida **no** aparece |
| `done` | Semana en la que está marcada como hecha; se reinicia cada lunes |

Una actividad aparece en una semana `w` si (`wk === w`) o, cuando se repite, si `w` está dentro de `from`–`until` y no figura en `skip`. Con esto se consiguen las dos modalidades ("cada semana" y "solo esta semana") sin duplicar datos: editar "solo esta semana" añade esa semana a `skip` y crea una copia suelta con los cambios.

### Almacenamiento
Todo se guarda **en el dispositivo**, sin servidor ni cuenta:

- `localStorage`: actividades, categorías, notas, festivos propios, eventos con fecha, historial de estadísticas, tema, color de acento, provincia y ajustes de fondo.
- `IndexedDB`: las imágenes de fondo que añade el usuario (reducidas y comprimidas a WebP/JPEG).

Cada dispositivo guarda sus datos por separado. Para pasarlos de uno a otro se usa la [copia de seguridad](#copia-de-seguridad).

### Notificaciones

| Plataforma | Mecanismo | Funciona con la app cerrada |
|---|---|---|
| **Windows** | La página pide una notificación al proceso principal de Electron (`ipcMain`), que la muestra con `Notification` nativo. Al cerrar la ventana, la app queda en la **bandeja del sistema** y sigue avisando. Opción "Iniciar con Windows" en el menú de la bandeja | Sí, mientras la app esté en la bandeja |
| **Android / iOS** | Al guardar o abrir la app se **programan avisos con fecha y hora exactas** para las próximas 4 semanas (`LocalNotifications`). Se reprograman al volver a la app y cada 6 horas. Marcar una actividad como hecha cancela el aviso de esa semana | Sí, aunque la app esté cerrada |
| **Navegador** | API `Notification` + aviso dentro de la página con sonido | Solo con la pestaña abierta |

Detalles técnicos de la parte móvil:
- Los avisos tienen un **identificador estable** por actividad y día, de modo que si la app está abierta y salta el aviso inmediato, sustituye al programado en vez de duplicarlo.
- La programación va en **cola**: primero crea los avisos nuevos y después borra los sobrantes, para no quedarse sin ninguno si algo se interrumpe.
- En Android se usan **alarmas exactas** (`SCHEDULE_EXACT_ALARM` / `USE_EXACT_ALARM`) y un canal de notificaciones de importancia alta. Hay botones de prueba en Ajustes (incluida una prueba con la app cerrada).
- **iOS** admite como máximo 64 avisos pendientes; la app programa hasta 60.

### Festivos de España
- **2026:** datos **oficiales del BOE** (Resolución de 17 de octubre de 2025) integrados en la app, por comunidad autónoma y con las festividades por isla de Canarias.
- **Otros años:** la app descarga los festivos de **Nager.Date**, los guarda en el dispositivo y los renueva cada 30 días. En Electron la petición la hace el proceso principal (sin problemas de CORS); en el móvil, el propio WebView.
- Si no hay conexión o datos, se muestran solo los **festivos nacionales calculados** (incluido el Viernes Santo, que cambia cada año).
- La línea bajo el calendario indica en cada momento si el dato es oficial o no.
- **No incluye** los festivos locales de cada municipio (se añaden a mano) ni el festivo propio del Valle de Arán.

> Los datos de Nager.Date **no son oficiales**: conviene contrastarlos con el BOE cuando se publique el calendario de cada año.

### Editor de notas
Está construido con `contenteditable` y la API `document.execCommand`. El contenido se guarda como HTML y se **sanea al cargarlo** con una lista blanca de etiquetas y estilos permitidos, por lo que un archivo de copia de seguridad manipulado no puede ejecutar código. El pegado se fuerza a texto plano.

### Fondos personalizados
Una capa fija detrás de la interfaz muestra la imagen elegida con una **opacidad ajustable** (tenue), de modo que se mezcla con el color del tema y no afecta a la legibilidad de botones y textos. En pantallas anchas se usan las imágenes de ordenador y en pantallas estrechas las de móvil. Las imágenes del usuario se reducen en el propio dispositivo y se guardan en IndexedDB.

### Tema y diseño
- Variables CSS para colores, con versión clara y oscura (y modo automático por `prefers-color-scheme`).
- Diseño responsive con *media queries*, *container queries* (las tarjetas de actividad reorganizan sus botones si la columna es estrecha) y barra inferior en móvil.
- En Android se detectan las barras del sistema (modo "edge-to-edge") para dejar márgenes, y la barra de estado cambia de color con el tema.

---

## Estructura del repositorio

```
texar-organizer/
├── texar-organizer-desktop/        # App de Windows (Electron)
│   ├── index.html                  # Toda la aplicación
│   ├── main.js                     # Proceso principal: ventana, bandeja, notificaciones, festivos
│   ├── preload.js                  # Puente seguro entre la página y Electron
│   ├── package.json                # Dependencias y configuración del instalador
│   ├── icon.png / icon.ico         # Iconos
│   └── README.md
│
├── texar-organizer-mobile/         # App de Android / iOS (Capacitor)
│   ├── www/index.html              # La misma aplicación
│   ├── capacitor.config.json       # appId y carpeta web
│   ├── package.json                # Dependencias y scripts
│   ├── assets/icon-only.png        # Icono base (1024×1024)
│   ├── scripts/patch-android.js    # Añade permisos al AndroidManifest
│   └── README.md
│
└── README.md                       # Este archivo
```

> Las carpetas `node_modules/`, `dist/`, `android/` e `ios/` se generan al compilar y no hace falta subirlas al repositorio (añádelas a `.gitignore`).

**Identificador de la app:** `com.joselitox.texarorganizer`

---

## Compilar la app

Requisito común: **[Node.js](https://nodejs.org/) 20 o superior**.

### Windows (instalador `.exe`)

```bash
cd texar-organizer-desktop
npm install
npm start          # probar sin instalar
npm run dist       # generar el instalador
```

El instalador queda en `dist/` (`Texar Organizer Setup X.Y.Z.exe`). La carpeta `dist/win-unpacked` es solo la app sin instalar.

- Windows puede mostrar un aviso de "editor desconocido", porque el instalador no está firmado digitalmente.
- Si la compilación se corta con el error *"Cannot create symbolic link"*, activa el **Modo desarrollador** de Windows (Configuración › Sistema › Para desarrolladores) o abre la terminal como administrador.
- Para ver las notificaciones, revisa Configuración › Sistema › Notificaciones y que "No molestar" esté desactivado.

### Linux

Si fallan el .deb ejecutar en la terminal en la carpeta donde este el deb:
```bash
sudo apt install -y ~/texar-organizer/dist/Texar-Organizer-x.x.x-amd64.deb
```
Sustituye las x del comando por los números de la versión de la app.

### Android

Requisito adicional: **[Android Studio](https://developer.android.com/studio)**.

```bash
cd texar-organizer-mobile
npm install
npm run android:init    # crea el proyecto Android, añade permisos e iconos (solo la primera vez)
npm run android:open    # abre Android Studio
```

En Android Studio: **Build › Build Bundle(s) / APK(s) › Build APK(s)**. El archivo queda en `android/app/build/outputs/apk/debug/app-debug.apk`. Cópialo al móvil y permite instalar apps de origen desconocido.

Si el proyecto de Android ya existía antes, comprueba que `android/app/src/main/AndroidManifest.xml` incluye, antes de `<application`:

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

**Para que los avisos funcionen bien en el móvil:**
- Acepta el permiso de notificaciones la primera vez.
- Activa "Alarmas y recordatorios" para la app (Android 12 o superior).
- Quita el ahorro de batería a la app y permite el inicio automático (Xiaomi, Huawei, Samsung, Oppo… lo restringen por defecto).

---

## Copia de seguridad

**Ajustes › Datos › Copia de seguridad** copia el contenido al portapapeles y descarga un archivo `texar-organizer.json`. **Importar copia** lo carga de nuevo.

```json
{
  "events": [],
  "cats": [],
  "notes": [],
  "custom": [],
  "dated": [],
  "hist": []
}
```

| Clave | Contenido |
|---|---|
| `events` | Actividades del horario |
| `cats` | Categorías con su nombre y color |
| `notes` | Notas (con su formato) |
| `custom` | Festivos propios |
| `dated` | Eventos con fecha concreta |
| `hist` | Historial semanal para las estadísticas |

Las imágenes de fondo añadidas por el usuario **no** entran en la copia de seguridad. En la versión de móvil, ten en cuenta que **desinstalar la app borra sus datos**: haz una copia antes.

---

## Privacidad

- No hay cuentas, registro ni anuncios.
- Todos los datos se guardan en el dispositivo y no se envían a ningún servidor.
- Las únicas conexiones a internet son opcionales: cargar la tipografía de Google Fonts y descargar los festivos de años distintos de 2026 desde Nager.Date.

---

## Limitaciones conocidas

- **Sin sincronización entre dispositivos.** Cada instalación guarda sus datos por separado; se pasan a mano con la copia de seguridad.
- En **Android** los avisos dependen del sistema: algunos fabricantes cancelan las alarmas de apps en segundo plano. Ver los consejos de [Android](#android).
- **Marcar como hecho** solo se explica por semana: cada lunes se reinicia.
- Los **eventos con fecha** aparecen en el calendario y el panel lateral, pero **no generan aviso**.
- Los festivos de años distintos de 2026 **no son oficiales** y pueden no incluir los traslados a lunes de cada comunidad ni los festivos por isla de Canarias.
- No incluye los **festivos locales** de cada municipio (se pueden añadir a mano).
- Las fuentes se cargan de Google Fonts; sin conexión se usa una tipografía del sistema.
- El soporte de linux está sin probar.

---

## Créditos y licencia

- Creado por **joselitox**.
- Festivos de 2026: Boletín Oficial del Estado (Resolución de 17 de octubre de 2025, BOE-A-2025-21667).
- Festivos de otros años: [Nager.Date](https://date.nager.at/).
- Imágenes de fondo incluidas: aportadas por el autor. *Antes de publicar el repositorio, comprueba que tienes derecho a redistribuirlas.*
- Licencia: **por definir** (añade aquí la que elijas, por ejemplo MIT).
