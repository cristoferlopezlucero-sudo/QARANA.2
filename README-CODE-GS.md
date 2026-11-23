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
