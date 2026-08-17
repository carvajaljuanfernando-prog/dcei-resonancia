# DCEI &amp; Resonancia Magnética

Asistente de apoyo a la decisión para portadores de **dispositivos cardíacos electrónicos implantables** (marcapasos, desfibriladores, resincronizadores, DAI subcutáneos, marcapasos sin cables y monitores implantables).

Resuelve dos preguntas clínicas distintas:

1. **¿Este paciente puede entrar a resonancia magnética, y bajo qué condiciones?**
2. **¿Qué hay que hacer con el dispositivo si va a cirugía o a un procedimiento con electrocauterio?**

Es una aplicación estática de un solo archivo. No tiene backend, no envía datos a ningún servidor propio y funciona sin conexión una vez instalada.

---

## Aviso clínico

Esta herramienta **orienta y genera el checklist**. No sustituye el manual técnico del fabricante, la verificación del número de modelo en el buscador oficial, ni la valoración del electrofisiólogo y del radiólogo responsables. Cada veredicto enlaza a la fuente del fabricante correspondiente y esa verificación es obligatoria antes de escanear.

Ningún marcapasos ni desfibrilador puede etiquetarse como *MR Safe*: los dispositivos activos implantables sólo pueden ser *MR Conditional* o *MR Unsafe*. «Condicional» nunca significa «seguro sin condiciones».

---

## Publicar en GitHub Pages

### Opción A — desde la interfaz web (sin línea de comandos)

1. Cree un repositorio nuevo en GitHub, por ejemplo `dcei-resonancia`. Puede ser público o privado, pero **GitHub Pages en repositorios privados requiere plan de pago**: para una cuenta gratuita, el repositorio debe ser público.
2. En el repositorio: **Add file → Upload files**, y arrastre **todo el contenido de esta carpeta** (no la carpeta en sí). Incluya el archivo `.nojekyll`, que es invisible en algunos exploradores — si su sistema lo oculta, actíve la vista de archivos ocultos.
3. **Commit changes**.
4. Vaya a **Settings → Pages**. En *Build and deployment*, elija **Source: Deploy from a branch**, rama `main`, carpeta `/ (root)`. Guarde.
5. Espere entre uno y tres minutos. La URL será:

   ```
   https://SU-USUARIO.github.io/dcei-resonancia/
   ```

### Opción B — desde la terminal

```bash
cd dcei-resonancia
git init
git add -A
git commit -m "Asistente DCEI y resonancia magnética"
git branch -M main
git remote add origin https://github.com/SU-USUARIO/dcei-resonancia.git
git push -u origin main
```

Después active Pages en **Settings → Pages** como en el paso 4 de la opción A.

Si prefiere despliegue por GitHub Actions en lugar de «deploy from a branch», este repositorio ya incluye `.github/workflows/pages.yml`. En ese caso elija **Source: GitHub Actions** y no haga nada más.

---

## Probar en local antes de publicar

El service worker y la instalación como aplicación **no funcionan abriendo el archivo con doble clic** (`file://`). Hace falta un servidor. Cualquiera de estos sirve:

```bash
# Python (viene instalado en macOS y Linux)
python3 -m http.server 8080

# Node
npx serve -l 8080
```

Abra `http://localhost:8080`. Todo lo demás — búsqueda de dispositivos, motor de reglas, informe — sí funciona con doble clic sobre `index.html`, sólo se pierde el modo offline.

Para probar desde el celular en la misma red WiFi, use la IP del computador (`http://192.168.x.x:8080`). Tenga en cuenta que **la cámara y el service worker exigen HTTPS o localhost**: desde una IP local el navegador puede bloquear la cámara. Por eso conviene probar el flujo de foto ya publicado en GitHub Pages, que sirve por HTTPS.

---

## Instalación en el celular

Una vez publicada, abra la URL en el celular:

- **Android / Chrome:** aparece la barra «Instalar», o use el menú ⋮ → *Instalar aplicación*.
- **iPhone / Safari:** botón Compartir → *Añadir a pantalla de inicio*. En iOS la instalación sólo funciona desde Safari, no desde Chrome.

Instalada, abre a pantalla completa y **funciona sin señal**, que es la situación habitual dentro de una sala de resonancia. La extracción por IA sí necesita conexión; el resto no.

---

## Lectura del carné — gratuita y local

Es el método por defecto y **no necesita cuenta, clave ni conexión**. La foto se procesa dentro del navegador con Tesseract compilado a WebAssembly, que va incluido en la carpeta `vendor/`.

La clave del diseño es que el lector no necesita *entender* el carné. Extrae todo el texto que ve y lo compara contra los cientos de números de modelo de la base mediante una distancia de edición que penaliza poco las confusiones típicas del reconocimiento óptico: 0 con O, 1 con I, 5 con S, 8 con B, 6 con G. Es un problema de vocabulario cerrado, mucho más fácil que la lectura general. En las pruebas, leer `A3DRO1` con O en lugar de cero seguía proponiendo Advisa MRI como primera opción.

Los candidatos se presentan en tres bloques según el tipo de evidencia:

- **Número leído tal cual** — coincidencia exacta.
- **Parecidos** — con advertencia explícita, porque números vecinos pertenecen a familias distintas con condiciones de resonancia distintas.
- **Sólo por el nombre comercial** — sin número de modelo, que en varias familias no basta: hay versiones condicionales y no condicionales que sólo se distinguen por el número, y en el Sprint Quattro por la longitud del sufijo.

Cuando un número se lee exacto, la aplicación **descarta los parecidos nacidos de ese mismo texto**, para que un vecino no aparezca junto al acierto y se toque por error.

El motor pesa unos 7 MB y se descarga la primera vez que se usa el botón de lectura; después queda en la caché y funciona sin conexión. En **Ajustes** hay un botón para descargarlo por adelantado, antes de entrar a un sitio sin señal.

**Para que lea mejor:** toque la miniatura y recorte el bloque donde están los números —es lo que más mejora el resultado—, use luz difusa evitando el reflejo del plástico, apoye el carné plano y encuadre de frente, y tome una foto por carné en vez de una general.

### Identificar desde un texto

Si tiene el reporte de control en digital, un correo o un PDF, puede pegar el texto directamente. Usa el mismo emparejador, es instantáneo y no descarga nada.

---

## Extracción por IA (opcional, de pago)

Alternativa a la lectura local, más precisa con carnés difíciles, manuscritos o mal iluminados. La pestaña **Ajustes** permite pegar una clave de API de Anthropic.

- La clave se guarda **sólo en el navegador del usuario** (`localStorage`). No se sube al repositorio ni pasa por ningún servidor intermedio: la llamada va del navegador directo a `api.anthropic.com`.
- Cada usuario pone su propia clave. **Nunca escriba una clave dentro del código antes de publicar el repositorio**: quedaría expuesta a cualquiera.
- Si el modelo configurado deja de existir, cambie el nombre en Ajustes; el campo es editable a propósito.
- Sin clave la aplicación funciona completa, con búsqueda manual.

**Datos del paciente:** la imagen se envía al proveedor de IA. Fotografíe el bloque de modelos y evite incluir datos identificables que no necesite. Verifique la política de datos de su institución antes de usar esta función con pacientes reales.

---

## Actualizar la aplicación publicada

Al editar `index.html` hay que **subir también el número de versión en dos sitios**, o los usuarios que ya la tengan instalada seguirán viendo la versión antigua desde la caché:

1. `sw.js` → constante `VERSION`
2. `index.html` → constante `APP_VERSION` (se muestra en el pie de página)

Con la versión cambiada, quien tenga la aplicación abierta verá la barra «Hay una versión nueva disponible».

---

## Contenido del repositorio

| Archivo | Función |
|---|---|
| `index.html` | La aplicación completa: interfaz, base de datos y motores de reglas |
| `vendor/tesseract/` | Motor de reconocimiento óptico Tesseract (WebAssembly), licencia Apache 2.0 |
| `vendor/lang/eng.traineddata.gz` | Modelo de idioma del reconocedor |
| `sw.js` | Service worker: caché offline |
| `manifest.webmanifest` | Metadatos de la aplicación instalable |
| `icon.svg`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `favicon-32.png` | Iconos |
| `.nojekyll` | Evita que GitHub Pages procese el sitio con Jekyll |
| `.github/workflows/pages.yml` | Despliegue automático, alternativa a «deploy from a branch» |

---

## Qué hace el motor de resonancia

- Razona sobre el **sistema completo**, no sobre el generador: un generador condicional con un electrodo no condicional no es un sistema condicional.
- Aplica la **regla del eslabón más débil** para el campo magnético: el campo del sistema es el mínimo entre generador y todos los electrodos.
- Clasifica en las **categorías 1, 2 y 3** del consenso SCMR 2024, más contraindicación e indeterminado.
- Detecta los descalificadores del etiquetado: electrodo abandonado, fracturado, epicárdico, array subcutáneo abandonado, adaptador o extensor, puerto vacío sin tapón, generador abdominal, componentes de fabricantes distintos, batería en ERI o EOL, y menos de seis semanas desde el implante.
- El **electrodo transvenoso temporal** dispara contraindicación absoluta.
- Genera el protocolo paso a paso: interrogación previa, programación según dependencia, personal requerido, monitoría, límites técnicos y umbrales de cambio post-estudio.

## Qué hace el motor periquirúrgico

Implementa la distinción que causa más errores graves: **el imán sobre un marcapasos fuerza estimulación asincrónica; sobre un desfibrilador sólo suspende las terapias, sin alterar el modo ni la frecuencia**.

Escala a «reprogramación obligatoria» cuando el imán no es una vía válida: paciente dependiente con DAI, marcapasos Biotronik en modo auto (que da unos diez latidos y revierte aunque el imán siga puesto), Micra de Medtronic (que no responde al imán) y generador inaccesible en decúbito prono o dentro del campo quirúrgico.

Cubre además ablación por radiofrecuencia, litotricia, terapia electroconvulsiva y radioterapia, y el escenario de urgencia sin información del dispositivo.

---

## Alcance de la base de datos

Construida por **familias de producto** de los cinco fabricantes — Medtronic, Boston Scientific, Abbott, Biotronik y MicroPort CRM. Cubre la mayoría de los casos reales y es mantenible. Las referencias poco frecuentes se marcan deliberadamente como «no verificado» en lugar de recibir un veredicto inventado.

Las limitaciones y los datos que no se pudieron confirmar en fuente primaria están declarados dentro de la propia aplicación, en la pestaña **Ajustes**. Las listas de modelos son **regionales**: la lista europea de Boston Scientific incluye familias que no están en la matriz estadounidense.

---

## Fuentes

Consenso HRS 2017 · Consenso SCMR 2024 · ACC 2024 · HRS 2025 call to action · ISMRM/JMRI 2021 · CMS NCD 220.2 · FDA · MagnaSafe (NEJM 2017) · Nazarian (NEJM 2017) · Schaller (JAMA Cardiology 2021) · meta-análisis Europace 2024 · HRS/ASA 2011 · ASA 2020 · EHRA 2022 · AHA 2024 · Heart Rhythm New Zealand 2024 · BHRS 2019 · NHS Lothian 2023 · documentación primaria de los cinco fabricantes.

El listado completo con enlaces está en la pestaña **Datos** de la aplicación.

---

## Componentes de terceros

El motor de lectura óptica va incluido en `vendor/`, con sus licencias originales:

| Componente | Licencia |
|---|---|
| tesseract.js | MIT |
| tesseract.js-core (WebAssembly) | Apache 2.0 — texto en `vendor/tesseract/LICENSE-core` |
| `eng.traineddata` (modelo de idioma de Tesseract) | Apache 2.0 |

Todas permiten la redistribución dentro de este repositorio. No hay ninguna otra dependencia: la aplicación no carga nada desde CDN ni desde servidores externos.

## Licencia

MIT — ver `LICENSE`. La licencia cubre el código, no constituye respaldo clínico ni transfiere responsabilidad sobre las decisiones tomadas con la herramienta.
