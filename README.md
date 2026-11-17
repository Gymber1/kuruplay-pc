# KuruPlay PC

Programa de Windows para quienes apoyan KuruPlay: interfaz propia, reproductor sin anuncios, descargas para ver sin internet, exportar a MP4 y avisos de episodios nuevos.

Esta carpeta está fuera de git (`.gitignore`) y fuera del zip de Hostinger.

## Cómo funciona

- **Acceso:** solo cuentas con «KuruPlay PC» activo (Panel → Usuarios → ficha → *KuruPlay PC*: 1 mes, 3 meses, 1 año o permanente) y los administradores.
- **Iniciar sesión:** el programa pide un código a `/api/app/conexion` y abre el navegador en `/app/conectar/{código}`. Allí se entra con Google, Discord, Steam o Twitch y se pulsa «Conectar este equipo». El programa recibe una llave propia, que se guarda cifrada con DPAPI en `%APPDATA%\KuruPlay\sesion.dat`. Cada cuenta admite 3 equipos, que se desconectan desde el perfil o desde el panel.
- **API:** `routes/app.php` → `AppPcController` (inicio, catálogo, ficha, episodio, lista, progreso y ajustes). La lista y el progreso son los mismos de la web.
- **Descargas:** solo de los servidores HLS (Servidor 1-3). Los trozos `.ts` se guardan en `%APPDATA%\KuruPlay\descargas\<serie>-e<n>-<audio>\` junto a una `index.m3u8` local. Se reproducen con `kuru://descargas/…`. Si los enlaces firmados caducan, se piden nuevos a la API. **Exportar** los une con ffmpeg, sin recodificar, en `Descargas\<Título> - Episodio N.mp4`.
- **Sin conexión:** se puede entrar hasta 7 días desde la última comprobación y ver lo descargado.
- **Manga:** botón Anime / Manga en el lateral. `AppMangaController` (`/api/app/manga/inicio`, `catalogo`, `obra/{id}`, `capitulo/{id}`) usa la misma API de manga que la web. Lector en cascada o por páginas, calidad alta o ligera, ancho S/M/L y teclado (← →, N/P, Esc). Si una página no carga del nodo de MangaDex se pide por `/manga/pagina` de la web. Lo leído, el historial y «Mi lista» de manga se guardan en este equipo (localStorage). Los capítulos de editorial (MangaPlus…) se abren en el navegador.
- **Descubrir:** carga sola la página siguiente al acercarse al final (anime y manga). Filtros como la web: letra, tipo, géneros, año, estado y orden (`/api/app/catalogo`).
- **Horario:** `/api/app/horario`, los siete días con hora (12 h) y siguiente episodio.
- **Actualizaciones:** desde GitHub Releases del repositorio público `kuruplay.repoActualizaciones` de package.json (solo instaladores, sin código). Al abrir, si hay versión nueva sale una ventana para actualizar; también en Ajustes. Descarga con progreso, comprueba tamaño y SHA-256 y ejecuta el instalador en silencio, que vuelve a abrir el programa. Publicar: subir `version` en package.json, escribir `NOVEDADES.md`, `npm run instalador` y `npm run publicar` (con `$env:GITHUB_TOKEN`), o crear la release a mano en GitHub con la etiqueta `v<versión>` y el `.exe`.
- **Caché (`src/main/cache.js`):** las portadas se piden como `kuru://img/?u=<url>`; la primera vez se guardan en `%APPDATA%\KuruPlay\almacen\imagenes` (tope de 400 MB, se borran las más antiguas) y después salen del disco. Las consultas GET de `apiCache(ruta, segundos)` se guardan en `almacen\datos` y no se repiten hasta caducar, aunque se cierre el programa; sin internet se sirve la última copia. Las de más de 7 días se borran. La caché de Chromium (vídeo ya reproducido) tiene tope de 500 MB. Ojo: la carpeta no puede llamarse `cache`, porque en Windows choca con la `Cache` de Chromium y se vacía al arrancar. En Ajustes → Almacenamiento se ve cuánto ocupa cada cosa y se libera.
- **Avisos:** cada 15 minutos mira los últimos episodios; si uno es de una serie de Mi lista, lanza una notificación de Windows.

## Probar

```powershell
npm install
npm run local      # contra https://kuruplay.org (Laragon encendido y migración aplicada)
npm start          # contra https://kuruplay.online
```

En la web local, dale acceso a tu cuenta desde el panel (los administradores ya lo tienen).

Si Electron se instala sin `electron.exe` (Node 26 no lo descomprime), descomprime a mano el zip de `%LOCALAPPDATA%\electron\Cache` en `node_modules\electron\dist` y escribe `electron.exe` en `node_modules\electron\path.txt`.

Variables solo para pruebas:
- `KURUPLAY_DATOS=<carpeta>` guarda los datos y las exportaciones en otra carpeta.
- `KURUPLAY_SIN_NAVEGADOR=1` evita que se abra el navegador al iniciar sesión.

## Compilar el instalador

```powershell
npm run instalador
```

1. `scripts/empaquetar.mjs` monta `dist/KuruPlay/`: el Electron renombrado a `KuruPlay.exe`, con el icono y los datos puestos con rcedit, solo los archivos necesarios y solo los idiomas español e inglés.
2. `scripts/instalador.iss` usa Inno Setup 6 para generar `dist/KuruPlay-Setup-<versión>-x64.exe`. El instalador sale en español (permite elegir idioma), se instala por usuario en `%LOCALAPPDATA%\Programs\KuruPlay` sin pedir permisos de administrador, crea el acceso directo en el escritorio y ofrece «Abrir KuruPlay» al terminar. Al desinstalar pregunta si borrar también las descargas.

## Publicar una versión nueva

1. Sube `version` en `package.json`, escribe `NOVEDADES.md` y ejecuta `npm run instalador`.
2. Publica la release en GitHub (`Gymber1/kuruplay-pc`): `npm run publicar` con `$env:GITHUB_TOKEN`, o a mano con la etiqueta `v<versión>`, las notas de `NOVEDADES.md` y el `.exe`. Que quede marcada como **Latest**: es la que leen los programas instalados.
3. Para avisar a quien siga en la 1.0.0 o la 1.1.0 (comprobaban `/api/app/version` de la web), en el `.env` del servidor pon `KURUPLAY_PC_VERSION=<versión>` y `KURUPLAY_PC_URL=https://github.com/Gymber1/kuruplay-pc/releases/latest`, y ejecuta `php artisan config:cache`.
