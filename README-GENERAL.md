# QARANA – Cada alimento cuenta

> Plataforma web para conectar comedores populares con donantes de alimentos, facilitar solicitudes y apoyar la gestión de inventario, nutrición y planificación de comidas.

## 1. Información del proyecto

- **Nombre del equipo:** IA BOYS
- **Integrantes:** GAMBOA CORDOVA ALESSANDRO MIJAEL
                   GONZALES MAYHUIRE FRANCO AARON
                   LOPEZ LUCERO CRISTOFER JARED
- **Solución:** QARANA – Alimentación solidaria
- **ODS abordado:** **ODS 2 – Hambre cero**
- **Lugar de impacto:** Villa El Salvador, Lima, Perú.

### Descripción de la solución

QARANA es una plataforma web que busca reducir el desperdicio de alimentos y mejorar su distribución. Conecta a donantes con comedores populares para que los alimentos disponibles puedan publicarse, solicitarse y registrarse como entregas. También permite gestionar comedores, donantes, catálogo de alimentos, inventario, información nutricional y planificación de preparaciones y comidas.

La solución apunta a facilitar la coordinación entre organizaciones y contribuir al aprovechamiento de alimentos en la comunidad.

## 2. Arquitectura del sistema

El proyecto utiliza tres componentes principales:

```text
┌──────────────────────┐
│   GOOGLE SHEETS      │
│ Base de datos        │
│ (hojas y registros)  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ GOOGLE APPS SCRIPT   │
│ Backend / API        │
│ Lógica y validaciones│
└──────────┬───────────┘
           │ URL pública /exec
           │
           ▼
┌──────────────────────┐
│ GITHUB PAGES         │
│ index.html           │
│ Interfaz web         │
└──────────────────────┘
```

**Flujo de funcionamiento:**

1. El usuario interactúa con la página `index.html`, publicada mediante GitHub Pages.
2. El frontend envía las operaciones a Google Apps Script mediante la URL de la aplicación web terminada en `/exec`.
3. Apps Script valida la solicitud, ejecuta la operación correspondiente y consulta o actualiza Google Sheets.
4. La API devuelve el resultado al frontend para mostrarlo en pantalla.

> Google Sheets funciona como base de datos; Google Apps Script funciona como backend/API; GitHub Pages publica la interfaz. No es necesario instalar un servidor local para utilizar la versión publicada.

## 3. Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| HTML5 | Estructura de la interfaz |
| CSS3 | Diseño adaptable y estilos visuales |
| JavaScript | Interacciones y comunicación con la API |
| Chart.js | Gráficos del panel |
| Google Sheets | Almacenamiento de datos |
| Google Apps Script | Backend, API, autenticación y reglas del sistema |
| GitHub | Repositorio y control de versiones |
| GitHub Pages | Publicación del frontend |

## 4. Requisitos previos

Para instalar una instancia propia de QARANA se necesita:

- Una cuenta de Google.
- Acceso a Google Drive, Google Sheets y Google Apps Script.
- Una cuenta de GitHub.
- Un navegador actualizado (Chrome, Edge o Firefox).
- Los archivos del proyecto: `index.html` y el código de Apps Script (`Code.gs` o el archivo `.gs` entregado).

**Importante:** si solo deseas utilizar la instancia que ya está publicada, abre su enlace de GitHub Pages. No necesitas repetir la instalación ni crear otra base de datos. Los pasos siguientes son para desplegar una instancia independiente en otra cuenta/equipo.

## 5. Instalación en otra PC o cuenta

La aplicación no se instala como un programa de Windows. Se configura una copia en la nube y luego se accede desde el navegador. Para que la nueva instalación tenga datos independientes, crea una hoja de cálculo y un despliegue de Apps Script propios.

### Paso 1. Crear la base de datos en Google Sheets

1. Inicia sesión en la cuenta de Google que administrará la nueva instalación.
2. Entra a [Google Sheets](https://sheets.google.com).
3. Crea una hoja de cálculo en blanco.
4. Ponle un nombre identificable, por ejemplo: `QARANA - Base de datos`.
5. Conserva la hoja abierta; desde ella se creará el proyecto de Apps Script.

El backend crea/verifica automáticamente las siguientes pestañas y encabezados:

| Hoja | Contenido |
|---|---|
| `USUARIOS` | Cuentas, roles y datos de autenticación |
| `COMEDORES` | Comedores populares y sus responsables |
| `DONANTES` | Personas u organizaciones donantes |
| `ALIMENTOS` | Catálogo de alimentos |
| `DONACIONES` | Alimentos publicados por los donantes |
| `SOLICITUDES` | Solicitudes realizadas por los comedores |
| `MOVIMIENTOS` | Entradas y salidas del inventario |
| `NUTRICION` | Información nutricional de los alimentos |
| `PREPARACIONES` | Preparaciones y sus ingredientes |
| `CALENDARIO` | Planificación de comidas |
| `AUDITORIA` | Registro de acciones realizadas en el sistema |

No cambies manualmente los nombres de estas pestañas ni sus encabezados. La función de configuración puede crear las pestañas que falten y completar encabezados faltantes.

### Paso 2. Instalar el backend en Google Apps Script

1. Desde la hoja de cálculo, selecciona **Extensiones → Apps Script**.
2. Se abrirá el editor de Google Apps Script.
3. Borra el código de ejemplo si lo hubiera.
4. Copia el contenido completo del archivo backend `.gs` entregado por el equipo y pégalo en el editor.
5. Guarda el proyecto con un nombre como `QARANA API`.

**Vincular el script a la hoja correcta:** abre Apps Script desde la hoja de cálculo creada en el paso 1. Así, `SpreadsheetApp.getActiveSpreadsheet()` trabajará con esa base de datos.

### Paso 3. Crear las hojas y el administrador inicial

1. En el editor de Apps Script, selecciona la función `configurarSistema`.
2. Presiona **Ejecutar**.
3. Autoriza los permisos solicitados por Google. Revisa que estás ejecutando el proyecto de tu propia cuenta.
4. Comprueba que se hayan creado las 11 pestañas indicadas en el paso 1.
5. Para crear la primera cuenta, selecciona la función `crearAdminQarana` y ejecútala con tus propios datos desde el editor.

Ejemplo de llamada para crear el administrador:

```javascript
crearAdminQarana(
  "tu-correo@gmail.com",
  "UnaClaveSeguraDe10CaracteresOMas",
  "Nombre del administrador"
);
```

- Utiliza un correo al que tengas acceso.
- La contraseña debe tener **como mínimo 10 caracteres**.
- No uses la contraseña del ejemplo: reemplázala por una propia.
- La cuenta inicial se crea con el rol `ADMIN`.
- Ejecuta esta creación una sola vez. Si ya existe una cuenta con ese correo, el sistema lo indicará.

### Paso 4. Publicar Apps Script como API

1. En Apps Script, selecciona **Implementar → Nueva implementación**.
2. En el tipo de implementación, elige **Aplicación web**.
3. Configura:
   - **Ejecutar como:** tu cuenta.
   - **Quién tiene acceso:** cualquier usuario que necesite acceder a la aplicación, según las opciones disponibles en tu cuenta. Para esta arquitectura, la API debe poder recibir solicitudes desde GitHub Pages.
4. Presiona **Implementar** y autoriza si Google lo solicita.
5. Copia la URL de la aplicación web. Debe terminar en `/exec`.

Ejemplo de formato (no uses esta dirección como si fuera tuya):

```text
https://script.google.com/macros/s/ID_DE_TU_IMPLEMENTACION/exec
```

**Cada vez que modifiques el backend:** guarda los cambios y actualiza la implementación existente desde **Implementar → Administrar implementaciones → Editar → Nueva versión → Implementar**. Si creas otra implementación, copia su nueva URL y actualiza también el frontend.

### Paso 5. Configurar el origen permitido

El backend incluye una validación del origen de la página que puede llamar a la API.

- De forma predeterminada permite páginas HTTPS de `github.io` y `localhost`.
- Si deseas restringir el acceso a un origen concreto, en Apps Script entra a **Configuración del proyecto → Propiedades del script** y crea la propiedad:

| Propiedad | Valor |
|---|---|
| `QARANA_ALLOWED_ORIGINS` | `https://TU-USUARIO.github.io` |

Sustituye `TU-USUARIO` por el usuario real de GitHub. Si tu página se publica en un dominio personalizado, registra el origen exacto, incluyendo `https://` y sin la ruta del proyecto.

Si necesitas permitir más de un origen, sepáralos con comas. Por ejemplo:

```text
https://tu-usuario.github.io,http://localhost:5500
```

No agregues rutas como `/qarana/` a esta propiedad: se registra el origen, no la dirección completa de la página.

### Paso 6. Configurar el frontend (`index.html`)

1. Abre `index.html` en Visual Studio Code o en un editor de texto.
2. Busca la constante `API_URL`.
3. Reemplaza su valor por la URL `/exec` de tu propia implementación de Apps Script.

Fragmento que debes localizar:

```javascript
const API_URL = 'https://script.google.com/macros/s/ID_DE_TU_IMPLEMENTACION/exec';
```

4. Guarda el archivo.
5. Verifica que la URL corresponda a la misma hoja de cálculo donde ejecutaste `configurarSistema` y creaste el administrador.

**No compartas la URL de despliegue en documentos públicos si no es necesario.** La URL no sustituye la autenticación: los usuarios deben iniciar sesión en QARANA.

### Paso 7. Publicar el frontend en GitHub Pages

Si el repositorio ya existe:

1. Entra al repositorio del proyecto en GitHub.
2. Abre el archivo `index.html`.
3. Selecciona **Edit** (ícono de lápiz).
4. Pega o sube la versión actualizada que contiene tu `API_URL`.
5. Guarda los cambios con **Commit changes**.

Si vas a crear un repositorio nuevo:

1. En GitHub, selecciona **New repository**.
2. Escribe un nombre, por ejemplo `qarana`.
3. Selecciona la visibilidad que corresponda al proyecto.
4. Crea el repositorio.
5. Sube `index.html` a la raíz del repositorio.
6. Entra a **Settings → Pages**.
7. En **Build and deployment**, selecciona **Deploy from a branch**.
8. Elige la rama `main` y la carpeta `/ (root)`.
9. Guarda la configuración y espera a que GitHub Pages publique el sitio.

El enlace tendrá un formato similar a:

```text
https://TU-USUARIO.github.io/qarana/
```

La publicación puede tardar unos minutos. Abre el enlace en el navegador cuando GitHub Pages indique que el sitio está publicado.

## 6. Cómo utilizar QARANA

### 6.1 Iniciar sesión

1. Abre el enlace de QARANA publicado en GitHub Pages.
2. Escribe el correo y la contraseña del usuario creado por el administrador.
3. Presiona **Iniciar sesión**.
4. El sistema mostrará las opciones habilitadas según el rol.

No existen credenciales predeterminadas. El administrador inicial debe crear las cuentas de los demás usuarios desde la aplicación.

### 6.2 Roles de usuario

| Rol | Función |
|---|---|
| `ADMIN` | Administra usuarios, comedores, donantes, alimentos y la información general del sistema. |
| `COMEDOR` | Consulta el catálogo y gestiona las operaciones disponibles para su propio comedor. |
| `DONANTE` | Consulta el catálogo y gestiona las operaciones disponibles para su perfil de donante. |

El backend verifica los permisos de las operaciones. Que una opción no aparezca en pantalla no reemplaza la validación de permisos del servidor.

### 6.3 Módulos principales

- **Panel principal:** resumen de la actividad y datos generales.
- **Comedores:** registro y consulta de comedores populares.
- **Donantes:** registro y consulta de donantes.
- **Alimentos:** catálogo de alimentos disponibles.
- **Donaciones:** publicación y seguimiento de donaciones.
- **Solicitudes:** consulta y gestión de solicitudes de alimentos.
- **Inventario:** seguimiento de movimientos y existencias.
- **Nutrición:** consulta de información nutricional y gestión de preparaciones.
- **Calendario:** planificación de comidas.
- **Reportes:** consulta de indicadores y resúmenes.
- **Auditoría:** revisión de acciones registradas.

Las opciones visibles y las acciones permitidas dependen del rol con el que se inició sesión.

### 6.4 Flujo básico de una donación

1. El administrador registra una cuenta y los datos del donante.
2. El donante publica una donación indicando alimento, cantidad y datos de recojo.
3. El comedor consulta las donaciones disponibles.
4. El comedor registra una solicitud por la cantidad que necesita.
5. La solicitud se revisa y se actualiza según el flujo de atención.
6. Al confirmarse una entrega, el sistema registra el movimiento correspondiente en el inventario.

Registra información real y verifica cantidades, unidades y fechas antes de confirmar una operación.

### 6.5 Cerrar sesión

Utiliza la opción **Cerrar sesión** del menú. El sistema finaliza el token de sesión y elimina el token guardado en la sesión del navegador.

## 7. Uso desde otra computadora

Una vez publicada la aplicación:

1. En la otra computadora, abre Chrome, Edge o Firefox.
2. Ingresa al enlace de GitHub Pages.
3. Inicia sesión con una cuenta habilitada.
4. Utiliza el sistema desde el navegador.

No es necesario instalar Visual Studio Code, Python, Node.js ni descargar el código para el uso cotidiano.

**Importante:** si varias computadoras utilizan la misma URL de GitHub Pages y la misma API, estarán trabajando sobre la misma base de datos de Google Sheets. Para tener datos separados, hay que crear otra hoja, desplegar otra API y configurar otra URL en el frontend.

## 8. Solución de problemas

| Problema | Qué revisar |
|---|---|
| La página no abre | Verifica que GitHub Pages esté habilitado y que el enlace sea correcto. |
| La aplicación indica que la API no respondió | Comprueba que la URL `API_URL` termine en `/exec` y que la implementación esté activa. |
| Aparece “Origen no autorizado” | Revisa `QARANA_ALLOWED_ORIGINS` y registra el origen exacto de GitHub Pages. |
| Aparece “No existe la hoja…” | Ejecuta `configurarSistema` desde el proyecto de Apps Script vinculado a la hoja correcta. |
| No permite iniciar sesión | Comprueba el correo, la contraseña y que el usuario esté activo. Si es una instalación nueva, crea el administrador inicial. |
| Los cambios del backend no aparecen | Guarda el código y publica una nueva versión de la aplicación web. |
| Se muestra información de otra instalación | Revisa que `API_URL` apunte al despliegue asociado a la hoja de cálculo que corresponde. |

## 9. Seguridad y buenas prácticas

- No publiques contraseñas, hashes, salts ni datos privados en el repositorio.
- No guardes credenciales reales dentro de `index.html`.
- Utiliza contraseñas de al menos 10 caracteres.
- Comparte el acceso a la hoja de cálculo solo con personas autorizadas.
- No elimines ni renombres las pestañas o encabezados que utiliza el backend.
- Haz una copia de seguridad periódica de la hoja de cálculo.
- Antes de trabajar con datos reales, prueba el flujo completo con registros de demostración.
- Si se cambia el proyecto de Google o la hoja de cálculo, vuelve a desplegar la API y actualiza `API_URL`.

## 10. Alcance de la instalación

Esta guía describe la instalación de una instancia propia con Google Sheets, Google Apps Script y GitHub Pages. La disponibilidad de la aplicación depende de que la cuenta de Google, la hoja de cálculo, la implementación de Apps Script y el repositorio publicado continúen activos.

El archivo Excel de demostración no sustituye la configuración del backend. Para una instalación limpia, se recomienda crear una hoja de cálculo nueva y ejecutar `configurarSistema`. Si se decide importar datos de una plantilla, deben revisarse previamente los encabezados, las relaciones entre identificadores y los registros de prueba.

---

**QARANA – Cada alimento cuenta.**  
*Conectar corazones y compartir alimentos.*

DIOS LES BENDIGA---- HASTA LA PROXIMA---- FILIPENSES 3:14
