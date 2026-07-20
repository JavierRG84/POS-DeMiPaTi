# Demipati POS Offline — APK

Empaqueta el POS Offline (`DemipatiPOS-Offline.html`) como APK Android usando Capacitor,
compilado automáticamente por GitHub Actions (sin necesidad de Android Studio local).

La app se llama **POS**, usa el logo adjunto como ícono (legacy + adaptativo + splash,
ya generados en `resources/`) y un tema visual azul/rojo/blanco con bordes redondeados
(`www/theme.css`).

## Pasos para dejarlo funcionando

1. **Crear el repo en GitHub**
   - Nuevo repositorio (puede ser privado), por ejemplo `demipati-pos-apk`.
   - Clonarlo localmente o subir estos archivos directo por la web de GitHub.

2. **Copiar los archivos del POS offline dentro de `www/`**
   - Copia `DemipatiPOS-Offline.html` y renómbralo a `www/index.html`.
   - Copia `manifest-offline.json` a `www/manifest.json`.
   - Copia `sw-offline.js` a `www/sw.js`.
   - Copia cualquier ícono/imagen que use el manifest (íconos PWA) a `www/`.
   - Revisa que las rutas dentro del HTML a `ajax.php`, `pos-scanner.js`, `pos-printer.js`
     apunten a la URL completa `https://www.demipati.cu/gestion/custom/demipatipos/pos/...`
     (dentro de un APK ya no hay "mismo dominio", así que las llamadas a la API deben ser
     absolutas, no relativas).
   - Agrega dentro del `<head>` del `index.html` una línea para cargar el tema:
     `<link rel="stylesheet" href="theme.css">`
     (el archivo ya está en `www/theme.css`, con la paleta azul/rojo/blanco y bordes
     redondeados aplicados a botones, cards, inputs, modales y encabezado — solo hace
     falta que las clases coincidan con las que ya usa tu HTML, o ajustamos los
     selectores juntos si usan otros nombres).

3. **Subir todo al repo**: `package.json`, `capacitor.config.json`,
   `.github/workflows/build-apk.yml`, `.gitignore`, la carpeta `resources/` (ícono y
   splash ya generados desde el logo) y la carpeta `www/` con tus archivos.

4. **(Automático) Generación de íconos y splash**
   - El workflow corre `npx capacitor-assets generate --android`, que toma
     `resources/icon.png` (ícono clásico), `resources/icon-foreground.png` +
     `resources/icon-background.png` (ícono adaptativo redondeado de Android) y
     `resources/splash.png`, y genera automáticamente todas las resoluciones que
     necesita Android. No hay que tocar nada de esto manualmente.

5. **Ejecutar el workflow**
   - Cualquier `git push` a `main` dispara la compilación automáticamente.
   - También puedes ir a la pestaña **Actions** del repo y correrlo manualmente
     ("Run workflow" — está habilitado con `workflow_dispatch`).

6. **Descargar el APK**
   - Al terminar el workflow (5-8 min aprox.), entra a la ejecución en **Actions**.
   - Al final de la página hay un artefacto llamado `demipati-pos-offline-debug` — descárgalo,
     es un `.zip` que contiene el `app-debug.apk`.
   - Ese APK es de **debug**, sirve para instalar y probar directo (Android pedirá habilitar
     "orígenes desconocidos"), pero no está firmado para Play Store.

## Firmar el APK para distribución "real" (opcional, más adelante)

Si más adelante quieres un APK firmado (release) en vez de debug, se necesita:
- Generar un keystore (`keytool -genkey ...`).
- Subir el keystore y su contraseña como *GitHub Secrets*.
- Ajustar el workflow para correr `./gradlew assembleRelease` firmando con esos secrets.

Esto es opcional — para uso interno el APK debug funciona perfectamente igual que el de TRAZA.

## Notas

- El nombre "POS" sale de `capacitor.config.json` (`appName`). El ícono y splash ya
  están generados desde tu logo real en `resources/`; si más adelante quieres cambiar
  el logo, solo hay que reemplazar esos 4 archivos y el workflow los vuelve a procesar.
- Si el POS offline usa el scanner de cámara (BarcodeDetector) o Bluetooth para la
  impresora ESC/POS, dentro de un WebView de Capacitor puede que se necesiten permisos
  nativos adicionales (cámara, Bluetooth) en `AndroidManifest.xml` — lo ajustamos cuando
  lleguemos a esa prueba.
