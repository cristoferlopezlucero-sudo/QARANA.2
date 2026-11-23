# QARANA – Documentación del backend (`Code.gs`)

Este documento complementa el [`README.md`](./README.md) principal. Explica qué hace el archivo `Code.gs`, cómo configurarlo, qué hojas administra, cómo funciona la API y qué operaciones puede solicitar el frontend `index.html`.

> **Arquitectura:** `Google Sheets (datos) → Google Apps Script / Code.gs (backend y API) → index.html (GitHub Pages)`

---

## 1. ¿Qué es `Code.gs`?

`Code.gs` es el backend de QARANA. Se ejecuta en los servidores de Google Apps Script y conecta la interfaz web con la hoja de cálculo.

Sus responsabilidades son:

- Crear y verificar la estructura de la base de datos.
- Registrar e identificar usuarios.
- Controlar sesiones, permisos y roles.
- Validar los datos antes de guardarlos.
- Gestionar comedores, donantes, alimentos y donaciones.
- Procesar solicitudes y registrar entregas.
- Calcular inventarios a partir de sus movimientos.
- Administrar información nutricional, preparaciones y calendario alimentario.
- Registrar acciones importantes en la hoja de auditoría.
- Recibir solicitudes HTTP del frontend y devolver respuestas.

**No se debe abrir `Code.gs` en el navegador ni ejecutarlo en la computadora del usuario.** Se instala una vez en Google Apps Script y queda publicado como aplicación web.

## 2. Requisitos

- Una cuenta de Google con acceso a Google Sheets y Google Apps Script.
- Una hoja de cálculo destinada a QARANA.
- El archivo completo `Code.gs`.
- El `index.html` publicado en GitHub Pages.
- La URL de implementación de Apps Script que termina en `/exec`.

## 3. Instalación y configuración inicial

### 3.1 Crear o seleccionar la hoja de cálculo

1. Inicia sesión en Google.
2. Crea una hoja de cálculo nueva o abre una hoja vacía destinada a QARANA.
3. Desde esa misma hoja, entra a **Extensiones → Apps Script**.
4. Se abrirá el editor de Apps Script vinculado a la hoja.

Es importante abrir Apps Script desde la hoja correcta: el código utiliza `SpreadsheetApp.getActiveSpreadsheet()` para localizar la base de datos.

### 3.2 Copiar el código

1. Abre el archivo `Code.gs` entregado con el proyecto.
2. Copia todo su contenido.
3. Pégalo en el editor de Apps Script.
4. Guarda el proyecto, por ejemplo, con el nombre `QARANA - Backend`.

No copies fragmentos aislados: el archivo contiene funciones relacionadas entre sí y debe instalarse completo.

### 3.3 Crear las hojas automáticamente

En el selector de funciones del editor:

1. Selecciona `configurarSistema`.
2. Presiona **Ejecutar**.
3. Acepta los permisos solicitados por Google.
4. Regresa a la hoja de cálculo y verifica que aparezcan las pestañas de QARANA.

La función crea las hojas que no existan y coloca los encabezados. Si una hoja ya contiene información, intenta conservarla y añadir encabezados que falten. **No borra los registros existentes.**

### 3.4 Crear el primer administrador

El primer administrador se crea manualmente desde Apps Script. En el editor puedes ejecutar la función `crearAdminQarana` con tus propios datos:

```javascript
crearAdminQarana(
  "administrador@correo.com",
  "UnaContrasenaSegura123",
  "Nombre del administrador"
);
```

También existe la función interna `crearAdministradorInicial(correo, contrasena, nombre)`.

Condiciones:

- Usa un correo real al que tengas acceso.
- La contraseña debe tener al menos **10 caracteres**.
- Reemplaza los datos de ejemplo por los tuyos.
- No hay un correo ni una contraseña predeterminados.
- Ejecuta la creación del administrador inicial una sola vez. Si el correo ya está registrado, el sistema no crea un duplicado.

La contraseña no se guarda como texto legible: el backend genera un salt y un hash SHA-256 para almacenarlos en `USUARIOS`.

### 3.5 Publicar como aplicación web (API)

1. En Apps Script, selecciona **Implementar → Nueva implementación**.
2. En tipo, elige **Aplicación web**.
3. Configura **Ejecutar como: yo** (la cuenta propietaria del script).
4. Configura el acceso para que los usuarios de la página publicada puedan enviar solicitudes, de acuerdo con las opciones que permita la cuenta de Google.
5. Presiona **Implementar** y concede los permisos necesarios.
6. Copia la URL de la aplicación web. Debe terminar en `/exec`.

La URL `/exec` es el endpoint que utiliza el frontend.

### 3.6 Configurar el origen permitido

El backend valida desde qué sitio web se reciben solicitudes.

Por defecto admite:
- Sitios HTTPS cuyo dominio sea `usuario.github.io`.
- `http://localhost` y puertos locales, para pruebas de desarrollo.

Para restringir el acceso a un origen concreto:

1. En Apps Script, abre **Configuración del proyecto**.
2. Busca **Propiedades del script**.
3. Añade la propiedad:

| Nombre | Valor |
|---|---|
| `QARANA_ALLOWED_ORIGINS` | `https://TU-USUARIO.github.io` |

Cambia `TU-USUARIO` por el nombre real de tu cuenta de GitHub. Para admitir varios orígenes, sepáralos con comas:

```text
https://tu-usuario.github.io,http://localhost:5500
```

Escribe solo el origen (protocolo + dominio + puerto, si corresponde), sin rutas como `/qarana/`.

**Nota:** cuando se define `QARANA_ALLOWED_ORIGINS`, la lista configurada reemplaza los valores predeterminados. Añade todos los orígenes que realmente necesites.

### 3.7 Conectar el frontend

En `index.html`, busca la constante `API_URL` y comprueba que tenga la URL `/exec` de esta misma implementación:

```javascript
const API_URL = 'https://script.google.com/macros/s/ID_DE_TU_IMPLEMENTACION/exec';
```

El frontend y el backend deben pertenecer a la misma instalación. Si `API_URL` apunta a otra implementación, QARANA consultará otra hoja de cálculo.

### 3.8 Actualizar el backend

Cuando modifiques `Code.gs`:

1. Guarda los cambios.
2. Entra a **Implementar → Administrar implementaciones**.
3. Edita la implementación existente.
4. Selecciona **Nueva versión**.
5. Presiona **Implementar**.

Si mantienes la misma implementación, normalmente conservarás la URL `/exec`. Si creas una implementación nueva, actualiza `API_URL` en `index.html` con la URL nueva.

---

## 4. Estructura de la base de datos

`configurarSistema()` crea o verifica estas 11 hojas:

| Hoja | Para qué se utiliza |
|---|---|
| `USUARIOS` | Cuentas, credenciales cifradas mediante hash, roles y estado. |
| `COMEDORES` | Datos de los comedores populares y su cuenta vinculada. |
| `DONANTES` | Personas u organizaciones donantes y su cuenta vinculada. |
| `ALIMENTOS` | Catálogo de alimentos, categorías y unidades. |
| `DONACIONES` | Publicaciones de alimentos, cantidades disponibles, vencimiento y recojo. |
| `SOLICITUDES` | Solicitudes de alimentos hechas por los comedores y su estado. |
| `MOVIMIENTOS` | Entradas, compras, consumos y ajustes del inventario. |
| `NUTRICION` | Calorías, proteínas, carbohidratos, grasas y fuente de información. |
| `PREPARACIONES` | Platos preparados, porciones, ingredientes e información nutricional. |
| `CALENDARIO` | Planificación de comidas y preparaciones por fecha. |
| `AUDITORIA` | Registro de operaciones importantes realizadas por usuarios. |

### Encabezados principales

El backend define los encabezados en `AS.SHEETS`. Estos son los campos que utiliza:

<details>
<summary><strong>Ver estructura de columnas</strong></summary>

**USUARIOS**

`id`, `nombre`, `correo`, `password_hash`, `salt`, `rol`, `estado`, `fecha_creacion`

**COMEDORES**

`id`, `usuario_id`, `nombre`, `responsable`, `telefono`, `direccion`, `distrito`, `beneficiarios`, `estado`, `fecha_creacion`

**DONANTES**

`id`, `usuario_id`, `nombre`, `tipo`, `contacto`, `telefono`, `direccion`, `estado`, `fecha_creacion`

**ALIMENTOS**

`id`, `nombre`, `categoria`, `unidad`, `descripcion`, `estado`, `fecha_creacion`

**DONACIONES**

`id`, `donante_id`, `alimento_id`, `cantidad_publicada`, `cantidad_disponible`, `unidad`, `fecha_vencimiento`, `fecha_recojo`, `direccion_recojo`, `observaciones`, `estado`, `fecha_creacion`

**SOLICITUDES**

`id`, `donacion_id`, `comedor_id`, `cantidad_solicitada`, `estado`, `observaciones`, `fecha_creacion`, `fecha_actualizacion`

**MOVIMIENTOS**

`id`, `comedor_id`, `alimento_id`, `tipo`, `cantidad`, `unidad`, `referencia_id`, `observaciones`, `fecha`

**NUTRICION**

`alimento_id`, `calorias_100g`, `proteinas_100g`, `carbohidratos_100g`, `grasas_100g`, `fuente`, `actualizado`

**PREPARACIONES**

`id`, `comedor_id`, `nombre_plato`, `fecha`, `porciones`, `ingredientes_json`, `calorias_por_porcion`, `proteinas_por_porcion`, `carbohidratos_por_porcion`, `grasas_por_porcion`, `observaciones`, `fecha_creacion`, `calendario_id`

**CALENDARIO**

`id`, `comedor_id`, `fecha`, `tipo_comida`, `nombre_plato`, `porciones_planificadas`, `ingredientes_json`, `observaciones`, `estado`, `preparacion_id`, `fecha_creacion`, `fecha_actualizacion`

**AUDITORIA**

`id`, `usuario_id`, `usuario_nombre`, `rol`, `accion`, `modulo`, `registro_id`, `detalle`, `fecha`

</details>

**Regla importante:** no renombres las pestañas ni los encabezados. El código identifica las columnas por sus nombres. Si se cambia la estructura manualmente, algunas operaciones pueden dejar de funcionar.

---

## 5. Roles y permisos

El backend contempla tres roles:

| Rol | Alcance general |
|---|---|
| `ADMIN` | Administra usuarios, comedores, donantes, alimentos, donaciones, solicitudes, inventarios, nutrición, preparaciones, calendario y auditoría. |
| `COMEDOR` | Consulta su propio comedor, consulta alimentos y donaciones, solicita alimentos y consulta sus solicitudes e inventario. |
| `DONANTE` | Consulta su perfil, consulta alimentos y publica y administra sus propias donaciones dentro de las reglas del sistema. |

Los permisos se validan en el servidor mediante `usuarioDesdeToken_()`. Ocultar un botón en `index.html` no es suficiente para autorizar una operación: el backend vuelve a comprobar el rol y la propiedad del registro.

### Crear cuentas para comedores y donantes

El administrador crea primero una cuenta con `crearUsuario`. Después debe crear el perfil correspondiente:

- Para una cuenta `COMEDOR`, registrar el perfil con `crearComedor`.
- Para una cuenta `DONANTE`, registrar el perfil con `crearDonante`.

La cuenta y el perfil se relacionan mediante `usuario_id`. Una cuenta COMEDOR no puede vincularse como donante, ni una cuenta DONANTE como comedor.

---

## 6. Cómo funciona la API

### 6.1 Comunicación entre `index.html` y `Code.gs`

El frontend no utiliza `google.script.run`, porque está publicado fuera de Apps Script. Tampoco intenta leer la respuesta directamente con `fetch`.

La comunicación funciona así:

1. `index.html` prepara una solicitud con el nombre de la operación y sus argumentos.
2. Envía la solicitud mediante un formulario POST oculto a la URL `/exec`.
3. Apps Script recibe el parámetro `qaranaRequest` en `doPost(e)`.
4. `doPost` comprueba el origen, el identificador y que la operación esté en la lista permitida.
5. La función solicitada ejecuta las validaciones de sesión, rol y datos.
6. Apps Script genera una respuesta HTML mínima que envía el resultado al frontend mediante `postMessage`.
7. El frontend identifica la respuesta por `requestId` y actualiza la pantalla.

El tiempo máximo de espera configurado en el frontend es de **60 segundos**.

### 6.2 Parámetros de una solicitud

El objeto enviado por el frontend contiene:

| Campo | Significado |
|---|---|
| `type` | Identifica la solicitud como `QARANA_API_REQUEST`. |
| `requestId` | Identificador único para relacionar la respuesta con la solicitud. |
| `clientOrigin` | Origen de la página que realiza la llamada. |
| `method` | Nombre de la operación solicitada. |
| `args` | Lista ordenada de argumentos de esa operación. |

El backend limita el tamaño del contenido recibido y admite hasta 8 argumentos por operación.

### 6.3 Lista de operaciones públicas

Solo se pueden invocar desde el frontend las funciones incluidas en `QARANA_API_METODOS_`. Esta lista blanca evita que el navegador pueda ejecutar cualquier función interna del proyecto.

| Módulo | Operaciones disponibles desde la API |
|---|---|
| Sesión | `iniciarSesion`, `cerrarSesion`, `obtenerSesion` |
| Usuarios | `crearUsuario` |
| Comedores | `crearComedor`, `listarComedores` |
| Donantes | `crearDonante`, `listarDonantes` |
| Alimentos | `crearAlimento`, `listarAlimentos` |
| Donaciones | `crearDonacion`, `listarDonaciones` |
| Solicitudes | `listarSolicitudes`, `resolverSolicitud` |
| Entregas | `registrarEntrega` |
| Panel | `obtenerDashboard` |
| Inventario | `obtenerInventario`, `registrarCompraInventario`, `registrarAjusteInventario` |
| Nutrición | `guardarInformacionNutricional`, `obtenerInformacionNutricional` |
| Preparaciones | `listarPreparaciones`, `registrarPreparacion` |
| Calendario | `guardarEventoCalendario`, `listarEventosCalendario`, `cancelarEventoCalendario`, `obtenerInventarioCalendario` |
| Auditoría | `listarAuditoria` |

**Importante:** el archivo contiene otras funciones internas o de negocio, pero no todas están expuestas a través de la API. Para permitir una operación nueva desde el frontend, el desarrollador debe revisar sus permisos, agregarla explícitamente a `QARANA_API_METODOS_` y actualizar el despliegue.

### 6.4 Cómo invocar una operación desde el frontend

El frontend utiliza una función auxiliar `api(method, ...args)`. Ejemplos conceptuales:

```javascript
// Iniciar sesión
const respuesta = await api(
  'iniciarSesion',
  correo,
  contrasena
);

// Consultar comedores
const comedores = await api(
  'listarComedores',
  token,
  {}
);

// Consultar inventario del comedor autenticado
const inventario = await api(
  'obtenerInventario',
  token,
  {}
);
```

El orden de los argumentos debe coincidir con la firma de la función en `Code.gs`. No cambies el nombre de una operación en el frontend sin actualizar también la lista blanca y comprobar la función del backend.

---

## 7. Flujo operativo del sistema

### 7.1 Preparación inicial

1. El administrador inicia sesión.
2. Crea cuentas de tipo `COMEDOR` y `DONANTE`.
3. Registra el perfil de cada comedor y donante.
4. Registra los alimentos del catálogo.

### 7.2 Publicación de una donación

1. El donante inicia sesión.
2. Selecciona un alimento del catálogo.
3. Indica cantidad, unidad, fecha de vencimiento y datos de recojo.
4. Publica la donación.
5. El comedor puede consultar las donaciones disponibles.

### 7.3 Solicitud y aprobación

1. El comedor selecciona una donación.
2. Registra la cantidad que necesita.
3. La solicitud queda en estado `PENDIENTE`.
4. El administrador revisa y aprueba o rechaza la solicitud.
5. Al aprobarla, el backend reserva la cantidad solicitada y la descuenta de la cantidad disponible de la donación.

El backend evita solicitudes duplicadas pendientes o aprobadas del mismo comedor para la misma donación y verifica que la cantidad solicitada no supere lo disponible.

### 7.4 Registro de entrega

1. El administrador abre una solicitud aprobada.
2. Confirma que el alimento fue entregado.
3. Ejecuta `registrarEntrega`.
4. El backend registra un movimiento `INGRESO_ENTREGA` para el inventario del comedor y cambia la solicitud a `ENTREGADA`.

La aprobación ya reservó la cantidad en la donación. Por eso, registrar la entrega **no vuelve a descontar** esa cantidad de la donación; la incorpora al inventario del comedor.

### 7.5 Compras, ajustes y consumo

- Las compras se registran como `INGRESO_COMPRA`.
- Los ajustes positivos se registran como `AJUSTE_POSITIVO`.
- Los ajustes negativos se registran como `AJUSTE_NEGATIVO`.
- Los consumos de preparaciones se registran como `CONSUMO_PREPARACION`.

El inventario se calcula a partir del historial de `MOVIMIENTOS`; no debe modificarse escribiendo un stock manual en otra columna. Para que los cálculos sean correctos, utiliza unidades consistentes. El backend convierte kg a g para consolidar cantidades de masa; no mezcla unidades diferentes como litros, unidades y gramos.

---

## 8. Sesiones y seguridad

- La sesión se identifica mediante un token temporal generado al iniciar sesión.
- La duración configurada del token es de **6 horas**.
- El backend limita los intentos fallidos de inicio de sesión: después de 5 intentos, bloquea nuevos intentos durante 15 minutos.
- Las contraseñas se almacenan como hash con salt, no como texto plano.
- Las funciones comprueban el rol antes de ejecutar acciones protegidas.
- El backend valida el origen de la página que envía la solicitud.
- Se utiliza `LockService` en operaciones sensibles para reducir conflictos cuando varias personas modifican datos al mismo tiempo.
- Se registran acciones relevantes en `AUDITORIA`.

### Recomendaciones para el administrador

1. No compartas el acceso de propietario de Apps Script con usuarios que solo necesitan utilizar QARANA.
2. Comparte la hoja de cálculo únicamente con personas autorizadas.
3. No publiques contraseñas, tokens, salts ni datos personales en GitHub.
4. No elimines manualmente registros de `MOVIMIENTOS`: el inventario depende de su historial.
5. Realiza copias de seguridad de la hoja de cálculo.
6. Utiliza datos de prueba antes de registrar información real.
7. Revisa los permisos de despliegue de Apps Script antes de publicar la API.

> La validación de origen ayuda a limitar desde qué páginas se reciben solicitudes, pero no reemplaza la autenticación ni los controles de permisos. La URL `/exec` no debe considerarse una contraseña.

---

## 9. Pruebas y comprobaciones

Después de instalar el backend, realiza estas comprobaciones:

- [ ] `configurarSistema()` se ejecutó sin errores.
- [ ] Existen las 11 pestañas de la base de datos.
- [ ] Los encabezados coinciden con `AS.SHEETS`.
- [ ] Se creó el administrador inicial.
- [ ] La aplicación web está implementada y su URL termina en `/exec`.
- [ ] `QARANA_ALLOWED_ORIGINS` contiene el origen correcto, si se configuró.
- [ ] `API_URL` en `index.html` apunta a la implementación correcta.
- [ ] El endpoint `/exec` muestra un mensaje JSON que identifica el servicio como `QARANA API`.
- [ ] El administrador puede iniciar sesión.
- [ ] Se pueden crear y consultar un comedor, un donante y un alimento.
- [ ] Se puede publicar una donación, solicitarla, aprobarla y registrar su entrega.
- [ ] El inventario refleja la entrega registrada.
- [ ] Un usuario COMEDOR no puede ejecutar operaciones reservadas al ADMIN.
- [ ] Un usuario DONANTE no puede modificar registros de otro donante.

Para comprobar el endpoint, abre la URL `/exec` en el navegador. `doGet(e)` devuelve información básica del servicio. Las operaciones de datos se realizan mediante POST desde el frontend.

---

## 10. Errores frecuentes

| Mensaje o problema | Causa posible | Solución |
|---|---|---|
| `No existe la hoja...` | El script está vinculado a otra hoja o no se ejecutó la configuración. | Abre Apps Script desde la hoja correcta y ejecuta `configurarSistema()`. |
| `Origen no autorizado` | El dominio de GitHub Pages no está permitido. | Revisa `QARANA_ALLOWED_ORIGINS` y escribe el origen exacto. |
| `Operación no permitida` | El nombre solicitado no está en la lista blanca. | Comprueba el nombre y agrégalo a `QARANA_API_METODOS_` solo si corresponde. |
| `La API no respondió en 60 segundos` | La implementación está desactualizada, la URL es incorrecta o la ejecución falló. | Revisa `API_URL`, los registros de ejecución y vuelve a implementar una nueva versión. |
| `Correo o contraseña incorrectos` | Credenciales erróneas o usuario inactivo/no existente. | Comprueba el correo y el estado de la cuenta. |
| `Demasiados intentos fallidos` | Se alcanzó el límite de intentos de acceso. | Espera 15 minutos antes de volver a intentar. |
| El frontend muestra datos de otra instalación | `API_URL` apunta a otro despliegue. | Configura la URL correspondiente a la hoja de cálculo correcta. |
| No aparecen cambios recientes | Se guardó el código, pero no se publicó una nueva versión. | Actualiza la implementación desde Administrar implementaciones. |

Para investigar errores, abre Apps Script y consulta **Ejecuciones**. Allí se muestran las ejecuciones recientes y sus errores.

---

## 11. Mantenimiento y ampliación

### Agregar una nueva hoja

1. Define la nueva hoja y sus encabezados en `AS.SHEETS`.
2. Añade el nombre en `nombreHoja_()` si la función lo requiere.
3. Implementa las funciones de lectura, validación y escritura.
4. Comprueba los permisos y el acceso por rol.
5. Si el frontend necesita usarla, agrega la operación a `QARANA_API_METODOS_`.
6. Prueba la operación y vuelve a implementar el backend.

### Agregar una nueva operación a la API

1. Crea o identifica la función de negocio.
2. Valida los argumentos recibidos.
3. Comprueba la sesión y los roles permitidos.
4. Evita confiar en identificadores enviados por el navegador sin comprobar que pertenecen al usuario.
5. Añade la operación a `QARANA_API_METODOS_`.
6. Llama a la operación desde `index.html` mediante `api('nombreOperacion', ...)`.
7. Prueba los casos permitidos y los no permitidos.
8. Publica una nueva versión de Apps Script.

### Copias de seguridad

Desde Google Sheets, crea periódicamente una copia de la base de datos. Conserva las hojas de movimientos y auditoría para mantener la trazabilidad de las operaciones.

---

## 12. Relación con el README principal

- **`README.md`:** presenta QARANA, su propósito, tecnologías, arquitectura general, instalación del proyecto y uso de la plataforma.
- **`README-CODE-GS.md`:** documenta el backend, la estructura de las hojas, las funciones de API, roles, validaciones, seguridad, flujos de datos y mantenimiento de `Code.gs`.

Ambos documentos deben mantenerse juntos en el repositorio para que una persona pueda comprender tanto cómo instalar la solución completa como el funcionamiento de su backend.

---

**QARANA – Cada alimento cuenta.**  
*Conectar corazones y compartir alimentos.*
---

## Código completo de `Code.gs` (copiar y pegar)


Este es el código fuente completo proporcionado para QARANA (Alimentación Solidaria), versión indicada en el propio archivo. Para instalarlo:

1. Abre el proyecto de Google Apps Script vinculado a tu hoja de cálculo.
2. En el editor, abre `Code.gs` (o crea ese archivo si no existe).
3. Selecciona todo el contenido anterior y reemplázalo por el código de abajo.
4. Copia desde la primera línea hasta la última, incluyendo todos los comentarios.
5. Guarda el proyecto antes de ejecutar `configurarSistema()`.

> **Importante:** no pegues este bloque dentro de otro archivo Markdown. Copia únicamente el contenido del bloque `javascript` y pégalo en el editor de Apps Script. No compartas contraseñas, tokens ni credenciales.

```javascript
/**
 * ALIMENTACIÓN SOLIDARIA - Backend v1.1 (unificado)
 * Google Apps Script + Google Sheets
 *
 * Ejecutar primero: configurarSistema()
 * Luego: crearAdministradorInicial("correo@ejemplo.com", "ContraseñaSegura", "Administrador")
 *
 * Frontend independiente: GitHub Pages -> Web App (/exec) -> Google Sheets.
 * No publicar contraseñas ni tokens en hojas visibles.
 */

const AS = {
  VERSION: "1.1.0",
  TOKEN_TTL_SECONDS: 21600, // 6 horas
  MAX_INTENTOS_LOGIN: 5,
  BLOQUEO_LOGIN_SECONDS: 900, // 15 minutos
  SHEETS: {
    USUARIOS: ["id", "nombre", "correo", "password_hash", "salt", "rol", "estado", "fecha_creacion"],
    COMEDORES: ["id", "usuario_id", "nombre", "responsable", "telefono", "direccion", "distrito", "beneficiarios", "estado", "fecha_creacion"],
    DONANTES: ["id", "usuario_id", "nombre", "tipo", "contacto", "telefono", "direccion", "estado", "fecha_creacion"],
    ALIMENTOS: ["id", "nombre", "categoria", "unidad", "descripcion", "estado", "fecha_creacion"],
    DONACIONES: ["id", "donante_id", "alimento_id", "cantidad_publicada", "cantidad_disponible", "unidad", "fecha_vencimiento", "fecha_recojo", "direccion_recojo", "observaciones", "estado", "fecha_creacion"],
    SOLICITUDES: ["id", "donacion_id", "comedor_id", "cantidad_solicitada", "estado", "observaciones", "fecha_creacion", "fecha_actualizacion"],
    MOVIMIENTOS: ["id", "comedor_id", "alimento_id", "tipo", "cantidad", "unidad", "referencia_id", "observaciones", "fecha"],
    NUTRICION: ["alimento_id", "calorias_100g", "proteinas_100g", "carbohidratos_100g", "grasas_100g", "fuente", "actualizado"],
    PREPARACIONES: ["id", "comedor_id", "nombre_plato", "fecha", "porciones", "ingredientes_json", "calorias_por_porcion", "proteinas_por_porcion", "carbohidratos_por_porcion", "grasas_por_porcion", "observaciones", "fecha_creacion", "calendario_id"],
    CALENDARIO: ["id", "comedor_id", "fecha", "tipo_comida", "nombre_plato", "porciones_planificadas", "ingredientes_json", "observaciones", "estado", "preparacion_id", "fecha_creacion", "fecha_actualizacion"],
    AUDITORIA: ["id", "usuario_id", "usuario_nombre", "rol", "accion", "modulo", "registro_id", "detalle", "fecha"]
  }
};

/** Crea las hojas y encabezados sin borrar datos existentes. */
function configurarSistema() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  if (!ss) throw new Error("Abre este proyecto desde la hoja de cálculo o configura un ID de Spreadsheet.");

  Object.keys(AS.SHEETS).forEach(function(key) {
    const name = nombreHoja_(key);
    let sh = ss.getSheetByName(name);
    if (!sh) sh = ss.insertSheet(name);

    const headers = AS.SHEETS[key];
    if (sh.getLastRow() === 0) {
      sh.getRange(1, 1, 1, headers.length).setValues([headers]);
      sh.setFrozenRows(1);
    } else {
      const current = sh.getRange(1, 1, 1, Math.max(sh.getLastColumn(), headers.length))
        .getValues()[0].filter(String);
      const missing = headers.filter(h => current.indexOf(h) === -1);
      if (missing.length) {
        sh.getRange(1, current.length + 1, 1, missing.length).setValues([missing]);
      }
    }
  });

  return { ok: true, message: "Estructura de Alimentación Solidaria creada/verificada.", version: AS.VERSION };
}

/** Ejecutar manualmente una sola vez para crear el primer administrador. */
function crearAdministradorInicial(correo, contrasena, nombre) {
  configurarSistema();
  correo = normalizarCorreo_(correo);
  if (!correo || !contrasena || String(contrasena).length < 10) {
    throw new Error("Indica un correo y una contraseña de al menos 10 caracteres.");
  }

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const usuarios = leerHoja_("USUARIOS");
    if (usuarios.some(u => normalizarCorreo_(u.correo) === correo)) {
      throw new Error("Ya existe una cuenta con ese correo.");
    }

    const salt = Utilities.getUuid();
    const hash = hashPassword_(contrasena, salt);
    insertar_("USUARIOS", {
      id: nuevoId_("USR"), nombre: nombre || "Administrador", correo: correo,
      password_hash: hash, salt: salt, rol: "ADMIN", estado: "ACTIVO",
      fecha_creacion: new Date()
    });
  } finally {
    lock.releaseLock();
  }
  return { ok: true, message: "Administrador creado. Inicia sesión desde la aplicación." };
}

/** Inicio de sesión. Devuelve un token temporal que el frontend debe conservar. */
function iniciarSesion(correo, contrasena) {
  const email = normalizarCorreo_(correo);
  if (!email || !contrasena) throw new Error("Indica tu correo y contraseña.");

  const cache = CacheService.getScriptCache();
  const claveFallos = "AS_FAIL_" + hashPassword_(email, "intentos");
  const fallos = Number(cache.get(claveFallos) || 0);
  if (fallos >= AS.MAX_INTENTOS_LOGIN) {
    throw new Error("Demasiados intentos fallidos. Espera unos minutos e inténtalo nuevamente.");
  }

  const user = leerHoja_("USUARIOS").find(u =>
    normalizarCorreo_(u.correo) === email && String(u.estado).toUpperCase() === "ACTIVO"
  );
  if (!user || hashPassword_(contrasena, user.salt) !== user.password_hash) {
    cache.put(claveFallos, String(fallos + 1), AS.BLOQUEO_LOGIN_SECONDS);
    throw new Error("Correo o contraseña incorrectos.");
  }
  cache.remove(claveFallos);

  const token = Utilities.getUuid() + Utilities.getUuid();
  const safeUser = {
    id: user.id, nombre: user.nombre, correo: user.correo, rol: String(user.rol).toUpperCase()
  };
  cache.put("AS_TOKEN_" + token, JSON.stringify(safeUser), AS.TOKEN_TTL_SECONDS);
  return { ok: true, token: token, usuario: safeUser, expiraEnSegundos: AS.TOKEN_TTL_SECONDS };
}

function cerrarSesion(token) {
  if (token) CacheService.getScriptCache().remove("AS_TOKEN_" + token);
  return { ok: true };
}

function obtenerSesion(token) {
  return { ok: true, usuario: usuarioDesdeToken_(token) };
}


/** Crear una cuenta de COMEDOR o DONANTE. Solo ADMIN puede hacerlo. */
function crearUsuario(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["nombre", "correo", "contrasena", "rol"]);
  const rol = String(datos.rol).toUpperCase();
  if (["COMEDOR", "DONANTE"].indexOf(rol) === -1) {
    throw new Error("El rol debe ser COMEDOR o DONANTE. El ADMIN inicial se crea con crearAdministradorInicial().");
  }
  const correo = normalizarCorreo_(datos.correo);
  if (String(datos.contrasena).length < 10) throw new Error("La contraseña debe tener al menos 10 caracteres.");

  const salt = Utilities.getUuid();
  const user = {
    id: nuevoId_("USR"), nombre: limpiar_(datos.nombre), correo: correo,
    password_hash: hashPassword_(datos.contrasena, salt), salt: salt,
    rol: rol, estado: "ACTIVO", fecha_creacion: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    if (leerHoja_("USUARIOS").some(u => normalizarCorreo_(u.correo) === correo)) {
      throw new Error("Ya existe una cuenta con ese correo.");
    }
    insertar_("USUARIOS", user);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: user.id, nombre: user.nombre, correo: user.correo, rol: user.rol };
}


/* ============================================================
   GESTIÓN DE COMEDORES
   ============================================================ */

/**
 * Registra un comedor y lo vincula con una cuenta de usuario COMEDOR.
 * datos: { usuario_id, nombre, responsable, telefono, direccion,
 *          distrito, beneficiarios }
 */
function crearComedor(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["usuario_id", "nombre", "responsable", "direccion"]);

  const usuario = exigirRegistro_("USUARIOS", datos.usuario_id);
  if (String(usuario.rol).toUpperCase() !== "COMEDOR") {
    throw new Error("La cuenta vinculada debe tener el rol COMEDOR.");
  }
  if (String(usuario.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("La cuenta del comedor no está activa.");
  }

  const comedor = {
    id: nuevoId_("COM"),
    usuario_id: limpiar_(datos.usuario_id),
    nombre: limpiar_(datos.nombre),
    responsable: limpiar_(datos.responsable),
    telefono: limpiar_(datos.telefono),
    direccion: limpiar_(datos.direccion),
    distrito: limpiar_(datos.distrito),
    beneficiarios: datos.beneficiarios === "" || datos.beneficiarios === undefined || datos.beneficiarios === null
      ? 0 : numeroNoNegativo_(datos.beneficiarios),
    estado: "ACTIVO",
    fecha_creacion: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const existente = leerHoja_("COMEDORES").some(c =>
      String(c.usuario_id) === String(datos.usuario_id)
    );
    if (existente) throw new Error("Esta cuenta ya está vinculada a un comedor.");
    insertar_("COMEDORES", comedor);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: comedor.id, data: comedor, message: "Comedor registrado correctamente." };
}

/** El administrador puede actualizar los datos de un comedor. */
function actualizarComedor(token, comedorId, cambios) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  if (!cambios || typeof cambios !== "object") throw new Error("No se recibieron cambios.");

  const permitidos = ["nombre", "responsable", "telefono", "direccion", "distrito", "beneficiarios"];
  const limpios = {};
  Object.keys(cambios).forEach(k => {
    if (permitidos.indexOf(k) !== -1) {
      limpios[k] = k === "beneficiarios" ? numeroNoNegativo_(cambios[k]) : limpiar_(cambios[k]);
    }
  });
  if (!Object.keys(limpios).length) throw new Error("No hay campos válidos para actualizar.");

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    actualizar_("COMEDORES", comedorId, limpios);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: comedorId, message: "Comedor actualizado correctamente." };
}

/** Lista de comedores. ADMIN ve todos; COMEDOR solo ve su propio registro. */
function listarComedores(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("COMEDORES");

  if (user.rol === "COMEDOR") {
    lista = lista.filter(c => String(c.usuario_id) === String(user.id));
  } else {
    if (filtros.estado) {
      lista = lista.filter(c => String(c.estado).toUpperCase() === String(filtros.estado).toUpperCase());
    }
    if (filtros.distrito) {
      lista = lista.filter(c => String(c.distrito).toLowerCase().indexOf(String(filtros.distrito).toLowerCase()) !== -1);
    }
  }
  return { ok: true, data: lista };
}

/** Cambia el estado de un comedor a ACTIVO o INACTIVO. */
function cambiarEstadoComedor(token, comedorId, estado) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  estado = String(estado || "").toUpperCase();
  if (["ACTIVO", "INACTIVO"].indexOf(estado) === -1) {
    throw new Error("El estado debe ser ACTIVO o INACTIVO.");
  }
  actualizar_("COMEDORES", comedorId, { estado: estado });
  return { ok: true, id: comedorId, message: "Estado del comedor actualizado." };
}


/* ============================================================
   GESTIÓN DE DONANTES
   ============================================================ */

/**
 * Registra un donante vinculado a una cuenta DONANTE.
 * datos: { usuario_id, nombre, tipo, contacto, telefono, direccion }
 * tipo sugerido: PERSONA, MERCADO, RESTAURANTE, EMPRESA u OTRO.
 */
function crearDonante(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["usuario_id", "nombre", "tipo"]);

  const usuario = exigirRegistro_("USUARIOS", datos.usuario_id);
  if (String(usuario.rol).toUpperCase() !== "DONANTE") {
    throw new Error("La cuenta vinculada debe tener el rol DONANTE.");
  }
  if (String(usuario.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("La cuenta del donante no está activa.");
  }

  const donante = {
    id: nuevoId_("DON"),
    usuario_id: limpiar_(datos.usuario_id),
    nombre: limpiar_(datos.nombre),
    tipo: limpiar_(datos.tipo).toUpperCase(),
    contacto: limpiar_(datos.contacto),
    telefono: limpiar_(datos.telefono),
    direccion: limpiar_(datos.direccion),
    estado: "ACTIVO",
    fecha_creacion: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    if (leerHoja_("DONANTES").some(d => String(d.usuario_id) === String(datos.usuario_id))) {
      throw new Error("Esta cuenta ya está vinculada a un donante.");
    }
    insertar_("DONANTES", donante);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: donante.id, data: donante, message: "Donante registrado correctamente." };
}

function actualizarDonante(token, donanteId, cambios) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  if (!cambios || typeof cambios !== "object") throw new Error("No se recibieron cambios.");

  const permitidos = ["nombre", "tipo", "contacto", "telefono", "direccion"];
  const limpios = {};
  Object.keys(cambios).forEach(k => {
    if (permitidos.indexOf(k) !== -1) {
      limpios[k] = k === "tipo" ? limpiar_(cambios[k]).toUpperCase() : limpiar_(cambios[k]);
    }
  });
  if (!Object.keys(limpios).length) throw new Error("No hay campos válidos para actualizar.");

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    actualizar_("DONANTES", donanteId, limpios);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: donanteId, message: "Donante actualizado correctamente." };
}

/** ADMIN ve todos; DONANTE solo ve su propio registro. */
function listarDonantes(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "DONANTE"]);
  filtros = filtros || {};
  let lista = leerHoja_("DONANTES");

  if (user.rol === "DONANTE") {
    lista = lista.filter(d => String(d.usuario_id) === String(user.id));
  } else {
    if (filtros.estado) {
      lista = lista.filter(d => String(d.estado).toUpperCase() === String(filtros.estado).toUpperCase());
    }
    if (filtros.tipo) {
      lista = lista.filter(d => String(d.tipo).toUpperCase() === String(filtros.tipo).toUpperCase());
    }
  }
  return { ok: true, data: lista };
}

function cambiarEstadoDonante(token, donanteId, estado) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  estado = String(estado || "").toUpperCase();
  if (["ACTIVO", "INACTIVO"].indexOf(estado) === -1) {
    throw new Error("El estado debe ser ACTIVO o INACTIVO.");
  }
  actualizar_("DONANTES", donanteId, { estado: estado });
  return { ok: true, id: donanteId, message: "Estado del donante actualizado." };
}


/* ============================================================
   CATÁLOGO DE ALIMENTOS
   ============================================================ */

/** ADMIN crea alimentos del catálogo. */
function crearAlimento(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["nombre", "categoria", "unidad"]);

  const nombre = limpiar_(datos.nombre);
  const categoria = limpiar_(datos.categoria);
  const unidad = limpiar_(datos.unidad).toLowerCase();

  const alimento = {
    id: nuevoId_("ALI"),
    nombre: nombre,
    categoria: categoria,
    unidad: unidad,
    descripcion: limpiar_(datos.descripcion),
    estado: "ACTIVO",
    fecha_creacion: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const duplicado = leerHoja_("ALIMENTOS").some(a =>
      String(a.nombre).trim().toLowerCase() === nombre.toLowerCase() &&
      String(a.estado).toUpperCase() !== "INACTIVO"
    );
    if (duplicado) throw new Error("Ya existe un alimento activo con ese nombre.");
    insertar_("ALIMENTOS", alimento);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: alimento.id, data: alimento, message: "Alimento creado correctamente." };
}

function actualizarAlimento(token, alimentoId, cambios) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  if (!cambios || typeof cambios !== "object") throw new Error("No se recibieron cambios.");

  const permitidos = ["nombre", "categoria", "unidad", "descripcion"];
  const limpios = {};
  Object.keys(cambios).forEach(k => {
    if (permitidos.indexOf(k) !== -1) {
      limpios[k] = k === "unidad" ? limpiar_(cambios[k]).toLowerCase() : limpiar_(cambios[k]);
    }
  });
  if (!Object.keys(limpios).length) throw new Error("No hay campos válidos para actualizar.");

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    actualizar_("ALIMENTOS", alimentoId, limpios);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: alimentoId, message: "Alimento actualizado correctamente." };
}

/** ADMIN, DONANTE y COMEDOR pueden consultar el catálogo activo. */
function listarAlimentos(token, filtros) {
  usuarioDesdeToken_(token, ["ADMIN", "DONANTE", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("ALIMENTOS");

  lista = lista.filter(a => String(a.estado).toUpperCase() === "ACTIVO");
  if (filtros.categoria) {
    lista = lista.filter(a => String(a.categoria).toLowerCase() === String(filtros.categoria).toLowerCase());
  }
  if (filtros.buscar) {
    const q = String(filtros.buscar).toLowerCase();
    lista = lista.filter(a => String(a.nombre).toLowerCase().indexOf(q) !== -1 ||
      String(a.descripcion).toLowerCase().indexOf(q) !== -1);
  }
  return { ok: true, data: lista };
}

function cambiarEstadoAlimento(token, alimentoId, estado) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  estado = String(estado || "").toUpperCase();
  if (["ACTIVO", "INACTIVO"].indexOf(estado) === -1) {
    throw new Error("El estado debe ser ACTIVO o INACTIVO.");
  }
  actualizar_("ALIMENTOS", alimentoId, { estado: estado });
  return { ok: true, id: alimentoId, message: "Estado del alimento actualizado." };
}


/* ============================================================
   PUBLICACIÓN Y GESTIÓN DE DONACIONES
   ============================================================ */

/**
 * El donante publica un alimento disponible.
 * datos: { alimento_id, cantidad, unidad, fecha_vencimiento,
 *          fecha_recojo, direccion_recojo, observaciones }
 */
function crearDonacion(token, datos) {
  const user = usuarioDesdeToken_(token, ["DONANTE", "ADMIN"]);
  validarRequeridos_(datos, ["alimento_id", "cantidad", "unidad", "fecha_vencimiento"]);

  const alimento = exigirRegistro_("ALIMENTOS", datos.alimento_id);
  if (String(alimento.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("El alimento seleccionado no está activo.");
  }

  let donante;
  if (user.rol === "DONANTE") {
    donante = leerHoja_("DONANTES").find(d =>
      String(d.usuario_id) === String(user.id) && String(d.estado).toUpperCase() === "ACTIVO"
    );
    if (!donante) throw new Error("Tu usuario no tiene un perfil de donante activo.");
  } else {
    validarRequeridos_(datos, ["donante_id"]);
    donante = exigirRegistro_("DONANTES", datos.donante_id);
  }
  if (String(donante.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("El perfil del donante está inactivo.");
  }

  const cantidad = numeroPositivo_(datos.cantidad);
  const vencimiento = fechaValida_(datos.fecha_vencimiento);
  const hoy = new Date();
  hoy.setHours(0, 0, 0, 0);
  if (vencimiento < hoy) throw new Error("La fecha de vencimiento no puede estar en el pasado.");

  const fechaRecojo = datos.fecha_recojo ? fechaValida_(datos.fecha_recojo) : "";
  const donacion = {
    id: nuevoId_("DNC"),
    donante_id: donante.id,
    alimento_id: alimento.id,
    cantidad_publicada: cantidad,
    cantidad_disponible: cantidad,
    unidad: limpiar_(datos.unidad).toLowerCase(),
    fecha_vencimiento: vencimiento,
    fecha_recojo: fechaRecojo,
    direccion_recojo: limpiar_(datos.direccion_recojo),
    observaciones: limpiar_(datos.observaciones),
    estado: "DISPONIBLE",
    fecha_creacion: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    insertar_("DONACIONES", donacion);
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: donacion.id, data: donacion, message: "Donación publicada correctamente." };
}

/** Consulta donaciones. ADMIN ve todas; DONANTE ve las propias; COMEDOR ve disponibles y vigentes. */
function listarDonaciones(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "DONANTE", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("DONACIONES");

  if (user.rol === "DONANTE") {
    const perfil = leerHoja_("DONANTES").find(d => String(d.usuario_id) === String(user.id));
    if (!perfil) return { ok: true, data: [] };
    lista = lista.filter(d => String(d.donante_id) === String(perfil.id));
  } else if (user.rol === "COMEDOR") {
    const hoy = new Date();
    hoy.setHours(0, 0, 0, 0);
    lista = lista.filter(d => {
      if (String(d.estado).toUpperCase() !== "DISPONIBLE" || Number(d.cantidad_disponible) <= 0) return false;
      const venc = new Date(d.fecha_vencimiento);
      return isNaN(venc.getTime()) || venc >= hoy;
    });
  }

  if (filtros.estado && user.rol === "ADMIN") {
    lista = lista.filter(d => String(d.estado).toUpperCase() === String(filtros.estado).toUpperCase());
  }
  if (filtros.alimento_id) {
    lista = lista.filter(d => String(d.alimento_id) === String(filtros.alimento_id));
  }
  if (filtros.donante_id && user.rol === "ADMIN") {
    lista = lista.filter(d => String(d.donante_id) === String(filtros.donante_id));
  }
  return { ok: true, data: lista };
}

/** ADMIN puede cambiar estado; DONANTE puede cancelar su propia publicación si aún no fue aprobada/entregada. */
function cambiarEstadoDonacion(token, donacionId, estado) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "DONANTE"]);
  estado = String(estado || "").toUpperCase();
  const estados = ["DISPONIBLE", "AGOTADA", "VENCIDA", "CANCELADA"];
  if (estados.indexOf(estado) === -1) throw new Error("Estado de donación no válido.");

  const donacion = exigirRegistro_("DONACIONES", donacionId);
  if (user.rol === "DONANTE") {
    const perfil = leerHoja_("DONANTES").find(d => String(d.usuario_id) === String(user.id));
    if (!perfil || String(donacion.donante_id) !== String(perfil.id)) {
      throw new Error("No tienes permiso para modificar esta donación.");
    }
    if (estado !== "CANCELADA") throw new Error("El donante solo puede cancelar su publicación.");
    const solicitudes = leerHoja_("SOLICITUDES").filter(s =>
      String(s.donacion_id) === String(donacionId) &&
      ["APROBADA", "ENTREGADA"].indexOf(String(s.estado).toUpperCase()) !== -1
    );
    if (solicitudes.length) throw new Error("No se puede cancelar una donación con solicitudes aprobadas o entregadas.");
  }

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    actualizar_("DONACIONES", donacionId, { estado: estado });
  } finally {
    lock.releaseLock();
  }
  return { ok: true, id: donacionId, message: "Estado de la donación actualizado." };
}


/* ============================================================
   SOLICITUDES DE ALIMENTOS
   ============================================================ */

/**
 * El comedor solicita una cantidad de una donación disponible.
 * datos: { donacion_id, cantidad_solicitada, observaciones }
 */
function crearSolicitud(token, datos) {
  const user = usuarioDesdeToken_(token, ["COMEDOR", "ADMIN"]);
  validarRequeridos_(datos, ["donacion_id", "cantidad_solicitada"]);

  const cantidad = numeroPositivo_(datos.cantidad_solicitada);

  let comedor;
  if (user.rol === "COMEDOR") {
    comedor = leerHoja_("COMEDORES").find(c =>
      String(c.usuario_id) === String(user.id) && String(c.estado).toUpperCase() === "ACTIVO"
    );
    if (!comedor) throw new Error("Tu usuario no tiene un perfil de comedor activo.");
  } else {
    validarRequeridos_(datos, ["comedor_id"]);
    comedor = exigirRegistro_("COMEDORES", datos.comedor_id);
  }
  if (String(comedor.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("El comedor está inactivo.");
  }

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    // Leer la donación dentro del bloqueo para evitar solicitudes que excedan el saldo.
    const actual = exigirRegistro_("DONACIONES", datos.donacion_id);
    if (String(actual.estado).toUpperCase() !== "DISPONIBLE") {
      throw new Error("Esta donación no está disponible.");
    }
    const hoy = new Date();
    hoy.setHours(0, 0, 0, 0);
    const venc = new Date(actual.fecha_vencimiento);
    if (!isNaN(venc.getTime()) && venc < hoy) {
      throw new Error("Esta donación ya venció.");
    }
    if (cantidad > Number(actual.cantidad_disponible)) {
      throw new Error("La cantidad solicitada supera la cantidad disponible.");
    }

    const duplicada = leerHoja_("SOLICITUDES").some(s =>
      String(s.donacion_id) === String(datos.donacion_id) &&
      String(s.comedor_id) === String(comedor.id) &&
      ["PENDIENTE", "APROBADA"].indexOf(String(s.estado).toUpperCase()) !== -1
    );
    if (duplicada) throw new Error("Este comedor ya tiene una solicitud pendiente o aprobada para esta donación.");

    const solicitud = {
      id: nuevoId_("SOL"),
      donacion_id: String(datos.donacion_id),
      comedor_id: comedor.id,
      cantidad_solicitada: cantidad,
      estado: "PENDIENTE",
      observaciones: limpiar_(datos.observaciones),
      fecha_creacion: new Date(),
      fecha_actualizacion: new Date()
    };
    insertar_("SOLICITUDES", solicitud);
    return { ok: true, id: solicitud.id, data: solicitud, message: "Solicitud registrada y pendiente de revisión." };
  } finally {
    lock.releaseLock();
  }
}

/** ADMIN revisa solicitudes. COMEDOR solo consulta las propias. */
function listarSolicitudes(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("SOLICITUDES");

  if (user.rol === "COMEDOR") {
    const comedor = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id));
    if (!comedor) return { ok: true, data: [] };
    lista = lista.filter(s => String(s.comedor_id) === String(comedor.id));
  } else {
    if (filtros.estado) {
      lista = lista.filter(s => String(s.estado).toUpperCase() === String(filtros.estado).toUpperCase());
    }
    if (filtros.comedor_id) {
      lista = lista.filter(s => String(s.comedor_id) === String(filtros.comedor_id));
    }
    if (filtros.donacion_id) {
      lista = lista.filter(s => String(s.donacion_id) === String(filtros.donacion_id));
    }
  }
  return { ok: true, data: lista };
}

/**
 * ADMIN aprueba o rechaza una solicitud.
 * estados permitidos: APROBADA, RECHAZADA.
 * La aprobación reserva la cantidad restándola de cantidad_disponible.
 */
function resolverSolicitud(token, solicitudId, decision, observaciones) {
  const admin = usuarioDesdeToken_(token, ["ADMIN"]);
  decision = String(decision || "").toUpperCase();
  if (["APROBADA", "RECHAZADA"].indexOf(decision) === -1) {
    throw new Error("La decisión debe ser APROBADA o RECHAZADA.");
  }

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const solicitud = exigirRegistro_("SOLICITUDES", solicitudId);
    if (String(solicitud.estado).toUpperCase() !== "PENDIENTE") {
      throw new Error("Solo se pueden resolver solicitudes pendientes.");
    }
    const donacion = exigirRegistro_("DONACIONES", solicitud.donacion_id);

    let donacionActualizada = false;
    if (decision === "APROBADA") {
      const cantidad = numeroPositivo_(solicitud.cantidad_solicitada);
      const disponible = numeroNoNegativo_(donacion.cantidad_disponible);
      if (String(donacion.estado).toUpperCase() !== "DISPONIBLE" || cantidad > disponible) {
        throw new Error("La donación ya no tiene cantidad suficiente disponible.");
      }
      const restante = redondear_(disponible - cantidad);
      actualizar_("DONACIONES", donacion.id, {
        cantidad_disponible: restante,
        estado: restante <= 0 ? "AGOTADA" : "DISPONIBLE"
      });
      donacionActualizada = true;
    }

    try {
      actualizar_("SOLICITUDES", solicitudId, {
        estado: decision,
        observaciones: observaciones === undefined ? solicitud.observaciones : limpiar_(observaciones),
        fecha_actualizacion: new Date()
      });
    } catch (errorActualizacion) {
      if (donacionActualizada) actualizar_("DONACIONES", donacion.id, {
        cantidad_disponible: donacion.cantidad_disponible, estado: donacion.estado
      });
      throw errorActualizacion;
    }
    registrarAuditoria_(admin, "RESOLVER_SOLICITUD_" + decision, "SOLICITUDES", solicitudId, "Solicitud " + decision.toLowerCase());
    return { ok: true, id: solicitudId, message: "Solicitud " + decision.toLowerCase() + " correctamente." };
  } finally {
    lock.releaseLock();
  }
}

/** Un comedor puede cancelar su propia solicitud si sigue pendiente. */
function cancelarSolicitud(token, solicitudId) {
  const user = usuarioDesdeToken_(token, ["COMEDOR"]);
  const comedor = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id));
  if (!comedor) throw new Error("No se encontró el perfil del comedor.");

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const solicitud = exigirRegistro_("SOLICITUDES", solicitudId);
    if (String(solicitud.comedor_id) !== String(comedor.id)) {
      throw new Error("No tienes permiso para cancelar esta solicitud.");
    }
    if (String(solicitud.estado).toUpperCase() !== "PENDIENTE") {
      throw new Error("Solo se pueden cancelar solicitudes pendientes.");
    }
    actualizar_("SOLICITUDES", solicitudId, {
      estado: "CANCELADA",
      fecha_actualizacion: new Date()
    });
    return { ok: true, id: solicitudId, message: "Solicitud cancelada." };
  } finally {
    lock.releaseLock();
  }
}


/* ============================================================
   REGISTRO DE ENTREGAS
   ============================================================ */

/**
 * Registra la entrega de una solicitud APROBADA al comedor.
 * La aprobación de la solicitud ya reservó/restó la cantidad de la donación.
 * Aquí se ingresa esa cantidad al inventario del comedor, sin volver a
 * descontarla de DONACIONES.
 */
function registrarEntrega(token, solicitudId, observaciones) {
  usuarioDesdeToken_(token, ["ADMIN"]);

  const lock = LockService.getScriptLock();
  lock.waitLock(15000);
  try {
    const solicitud = exigirRegistro_("SOLICITUDES", solicitudId);
    if (String(solicitud.estado).toUpperCase() !== "APROBADA") {
      throw new Error("Solo se pueden entregar solicitudes aprobadas.");
    }

    const yaEntregada = leerHoja_("MOVIMIENTOS").some(m =>
      String(m.referencia_id) === String(solicitudId) &&
      String(m.tipo).toUpperCase() === "INGRESO_ENTREGA"
    );
    if (yaEntregada) throw new Error("Esta solicitud ya tiene una entrega registrada.");

    const donacion = exigirRegistro_("DONACIONES", solicitud.donacion_id);
    const alimento = exigirRegistro_("ALIMENTOS", donacion.alimento_id);
    const comedor = exigirRegistro_("COMEDORES", solicitud.comedor_id);
    const cantidad = numeroPositivo_(solicitud.cantidad_solicitada);

    const movimiento = {
      id: nuevoId_("MOV"),
      comedor_id: comedor.id,
      alimento_id: alimento.id,
      tipo: "INGRESO_ENTREGA",
      cantidad: cantidad,
      unidad: limpiar_(donacion.unidad || alimento.unidad).toLowerCase(),
      referencia_id: String(solicitudId),
      observaciones: limpiar_(observaciones || "Entrega de solicitud aprobada"),
      fecha: new Date()
    };

    insertar_("MOVIMIENTOS", movimiento);
    try {
      actualizar_("SOLICITUDES", solicitudId, {
        estado: "ENTREGADA",
        observaciones: observaciones === undefined ? solicitud.observaciones : limpiar_(observaciones),
        fecha_actualizacion: new Date()
      });
    } catch (errorActualizacion) {
      eliminarRegistroPorId_("MOVIMIENTOS", movimiento.id);
      throw new Error("No se pudo confirmar la entrega; se revirtió el movimiento de inventario. " + errorActualizacion.message);
    }
    const admin = usuarioDesdeToken_(token, ["ADMIN"]);
    registrarAuditoria_(admin, "REGISTRAR_ENTREGA", "INVENTARIO", movimiento.id, "Entrega de solicitud " + solicitudId);

    return {
      ok: true,
      id: movimiento.id,
      data: movimiento,
      message: "Entrega registrada e inventario del comedor actualizado."
    };
  } finally {
    lock.releaseLock();
  }
}


/* ============================================================
   INVENTARIO Y MOVIMIENTOS
   ============================================================ */

/**
 * Calcula inventario a partir del historial MOVIMIENTOS.
 * Los ingresos suman; consumos y salidas restan.
 * No mezcla unidades distintas: convierte g/kg a gramos para consolidar.
 */
function obtenerInventario(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};

  let comedorId = filtros.comedor_id || "";
  if (user.rol === "COMEDOR") {
    const perfil = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id));
    if (!perfil) throw new Error("No se encontró el perfil del comedor.");
    comedorId = perfil.id;
  } else if (!comedorId) {
    throw new Error("El administrador debe indicar comedor_id para consultar un inventario.");
  }

  const movimientos = leerHoja_("MOVIMIENTOS").filter(m =>
    String(m.comedor_id) === String(comedorId)
  );
  const alimentos = leerHoja_("ALIMENTOS");
  const acumulado = {};

  movimientos.forEach(m => {
    const alimento = alimentos.find(a => String(a.id) === String(m.alimento_id));
    const nombre = alimento ? alimento.nombre : "Alimento eliminado o no identificado";
    const unidad = String(m.unidad || (alimento ? alimento.unidad : "")).toLowerCase();
    const conversion = convertirAUnidadBase_(Number(m.cantidad), unidad);
    const key = String(m.alimento_id) + "|" + conversion.unidadBase;

    if (!acumulado[key]) {
      acumulado[key] = {
        alimento_id: m.alimento_id,
        alimento: nombre,
        cantidad: 0,
        unidad: conversion.unidadBase,
        comedor_id: comedorId
      };
    }

    const tipo = String(m.tipo).toUpperCase();
    if (["INGRESO_ENTREGA", "INGRESO_COMPRA", "ENTRADA_COMPRA", "INGRESO", "AJUSTE_POSITIVO"].indexOf(tipo) !== -1) {
      acumulado[key].cantidad += conversion.cantidadBase;
    } else if (["CONSUMO_PREPARACION", "SALIDA", "AJUSTE_NEGATIVO"].indexOf(tipo) !== -1) {
      acumulado[key].cantidad -= conversion.cantidadBase;
    }
  });

  let resultado = Object.keys(acumulado).map(k => {
    const item = acumulado[k];
    item.cantidad = redondear_(item.cantidad);
    return item;
  }).filter(x => Math.abs(x.cantidad) > 0.000001);

  if (filtros.alimento_id) {
    resultado = resultado.filter(x => String(x.alimento_id) === String(filtros.alimento_id));
  }
  return { ok: true, data: resultado };
}

/** Devuelve el historial de movimientos. COMEDOR solo ve los propios. */
function listarMovimientos(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("MOVIMIENTOS");

  if (user.rol === "COMEDOR") {
    const perfil = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id));
    if (!perfil) return { ok: true, data: [] };
    lista = lista.filter(m => String(m.comedor_id) === String(perfil.id));
  } else {
    if (filtros.comedor_id) lista = lista.filter(m => String(m.comedor_id) === String(filtros.comedor_id));
  }

  if (filtros.alimento_id) lista = lista.filter(m => String(m.alimento_id) === String(filtros.alimento_id));
  if (filtros.tipo) lista = lista.filter(m => String(m.tipo).toUpperCase() === String(filtros.tipo).toUpperCase());

  lista.sort((a, b) => new Date(b.fecha).getTime() - new Date(a.fecha).getTime());
  return { ok: true, data: lista };
}

/**
 * Permite al ADMIN registrar un ajuste positivo o negativo justificado.
 * datos: { comedor_id, alimento_id, tipo, cantidad, unidad, observaciones }
 */
function registrarAjusteInventario(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["comedor_id", "alimento_id", "tipo", "cantidad", "unidad", "observaciones"]);

  const comedor = exigirRegistro_("COMEDORES", datos.comedor_id);
  const alimento = exigirRegistro_("ALIMENTOS", datos.alimento_id);
  if (String(comedor.estado).toUpperCase() !== "ACTIVO") throw new Error("El comedor está inactivo.");
  if (String(alimento.estado).toUpperCase() !== "ACTIVO") throw new Error("El alimento está inactivo.");

  const tipo = String(datos.tipo).toUpperCase();
  if (["AJUSTE_POSITIVO", "AJUSTE_NEGATIVO"].indexOf(tipo) === -1) {
    throw new Error("El tipo debe ser AJUSTE_POSITIVO o AJUSTE_NEGATIVO.");
  }
  const cantidad = numeroPositivo_(datos.cantidad);
  const unidad = limpiar_(datos.unidad).toLowerCase();

  const lock = LockService.getScriptLock();
  lock.waitLock(15000);
  try {
    if (tipo === "AJUSTE_NEGATIVO") {
      const stock = calcularStockAlimento_(datos.comedor_id, datos.alimento_id);
      const solicitadaBase = convertirAUnidadBase_(cantidad, unidad);
      if (stock.unidadBase !== solicitadaBase.unidadBase ||
          solicitadaBase.cantidadBase > stock.cantidadBase + 0.000001) {
        throw new Error("El ajuste dejaría el inventario negativo o usa una unidad incompatible.");
      }
    }

    const movimiento = {
      id: nuevoId_("MOV"),
      comedor_id: comedor.id,
      alimento_id: alimento.id,
      tipo: tipo,
      cantidad: cantidad,
      unidad: unidad,
      referencia_id: "",
      observaciones: limpiar_(datos.observaciones),
      fecha: new Date()
    };
    insertar_("MOVIMIENTOS", movimiento);
    const admin = usuarioDesdeToken_(token, ["ADMIN"]);
    registrarAuditoria_(admin, "AJUSTE_INVENTARIO", "INVENTARIO", movimiento.id, tipo + " · " + cantidad + " " + unidad);
    return { ok: true, id: movimiento.id, data: movimiento, message: "Ajuste de inventario registrado." };
  } finally {
    lock.releaseLock();
  }
}


/* ============================================================
   COMPRAS DE INVENTARIO
   ============================================================ */

/** Registra una compra como entrada trazable del inventario. Solo ADMIN. */
function registrarCompraInventario(token, datos) {
  const user = usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, ["comedor_id", "alimento_id", "cantidad", "unidad"]);
  const cantidad = numeroPositivo_(datos.cantidad);
  const unidad = limpiar_(datos.unidad).toLowerCase();
  const comedor = exigirRegistro_("COMEDORES", datos.comedor_id);
  const alimento = exigirRegistro_("ALIMENTOS", datos.alimento_id);
  if (String(comedor.estado).toUpperCase() !== "ACTIVO") throw new Error("El comedor está inactivo.");
  if (String(alimento.estado).toUpperCase() !== "ACTIVO") throw new Error("El alimento está inactivo.");
  const normal = convertirAUnidadBase_(cantidad, unidad);
  const unidadAlimento = convertirAUnidadBase_(1, String(alimento.unidad || unidad).toLowerCase()).unidadBase;
  if (normal.unidadBase !== unidadAlimento) throw new Error("La unidad de compra no coincide con la unidad del alimento (" + alimento.unidad + ").");

  const lock = LockService.getScriptLock();
  lock.waitLock(15000);
  try {
    const movimiento = {
      id: nuevoId_("MOV"), comedor_id: comedor.id, alimento_id: alimento.id,
      tipo: "INGRESO_COMPRA", cantidad: cantidad, unidad: unidad,
      referencia_id: nuevoId_("COM"), observaciones: limpiar_(datos.observaciones || "Compra de inventario"),
      fecha: new Date()
    };
    insertar_("MOVIMIENTOS", movimiento);
    registrarAuditoria_(user, "REGISTRAR_COMPRA", "INVENTARIO", movimiento.id,
      "Compra de " + cantidad + " " + unidad + " de " + alimento.nombre + " para " + comedor.nombre);
    return { ok: true, id: movimiento.id, data: movimiento, message: "Compra registrada y añadida al inventario." };
  } finally { lock.releaseLock(); }
}

/* ============================================================
   CALENDARIO ALIMENTARIO
   ============================================================ */

function comedorDelUsuario_(user) {
  const comedor = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id) && String(c.estado).toUpperCase() === "ACTIVO");
  if (!comedor) throw new Error("No se encontró un comedor activo vinculado a tu usuario.");
  return comedor;
}

function obtenerInventarioCalendario(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let comedorId = filtros.comedor_id || "";
  if (user.rol === "COMEDOR") comedorId = comedorDelUsuario_(user).id;
  if (!comedorId) throw new Error("Selecciona un comedor para consultar el inventario.");
  if (user.rol === "ADMIN") exigirRegistro_("COMEDORES", comedorId);

  const movimientos = leerHoja_("MOVIMIENTOS").filter(m => String(m.comedor_id) === String(comedorId));
  const alimentos = leerHoja_("ALIMENTOS");
  const mapa = {};
  movimientos.forEach(m => {
    const food = alimentos.find(a => String(a.id) === String(m.alimento_id));
    if (!food) return;
    const cv = convertirAUnidadBase_(m.cantidad, m.unidad || food.unidad);
    const key = String(m.alimento_id) + "|" + cv.unidadBase;
    if (!mapa[key]) mapa[key] = { alimento_id:m.alimento_id, alimento:food.nombre, unidad:cv.unidadBase, disponible:0, compras:0, donaciones:0, otros:0 };
    const tipo = String(m.tipo || "").toUpperCase();
    const entrada = ["INGRESO_ENTREGA","INGRESO_COMPRA","ENTRADA_COMPRA","INGRESO","AJUSTE_POSITIVO"].includes(tipo);
    const salida = ["CONSUMO_PREPARACION","SALIDA","AJUSTE_NEGATIVO"].includes(tipo);
    if (entrada) mapa[key].disponible += cv.cantidadBase;
    if (salida) mapa[key].disponible -= cv.cantidadBase;
    if (["INGRESO_COMPRA","ENTRADA_COMPRA"].includes(tipo)) mapa[key].compras += cv.cantidadBase;
    else if (tipo === "INGRESO_ENTREGA") mapa[key].donaciones += cv.cantidadBase;
    else mapa[key].otros += (entrada ? cv.cantidadBase : salida ? -cv.cantidadBase : 0);
  });
  return { ok:true, data:Object.keys(mapa).map(k => { const x=mapa[k]; x.disponible=redondear_(x.disponible); x.compras=redondear_(x.compras); x.donaciones=redondear_(x.donaciones); x.otros=redondear_(x.otros); return x; }).filter(x => x.disponible > 0) };
}

function guardarEventoCalendario(token, datos) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  validarRequeridos_(datos, ["fecha", "tipo_comida", "nombre_plato", "porciones_planificadas", "ingredientes"]);
  if (!Array.isArray(datos.ingredientes) || !datos.ingredientes.length) throw new Error("Agrega al menos un ingrediente.");
  const comedor = user.rol === "COMEDOR" ? comedorDelUsuario_(user) : exigirRegistro_("COMEDORES", datos.comedor_id);
  if (String(comedor.estado).toUpperCase() !== "ACTIVO") throw new Error("El comedor está inactivo.");
  const fecha = fechaValida_(datos.fecha);
  const tipo = String(datos.tipo_comida).toUpperCase();
  if (!["DESAYUNO","ALMUERZO","CENA"].includes(tipo)) throw new Error("Tipo de comida no válido.");
  const porciones = numeroPositivo_(datos.porciones_planificadas);
  const foods = leerHoja_("ALIMENTOS");
  const ingredientesMap = {};
  datos.ingredientes.forEach(ing => {
    validarRequeridos_(ing, ["alimento_id","cantidad_gramos"]);
    const alimento = foods.find(a => String(a.id) === String(ing.alimento_id) && String(a.estado).toUpperCase() === "ACTIVO");
    if (!alimento) throw new Error("Ingrediente inexistente o inactivo: " + ing.alimento_id);
    const gramos = numeroPositivo_(ing.cantidad_gramos);
    if (!["g","kg","gr","gramo","gramos","kilogramo","kilogramos"].includes(String(alimento.unidad).toLowerCase())) throw new Error("El calendario requiere alimentos inventariados en gramos o kilogramos: " + alimento.nombre);
    if (!ingredientesMap[alimento.id]) ingredientesMap[alimento.id] = { alimento_id:alimento.id, nombre:alimento.nombre, cantidad_gramos:0 };
    ingredientesMap[alimento.id].cantidad_gramos += gramos;
  });
  const evento = { id:nuevoId_("CAL"), comedor_id:comedor.id, fecha:fecha, tipo_comida:tipo,
    nombre_plato:limpiar_(datos.nombre_plato), porciones_planificadas:porciones,
    ingredientes_json:JSON.stringify(Object.keys(ingredientesMap).map(k=>ingredientesMap[k])),
    observaciones:limpiar_(datos.observaciones), estado:"PROGRAMADO", preparacion_id:"",
    fecha_creacion:new Date(), fecha_actualizacion:new Date() };
  insertar_("CALENDARIO", evento);
  registrarAuditoria_(user, "PROGRAMAR_COMIDA", "CALENDARIO", evento.id, evento.nombre_plato + " · " + tipo);
  return { ok:true, id:evento.id, data:evento, message:"Comida programada correctamente. El inventario no se ha descontado." };
}

function listarEventosCalendario(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("CALENDARIO");
  if (user.rol === "COMEDOR") {
    const comedor = comedorDelUsuario_(user);
    lista = lista.filter(x => String(x.comedor_id) === String(comedor.id));
  } else if (filtros.comedor_id) lista = lista.filter(x => String(x.comedor_id) === String(filtros.comedor_id));
  if (filtros.estado) lista = lista.filter(x => String(x.estado).toUpperCase() === String(filtros.estado).toUpperCase());
  if (filtros.desde) { const d=fechaValida_(filtros.desde); lista=lista.filter(x=>new Date(x.fecha)>=d); }
  if (filtros.hasta) { const d=fechaValida_(filtros.hasta); d.setHours(23,59,59,999); lista=lista.filter(x=>new Date(x.fecha)<=d); }
  const comedores=leerHoja_("COMEDORES");
  lista.forEach(x=>{const c=comedores.find(y=>String(y.id)===String(x.comedor_id));x.comedor_nombre=c?c.nombre:"Comedor";});
  lista.sort((a,b)=>new Date(a.fecha)-new Date(b.fecha));
  return { ok:true, data:lista };
}

function cancelarEventoCalendario(token, eventoId) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  const lock=LockService.getScriptLock(); lock.waitLock(15000);
  try {
    const evento=exigirRegistro_("CALENDARIO", eventoId);
    if (user.rol === "COMEDOR" && String(evento.comedor_id)!==String(comedorDelUsuario_(user).id)) throw new Error("No puedes cancelar eventos de otro comedor.");
    if (String(evento.estado).toUpperCase() !== "PROGRAMADO") throw new Error("Solo se pueden cancelar eventos programados.");
    actualizar_("CALENDARIO", eventoId, {estado:"CANCELADO", fecha_actualizacion:new Date()});
    registrarAuditoria_(user, "CANCELAR_COMIDA", "CALENDARIO", eventoId, evento.nombre_plato);
    return {ok:true,id:eventoId,message:"Programación cancelada. No se modificó el inventario."};
  } finally { lock.releaseLock(); }
}

function registrarAuditoria_(user, accion, modulo, registroId, detalle) {
  try { insertar_("AUDITORIA", {id:nuevoId_("AUD"), usuario_id:user.id, usuario_nombre:user.nombre,
    rol:user.rol, accion:accion, modulo:modulo, registro_id:String(registroId||""), detalle:limpiar_(detalle), fecha:new Date()}); }
  catch (e) { console.error("No se pudo guardar auditoría: " + e.message); }
}

function listarAuditoria(token, filtros) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  filtros=filtros||{};
  let rows=leerHoja_("AUDITORIA");
  if (filtros.modulo) rows=rows.filter(x=>x.modulo===filtros.modulo);
  if (filtros.usuario_id) rows=rows.filter(x=>String(x.usuario_id)===String(filtros.usuario_id));
  if (filtros.desde) {const d=fechaValida_(filtros.desde);rows=rows.filter(x=>new Date(x.fecha)>=d);}
  if (filtros.hasta) {const d=fechaValida_(filtros.hasta);d.setHours(23,59,59,999);rows=rows.filter(x=>new Date(x.fecha)<=d);}
  rows.sort((a,b)=>new Date(b.fecha)-new Date(a.fecha));
  return {ok:true,data:rows.slice(0,500)};
}

/* ============================================================
   INFORMACIÓN NUTRICIONAL
   ============================================================ */

/**
 * Registra o actualiza valores nutricionales por cada 100 gramos.
 * datos: { alimento_id, calorias_100g, proteinas_100g,
 *          carbohidratos_100g, grasas_100g, fuente }
 */
function guardarInformacionNutricional(token, datos) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  validarRequeridos_(datos, [
    "alimento_id", "calorias_100g", "proteinas_100g",
    "carbohidratos_100g", "grasas_100g"
  ]);

  const alimento = exigirRegistro_("ALIMENTOS", datos.alimento_id);
  if (String(alimento.estado).toUpperCase() !== "ACTIVO") {
    throw new Error("No se puede registrar nutrición para un alimento inactivo.");
  }

  const nutricion = {
    alimento_id: String(datos.alimento_id),
    calorias_100g: numeroNoNegativo_(datos.calorias_100g),
    proteinas_100g: numeroNoNegativo_(datos.proteinas_100g),
    carbohidratos_100g: numeroNoNegativo_(datos.carbohidratos_100g),
    grasas_100g: numeroNoNegativo_(datos.grasas_100g),
    fuente: limpiar_(datos.fuente || "Fuente no especificada"),
    actualizado: new Date()
  };

  const lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    const existente = leerHoja_("NUTRICION").some(n =>
      String(n.alimento_id) === String(datos.alimento_id)
    );
    if (existente) {
      actualizar_("NUTRICION", datos.alimento_id, nutricion, "alimento_id");
    } else {
      insertar_("NUTRICION", nutricion);
    }
  } finally {
    lock.releaseLock();
  }

  return { ok: true, data: nutricion, message: "Información nutricional guardada por cada 100 gramos." };
}

/** ADMIN, DONANTE y COMEDOR pueden consultar los datos nutricionales. */
function obtenerInformacionNutricional(token, alimentoId) {
  usuarioDesdeToken_(token, ["ADMIN", "DONANTE", "COMEDOR"]);
  const item = leerHoja_("NUTRICION").find(n => String(n.alimento_id) === String(alimentoId));
  if (!item) return { ok: true, data: null, message: "Este alimento aún no tiene información nutricional registrada." };
  return { ok: true, data: item };
}


/* ============================================================
   PREPARACIONES Y CÁLCULO NUTRICIONAL
   ============================================================ */

/**
 * Registra una preparación, descuenta sus ingredientes del inventario
 * y calcula nutrientes por preparación y por porción.
 *
 * datos: {
 *   comedor_id (solo ADMIN; el COMEDOR usa su propio perfil),
 *   nombre_plato, fecha, porciones,
 *   ingredientes: [{ alimento_id, cantidad_gramos }],
 *   observaciones
 * }
 *
 * La cantidad de cada ingrediente siempre se expresa en gramos.
 * El inventario del alimento debe estar en g o kg.
 */
function registrarPreparacion(token, datos) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  validarRequeridos_(datos, ["nombre_plato", "porciones", "ingredientes"]);
  if (!Array.isArray(datos.ingredientes) || datos.ingredientes.length === 0) {
    throw new Error("Agrega al menos un ingrediente a la preparación.");
  }

  let comedor;
  if (user.rol === "COMEDOR") {
    comedor = leerHoja_("COMEDORES").find(c =>
      String(c.usuario_id) === String(user.id) && String(c.estado).toUpperCase() === "ACTIVO"
    );
    if (!comedor) throw new Error("Tu usuario no tiene un comedor activo.");
  } else {
    validarRequeridos_(datos, ["comedor_id"]);
    comedor = exigirRegistro_("COMEDORES", datos.comedor_id);
  }

  const porciones = numeroPositivo_(datos.porciones);
  const nutriciones = leerHoja_("NUTRICION");
  const alimentos = leerHoja_("ALIMENTOS");

  // Consolidar ingredientes repetidos y validar antes de modificar hojas.
  const ingredientesMap = {};
  datos.ingredientes.forEach(ing => {
    validarRequeridos_(ing, ["alimento_id", "cantidad_gramos"]);
    const gramos = numeroPositivo_(ing.cantidad_gramos);
    const alimento = alimentos.find(a => String(a.id) === String(ing.alimento_id));
    if (!alimento || String(alimento.estado).toUpperCase() !== "ACTIVO") {
      throw new Error("Uno de los ingredientes no existe o está inactivo: " + ing.alimento_id);
    }
    if (!nutriciones.some(n => String(n.alimento_id) === String(ing.alimento_id))) {
      throw new Error("Falta registrar información nutricional para: " + alimento.nombre);
    }
    if (!ingredientesMap[ing.alimento_id]) {
      ingredientesMap[ing.alimento_id] = {
        alimento_id: String(ing.alimento_id),
        nombre: alimento.nombre,
        cantidad_gramos: 0,
        unidad_inventario: String(alimento.unidad || "").toLowerCase()
      };
    }
    ingredientesMap[ing.alimento_id].cantidad_gramos += gramos;
  });

  const ingredientes = Object.keys(ingredientesMap).map(k => ingredientesMap[k]);
  const lock = LockService.getScriptLock();
  lock.waitLock(20000);
  try {
    let eventoCalendario = null;
    if (datos.calendario_id) {
      eventoCalendario = exigirRegistro_("CALENDARIO", datos.calendario_id);
      if (String(eventoCalendario.comedor_id) !== String(comedor.id)) throw new Error("La programación no pertenece a este comedor.");
      if (String(eventoCalendario.estado).toUpperCase() !== "PROGRAMADO") throw new Error("Esta programación ya fue preparada o cancelada.");
      const yaExiste = leerHoja_("PREPARACIONES").some(p => String(p.calendario_id || "") === String(datos.calendario_id));
      if (yaExiste) throw new Error("Esta programación ya tiene una preparación registrada.");
    }
    // Comprobar existencias de todos los ingredientes antes de descontar alguno.
    ingredientes.forEach(ing => {
      const stock = calcularStockAlimento_(comedor.id, ing.alimento_id);
      const requerido = convertirAUnidadBase_(ing.cantidad_gramos, "g");
      if (stock.unidadBase !== requerido.unidadBase ||
          stock.cantidadBase + 0.000001 < requerido.cantidadBase) {
        throw new Error(
          "Inventario insuficiente para " + ing.nombre +
          ". Disponible: " + redondear_(stock.cantidadBase) + " " + stock.unidadBase +
          "; requerido: " + redondear_(requerido.cantidadBase) + " " + requerido.unidadBase + "."
        );
      }
    });

    let calorias = 0, proteinas = 0, carbohidratos = 0, grasas = 0;
    ingredientes.forEach(ing => {
      const n = nutriciones.find(x => String(x.alimento_id) === String(ing.alimento_id));
      const factor = ing.cantidad_gramos / 100;
      calorias += numeroNoNegativo_(n.calorias_100g) * factor;
      proteinas += numeroNoNegativo_(n.proteinas_100g) * factor;
      carbohidratos += numeroNoNegativo_(n.carbohidratos_100g) * factor;
      grasas += numeroNoNegativo_(n.grasas_100g) * factor;
    });

    const fecha = datos.fecha ? fechaValida_(datos.fecha) : new Date();
    const prepId = nuevoId_("PRE");
    const preparacion = {
      id: prepId,
      comedor_id: comedor.id,
      nombre_plato: limpiar_(datos.nombre_plato),
      fecha: fecha,
      porciones: porciones,
      ingredientes_json: JSON.stringify(ingredientes),
      calorias_por_porcion: redondear_(calorias / porciones),
      proteinas_por_porcion: redondear_(proteinas / porciones),
      carbohidratos_por_porcion: redondear_(carbohidratos / porciones),
      grasas_por_porcion: redondear_(grasas / porciones),
      observaciones: limpiar_(datos.observaciones),
      fecha_creacion: new Date(),
      calendario_id: datos.calendario_id || ""
    };

    // Registrar movimientos y preparación bajo bloqueo. Si hay fallo parcial, revertir
    // los movimientos insertados para no dejar un descuento sin preparación.
    const movimientosCreados = [];
    let preparacionInsertada = false;
    try {
      ingredientes.forEach(ing => {
      const movimiento = {
        id: nuevoId_("MOV"),
        comedor_id: comedor.id,
        alimento_id: ing.alimento_id,
        tipo: "CONSUMO_PREPARACION",
        cantidad: ing.cantidad_gramos,
        unidad: "g",
        referencia_id: prepId,
        observaciones: "Consumo para preparación: " + preparacion.nombre_plato,
        fecha: new Date()
      };
      insertar_("MOVIMIENTOS", movimiento);
      movimientosCreados.push(movimiento.id);
      });
      insertar_("PREPARACIONES", preparacion);
      preparacionInsertada = true;
      if (eventoCalendario) {
        actualizar_("CALENDARIO", eventoCalendario.id, {estado:"PREPARADO", preparacion_id:prepId, fecha_actualizacion:new Date()});
      }
    } catch (errorRegistro) {
      if (preparacionInsertada) eliminarRegistroPorId_("PREPARACIONES", prepId);
      movimientosCreados.forEach(id => eliminarRegistroPorId_("MOVIMIENTOS", id));
      throw new Error("No se pudo completar la preparación; se revirtieron los registros insertados. " + errorRegistro.message);
    }
    registrarAuditoria_(user, "REGISTRAR_PREPARACION", "PREPARACIONES", prepId, preparacion.nombre_plato);

    return {
      ok: true,
      id: prepId,
      data: preparacion,
      resumen_nutricional: {
        calorias_totales: redondear_(calorias),
        proteinas_totales: redondear_(proteinas),
        carbohidratos_totales: redondear_(carbohidratos),
        grasas_totales: redondear_(grasas),
        calorias_por_porcion: preparacion.calorias_por_porcion,
        proteinas_por_porcion: preparacion.proteinas_por_porcion,
        carbohidratos_por_porcion: preparacion.carbohidratos_por_porcion,
        grasas_por_porcion: preparacion.grasas_por_porcion
      },
      message: "Preparación registrada y consumo descontado del inventario."
    };
  } finally {
    lock.releaseLock();
  }
}

/** Lista preparaciones. COMEDOR solo ve las propias. */
function listarPreparaciones(token, filtros) {
  const user = usuarioDesdeToken_(token, ["ADMIN", "COMEDOR"]);
  filtros = filtros || {};
  let lista = leerHoja_("PREPARACIONES");

  if (user.rol === "COMEDOR") {
    const perfil = leerHoja_("COMEDORES").find(c => String(c.usuario_id) === String(user.id));
    if (!perfil) return { ok: true, data: [] };
    lista = lista.filter(p => String(p.comedor_id) === String(perfil.id));
  } else if (filtros.comedor_id) {
    lista = lista.filter(p => String(p.comedor_id) === String(filtros.comedor_id));
  }

  if (filtros.buscar) {
    const q = String(filtros.buscar).toLowerCase();
    lista = lista.filter(p => String(p.nombre_plato).toLowerCase().indexOf(q) !== -1);
  }
  lista.sort((a, b) => new Date(b.fecha).getTime() - new Date(a.fecha).getTime());
  return { ok: true, data: lista };
}


/* ============================================================
   DASHBOARD Y REPORTES
   ============================================================ */

/**
 * Dashboard general para ADMIN. Opcional: filtrar por comedor_id.
 * Los totales de kilos solo incluyen movimientos expresados en g o kg.
 */
function obtenerDashboard(token, filtros) {
  usuarioDesdeToken_(token, ["ADMIN"]);
  filtros = filtros || {};

  const comedorId = filtros.comedor_id || "";
  const movimientos = leerHoja_("MOVIMIENTOS").filter(m =>
    !comedorId || String(m.comedor_id) === String(comedorId)
  );
  const donaciones = leerHoja_("DONACIONES");
  const solicitudes = leerHoja_("SOLICITUDES");
  const comedores = leerHoja_("COMEDORES");
  const donantes = leerHoja_("DONANTES");
  const preparaciones = leerHoja_("PREPARACIONES");

  const ingresosKg = movimientos.reduce((total, m) => {
    if (["INGRESO_ENTREGA", "INGRESO_COMPRA", "ENTRADA_COMPRA", "INGRESO", "AJUSTE_POSITIVO"].indexOf(String(m.tipo).toUpperCase()) === -1) return total;
    const c = convertirAUnidadBase_(Number(m.cantidad), String(m.unidad || "").toLowerCase());
    return c.unidadBase === "g" ? total + c.cantidadBase / 1000 : total;
  }, 0);

  const consumidosKg = movimientos.reduce((total, m) => {
    if (["CONSUMO_PREPARACION", "SALIDA", "AJUSTE_NEGATIVO"].indexOf(String(m.tipo).toUpperCase()) === -1) return total;
    const c = convertirAUnidadBase_(Number(m.cantidad), String(m.unidad || "").toLowerCase());
    return c.unidadBase === "g" ? total + c.cantidadBase / 1000 : total;
  }, 0);

  const hoy = new Date();
  hoy.setHours(0, 0, 0, 0);
  const limite = new Date(hoy);
  limite.setDate(limite.getDate() + 7);
  const proximosVencer = donaciones.filter(d => {
    if (String(d.estado).toUpperCase() !== "DISPONIBLE" || Number(d.cantidad_disponible) <= 0) return false;
    const f = new Date(d.fecha_vencimiento);
    return !isNaN(f.getTime()) && f >= hoy && f <= limite;
  });

  const solicitudesPendientes = solicitudes.filter(s => String(s.estado).toUpperCase() === "PENDIENTE");
  const entregasRealizadas = movimientos.filter(m => String(m.tipo).toUpperCase() === "INGRESO_ENTREGA");
  const donacionesDisponibles = donaciones.filter(d =>
    String(d.estado).toUpperCase() === "DISPONIBLE" && Number(d.cantidad_disponible) > 0
  );

  return {
    ok: true,
    data: {
      total_donaciones: donaciones.length,
      donaciones_disponibles: donacionesDisponibles.length,
      total_comedores: comedores.filter(c => String(c.estado).toUpperCase() === "ACTIVO").length,
      total_donantes: donantes.filter(d => String(d.estado).toUpperCase() === "ACTIVO").length,
      solicitudes_pendientes: solicitudesPendientes.length,
      entregas_realizadas: entregasRealizadas.length,
      alimentos_proximos_a_vencer: proximosVencer.length,
      preparaciones_registradas: preparaciones.length,
      kilos_ingresados_a_comedores: redondear_(ingresosKg),
      kilos_consumidos_en_preparaciones: redondear_(consumidosKg),
      kilos_estimados_disponibles: redondear_(ingresosKg - consumidosKg),
      solicitudes: solicitudesPendientes,
      proximos_vencer: proximosVencer
    }
  };
}


/* ============================================================
   PUNTO DE ENTRADA DE LA APLICACIÓN WEB
   ============================================================ */

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({
      ok: true,
      servicio: "QARANA API",
      version: AS.VERSION,
      mensaje: "API activa. Envía solicitudes POST desde el frontend."
    }))
    .setMimeType(ContentService.MimeType.JSON);
}

/** Ejecutar manualmente con datos propios: crearAdminQarana("correo", "contraseña-larga", "Nombre"). */
function crearAdminQarana(correo, contrasena, nombre) {
  if (!correo || !contrasena) throw new Error("Por seguridad, ejecuta crearAdminQarana(correo, contrasena, nombre) con credenciales propias. No hay credenciales predeterminadas.");
  return crearAdministradorInicial(correo, contrasena, nombre || "Administrador");
}


/* ============================================================
   FUNCIONES INTERNAS
   ============================================================ */

function nombreHoja_(key) {
  const map = {
    USUARIOS: "USUARIOS", COMEDORES: "COMEDORES", DONANTES: "DONANTES",
    ALIMENTOS: "ALIMENTOS", DONACIONES: "DONACIONES", SOLICITUDES: "SOLICITUDES",
    MOVIMIENTOS: "MOVIMIENTOS", NUTRICION: "NUTRICION",
    PREPARACIONES: "PREPARACIONES", CALENDARIO: "CALENDARIO", AUDITORIA: "AUDITORIA"
  };
  return map[key];
}

function hoja_(key) {
  const sh = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(nombreHoja_(key));
  if (!sh) throw new Error("No existe la hoja " + nombreHoja_(key) + ". Ejecuta configurarSistema().");
  return sh;
}

function leerHoja_(key) {
  const sh = hoja_(key);
  const values = sh.getDataRange().getValues();
  if (values.length < 2) return [];
  const headers = values[0];
  return values.slice(1).filter(row => row.some(v => v !== "")).map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i] instanceof Date ? row[i].toISOString() : row[i]);
    return obj;
  });
}

/** Evita que textos que empiezan con = + - @ se interpreten como fórmulas. */
function valorSeguro_(v) {
  if (typeof v === "string" && /^[=+\-@]/.test(v)) return "'" + v;
  return v;
}

function insertar_(key, objeto) {
  const sh = hoja_(key);
  const headers = sh.getRange(1, 1, 1, sh.getLastColumn()).getValues()[0];
  const row = headers.map(h => Object.prototype.hasOwnProperty.call(objeto, h) ? valorSeguro_(objeto[h]) : "");
  sh.appendRow(row);
  return { ok: true, id: objeto.id || objeto.alimento_id, data: objeto };
}

function actualizar_(key, id, cambios, campoId) {
  const sh = hoja_(key);
  const values = sh.getDataRange().getValues();
  const headers = values[0];
  const idCol = headers.indexOf(campoId || "id");
  if (idCol < 0) throw new Error("No se encontró la columna identificadora.");
  for (let r = 1; r < values.length; r++) {
    if (String(values[r][idCol]) === String(id)) {
      Object.keys(cambios).forEach(k => {
        const c = headers.indexOf(k);
        if (c >= 0) sh.getRange(r + 1, c + 1).setValue(valorSeguro_(cambios[k]));
      });
      return { ok: true, id: id };
    }
  }
  throw new Error("No se encontró el registro: " + id);
}

function eliminarRegistroPorId_(key, id, campoId) {
  const sh=hoja_(key), values=sh.getDataRange().getValues(), headers=values[0];
  const col=headers.indexOf(campoId || "id");
  if (col<0) throw new Error("No se encontró la columna identificadora para revertir " + key + ".");
  for (let i=values.length-1;i>=1;i--) {
    if (String(values[i][col])===String(id)) { sh.deleteRow(i+1); return true; }
  }
  return false;
}

function exigirRegistro_(key, id) {
  const item = leerHoja_(key).find(x => String(x.id) === String(id) ||
    (key === "NUTRICION" && String(x.alimento_id) === String(id)));
  if (!item) throw new Error("No existe el registro " + id + " en " + nombreHoja_(key));
  return item;
}

function usuarioDesdeToken_(token, rolesPermitidos) {
  if (!token) throw new Error("Sesión no válida. Inicia sesión nuevamente.");
  const raw = CacheService.getScriptCache().get("AS_TOKEN_" + token);
  if (!raw) throw new Error("La sesión expiró. Inicia sesión nuevamente.");
  const user = JSON.parse(raw);
  if (rolesPermitidos && rolesPermitidos.indexOf(user.rol) === -1) {
    throw new Error("No tienes permisos para realizar esta operación.");
  }
  return user;
}

function hashPassword_(password, salt) {
  const bytes = Utilities.computeDigest(
    Utilities.DigestAlgorithm.SHA_256, String(salt) + String(password), Utilities.Charset.UTF_8
  );
  return Utilities.base64Encode(bytes);
}

function nuevoId_(prefijo) {
  return prefijo + "-" + Utilities.getUuid().split("-")[0].toUpperCase();
}

function validarRequeridos_(obj, campos) {
  if (!obj || typeof obj !== "object") throw new Error("Faltan los datos de entrada.");
  campos.forEach(c => {
    if (obj[c] === undefined || obj[c] === null || obj[c] === "") {
      throw new Error("El campo " + c + " es obligatorio.");
    }
  });
}

function limpiar_(v) {
  return v === undefined || v === null ? "" : String(v).trim();
}

function normalizarCorreo_(v) {
  return String(v || "").trim().toLowerCase();
}

function numeroNormalizado_(v) {
  if (typeof v === "number") return v;
  let s = String(v === undefined || v === null ? "" : v).trim().replace(/\s/g, "");
  if (!s) return NaN;
  // Admite coma decimal y formatos con separador de miles.
  if (s.indexOf(",") !== -1 && s.indexOf(".") !== -1) {
    if (s.lastIndexOf(",") > s.lastIndexOf(".")) s = s.replace(/\./g, "").replace(",", ".");
    else s = s.replace(/,/g, "");
  } else if (s.indexOf(",") !== -1) {
    s = s.replace(",", ".");
  }
  return Number(s);
}

function numeroPositivo_(v) {
  const n = numeroNormalizado_(v);
  if (!isFinite(n) || n <= 0) throw new Error("La cantidad debe ser un número mayor que cero.");
  return n;
}

function numeroNoNegativo_(v) {
  const n = numeroNormalizado_(v);
  if (!isFinite(n) || n < 0) throw new Error("El valor debe ser un número igual o mayor que cero.");
  return n;
}

/** Interpreta fechas AAAA-MM-DD en hora local (evita desfase de un día por UTC). */
function fechaValida_(v) {
  if (!v) return "";
  let d;
  if (v instanceof Date) {
    d = v;
  } else {
    const s = String(v).trim();
    const m = s.match(/^(\d{4})-(\d{2})-(\d{2})$/);
    d = m ? new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3])) : new Date(s);
  }
  if (isNaN(d.getTime())) throw new Error("La fecha indicada no es válida.");
  return d;
}

function redondear_(n) {
  return Math.round(Number(n) * 100) / 100;
}

/**
 * Convierte cantidades de kg a g. Las unidades no masivas se conservan
 * en su propia unidad y no se mezclan con gramos/kilogramos.
 */
function convertirAUnidadBase_(cantidad, unidad) {
  const u = String(unidad || "").trim().toLowerCase();
  const n = numeroNormalizado_(cantidad);
  if (!isFinite(n) || n < 0) throw new Error("Cantidad de movimiento no válida.");

  if (["kg", "kilo", "kilos", "kilogramo", "kilogramos"].indexOf(u) !== -1) {
    return { cantidadBase: n * 1000, unidadBase: "g" };
  }
  if (["g", "gr", "gramo", "gramos"].indexOf(u) !== -1) {
    return { cantidadBase: n, unidadBase: "g" };
  }
  return { cantidadBase: n, unidadBase: u || "unidad" };
}

/** Calcula stock de un alimento para un comedor en unidad base. */
function calcularStockAlimento_(comedorId, alimentoId) {
  const movimientos = leerHoja_("MOVIMIENTOS").filter(m =>
    String(m.comedor_id) === String(comedorId) &&
    String(m.alimento_id) === String(alimentoId)
  );

  let cantidadBase = 0;
  let unidadBase = "";
  movimientos.forEach(m => {
    const convertido = convertirAUnidadBase_(Number(m.cantidad), String(m.unidad || "").toLowerCase());
    if (!unidadBase) unidadBase = convertido.unidadBase;
    if (unidadBase !== convertido.unidadBase) {
      throw new Error("Hay movimientos con unidades incompatibles para el mismo alimento. Revisa el historial.");
    }

    const tipo = String(m.tipo).toUpperCase();
    if (["INGRESO_ENTREGA", "INGRESO_COMPRA", "ENTRADA_COMPRA", "INGRESO", "AJUSTE_POSITIVO"].indexOf(tipo) !== -1) {
      cantidadBase += convertido.cantidadBase;
    } else if (["CONSUMO_PREPARACION", "SALIDA", "AJUSTE_NEGATIVO"].indexOf(tipo) !== -1) {
      cantidadBase -= convertido.cantidadBase;
    }
  });

  return { cantidadBase: redondear_(cantidadBase), unidadBase: unidadBase || "g" };
}

/* ============================================================
   API HTTP PARA GITHUB PAGES
   GitHub Pages -> Apps Script Web App -> Google Sheets
   ============================================================ */

/**
 * Endpoint POST para el frontend independiente.
 * El navegador envía un formulario HTML normal a un iframe oculto y recibe
 * la respuesta mediante postMessage. Así no se depende de google.script.run
 * ni de fetch/CORS en el navegador.
 *
 * La lista blanca QARANA_API_METODOS_ limita las operaciones expuestas.
 * Las funciones de negocio siguen validando token y rol de usuario.
 */
function doPost(e) {
  let request = null;
  let targetOrigin = "";
  try {
    const raw = e && e.parameter && e.parameter.qaranaRequest
      ? String(e.parameter.qaranaRequest)
      : "";
    if (!raw) throw new Error("Solicitud vacía.");
    if (raw.length > 1000000) throw new Error("La solicitud supera el tamaño permitido.");

    request = JSON.parse(raw);
    targetOrigin = String(request.clientOrigin || "");
    const requestId = String(request.requestId || "");
    if (!/^[a-zA-Z0-9_-]{8,100}$/.test(requestId)) {
      throw new Error("Identificador de solicitud no válido.");
    }
    if (!origenPermitidoQarana_(targetOrigin)) {
      return respuestaHtmlQarana_({
        type: "QARANA_API_RESPONSE", requestId, ok: false,
        error: "Origen no autorizado. Configura el dominio de GitHub Pages en QARANA_ALLOWED_ORIGINS."
      }, targetOrigin);
    }

    const metodo = String(request.method || "");
    const args = Array.isArray(request.args) ? request.args : [];
    const ejecutor = QARANA_API_METODOS_[metodo];
    if (!ejecutor) throw new Error("Operación no permitida: " + metodo);
    if (args.length > 8) throw new Error("Demasiados parámetros.");

    const resultado = ejecutor(args);
    return respuestaHtmlQarana_({
      type: "QARANA_API_RESPONSE", requestId, ok: true, result: resultado
    }, targetOrigin);
  } catch (error) {
    const mensaje = error && error.message ? error.message : String(error);
    const requestId = request && request.requestId ? String(request.requestId) : "";
    if (targetOrigin && requestId) {
      return respuestaHtmlQarana_({
        type: "QARANA_API_RESPONSE", requestId, ok: false, error: mensaje
      }, targetOrigin);
    }
    return HtmlService.createHtmlOutput("QARANA API: solicitud no válida.");
  }
}

/**
 * Orígenes admitidos. Por defecto permite páginas HTTPS de github.io y localhost
 * para pruebas. Para restringirlo a un dominio concreto, define en Propiedades
 * del proyecto la propiedad QARANA_ALLOWED_ORIGINS con uno o varios orígenes
 * separados por comas, por ejemplo: https://usuario.github.io
 */
function origenPermitidoQarana_(origin) {
  if (!origin) return false;
  const configurados = PropertiesService.getScriptProperties()
    .getProperty("QARANA_ALLOWED_ORIGINS");
  if (configurados) {
    const lista = configurados.split(",").map(x => x.trim()).filter(Boolean);
    return lista.indexOf(origin) !== -1;
  }
  return /^https:\/\/[a-z0-9-]+\.github\.io$/i.test(origin) ||
         /^http:\/\/localhost(?::\d+)?$/i.test(origin);
}

/** Devuelve una página mínima que envía la respuesta al frontend que llamó. */
function respuestaHtmlQarana_(payload, targetOrigin) {
  if (!/^https?:\/\//i.test(targetOrigin)) {
    return HtmlService.createHtmlOutput("QARANA API: origen de respuesta no válido.");
  }
  const safePayload = JSON.stringify(payload)
    .replace(/</g, "\\u003c")
    .replace(/>/g, "\\u003e")
    .replace(/&/g, "\\u0026")
    .replace(/\u2028/g, "\\u2028")
    .replace(/\u2029/g, "\\u2029");
  const safeOrigin = JSON.stringify(targetOrigin);
  const html = "<!doctype html><html><head><meta charset='utf-8'></head><body><script>" +
    "(function(){var data=" + safePayload + ";var origin=" + safeOrigin + ";" +
    "try{window.top.postMessage(data,origin);}catch(e){try{window.parent.postMessage(data,origin);}catch(_){}}" +
    "})();</script></body></html>";
  return HtmlService.createHtmlOutput(html)
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

/**
 * Lista blanca de funciones públicas utilizadas por Index.html.
 * No usar un nombre de función recibido del navegador sin esta validación.
 */
const QARANA_API_METODOS_ = {
  iniciarSesion: a => iniciarSesion.apply(null, a),
  cerrarSesion: a => cerrarSesion.apply(null, a),
  obtenerSesion: a => obtenerSesion.apply(null, a),

  crearUsuario: a => crearUsuario.apply(null, a),
  crearComedor: a => crearComedor.apply(null, a),
  listarComedores: a => listarComedores.apply(null, a),
  crearDonante: a => crearDonante.apply(null, a),
  listarDonantes: a => listarDonantes.apply(null, a),
  crearAlimento: a => crearAlimento.apply(null, a),
  listarAlimentos: a => listarAlimentos.apply(null, a),
  crearDonacion: a => crearDonacion.apply(null, a),
  listarDonaciones: a => listarDonaciones.apply(null, a),
  listarSolicitudes: a => listarSolicitudes.apply(null, a),
  resolverSolicitud: a => resolverSolicitud.apply(null, a),
  registrarEntrega: a => registrarEntrega.apply(null, a),
  obtenerDashboard: a => obtenerDashboard.apply(null, a),
  obtenerInventario: a => obtenerInventario.apply(null, a),
  registrarCompraInventario: a => registrarCompraInventario.apply(null, a),
  registrarAjusteInventario: a => registrarAjusteInventario.apply(null, a),
  guardarInformacionNutricional: a => guardarInformacionNutricional.apply(null, a),
  obtenerInformacionNutricional: a => obtenerInformacionNutricional.apply(null, a),
  listarPreparaciones: a => listarPreparaciones.apply(null, a),
  registrarPreparacion: a => registrarPreparacion.apply(null, a),
  guardarEventoCalendario: a => guardarEventoCalendario.apply(null, a),
  listarEventosCalendario: a => listarEventosCalendario.apply(null, a),
  cancelarEventoCalendario: a => cancelarEventoCalendario.apply(null, a),
  obtenerInventarioCalendario: a => obtenerInventarioCalendario.apply(null, a),
  listarAuditoria: a => listarAuditoria.apply(null, a)
};
```
