# Bitácora de simuladores · Centro de Simulación Clínica (CSC)

Página para el **personal del CSC**: <https://rebecaleu.github.io/bitacora-simuladores/>

Es un proyecto aparte del sitio de solicitudes (`Centro_simul`). HTML, CSS y JavaScript puro (módulos ES):
sin frameworks, sin build y sin CDN; Firebase y MSAL están copiados en `js/vendor/`.

Cada equipo lleva una tarjeta NFC. Al leerla se registra cuándo sale, quién lo lleva, cuándo regresa,
en qué cuarto se deja y su verificación. **Todo se escribe también en el Excel del inventario.**

- **Plano:** los cuartos del inventario con sus simuladores, y la lista "Fuera ahora" (en rojo si pasan
  `bitacora.horasAlerta` horas fuera).
- **Ficha del equipo** (al leer su tarjeta o tocarlo): registrar salida, registrar regreso (cuarto donde se deja
  y si necesita revisión), cambiarlo de cuarto y verificarlo.
- **Verificar:** en "Leer tarjeta" se elige **Verificar**; cada tarjeta que se lea abre el formulario de verificación.
- **Inventario:** los simuladores o todo el inventario, con buscador.
- **Historial:** todos los movimientos, si ya están en el Excel, y descarga en CSV.
- **Ajustes:** conexión con el Excel, cómo se ven los cuartos, tarjetas de cuarto y sesión.

## Dónde vive cada dato

| Dato | Dónde |
|---|---|
| Equipos, IDs (`EQ-…`), nombre, categoría, cuarto, caja | **Excel del inventario** (hoja *1Inventario*). Los equipos se dan de alta ahí, no en la página. |
| Quién tiene cada equipo, desde cuándo, tarjeta NFC, historial | **Firebase** (en vivo, para todo el personal). Sin Firebase: modo demostración en ese navegador. |
| Cada movimiento | Se agrega como fila en la hoja **Movimientos** del Excel (la página la crea si no existe). |
| Regreso o cambio de cuarto | Actualiza **Zona del cuarto donde estaba** en *1Inventario*. |
| Verificación | En *1Inventario*: Estado general, # piezas dañadas, Material que falta, Revisado por, Fecha de revisión y se agrega el comentario a Notas. En la hoja **Verificación**: Estado físico, Funciona, Ajustes, Resultado, Comentarios, y el encabezado Fecha / Realizó / Hora. |

Si no hay internet o nadie ha conectado su cuenta del Tec, el movimiento queda guardado en la bitácora como
**pendiente** y se pasa al Excel después (Ajustes → "Pasar al Excel ahora"). No se repiten filas al reintentar.

## Antes de conectar el Excel

1. **IDs en la hoja Verificación:** la columna **ID** (B) debe tener el ID de cada equipo, igual que en *1Inventario*.
   Si un equipo no tiene ID ahí, la verificación solo se escribe en *1Inventario*.
2. **Opciones de verificación:** en `js/config.js` → `bitacora.excel.verificacion.opciones` deben ir las mismas opciones
   que las listas del Excel (hoy: Estado físico Bueno/Regular/Malo, Funciona Sí/No, Resultado Pasa/No pasa). **Por confirmar.**
3. **Bitácora de movimientos:** si ya existe una hoja de bitácora en el Excel, poner su nombre en
   `bitacora.excel.bitacora.hoja` y sus encabezados en `bitacora.excel.bitacora.columnas`. Si no, la página crea
   la hoja **Movimientos** con: Fecha, Hora, ID equipo, Equipo, Movimiento, Responsable, Destino / actividad, Cuarto,
   Tiempo fuera (min), Notas, Registró, Folio.

Las columnas se buscan por su **encabezado** (sin importar acentos ni mayúsculas), no por letra: se pueden mover.

## Conectar el Excel (Microsoft 365 del Tec)

1. Entrar a <https://entra.microsoft.com> con la cuenta del Tec → **Aplicaciones → Registros de aplicaciones → Nuevo registro**.
   - Nombre: *Bitácora de simuladores CSC*.
   - Tipos de cuenta: **Solo cuentas de este directorio organizativo** (o "cualquier directorio organizativo").
   - URI de redirección: plataforma **Aplicación de página única (SPA)**:
     `https://rebecaleu.github.io/bitacora-simuladores/`
2. En la app: **Autenticación → Agregar URI** (SPA): `https://rebecaleu.github.io/bitacora-simuladores/vacio.html`
3. **Permisos de API → Microsoft Graph → Permisos delegados:** `User.Read` y `Files.ReadWrite.All`.
   Si el Tec no deja que cada persona lo autorice, **TI tiene que dar "consentimiento de administrador"**.
   Si no se permite registrar apps, pedirle a TI que la registre con estos mismos datos.
4. Copiar el **Id. de aplicación (cliente)** en `js/config.js` → `bitacora.microsoft.clientId`
   (y, si se quiere limitar al Tec, el **Id. de directorio (inquilino)** en `tenant`).
5. En cada Excel: **Compartir → Copiar vínculo** (con permiso de **edición** para el personal) y pegarlo en
   `bitacora.excel.inventario.enlace` y `bitacora.excel.verificacion.enlace`.
6. Subir el cambio a `main`. En la bitácora: **Ajustes → Conectar cuenta del Tec**.

Cada persona que registre movimientos necesita **permiso de edición** en los dos Excel.

## Firebase (para que todo el personal vea lo mismo)

Mientras `bitacora.firebase` en `js/config.js` esté vacío, la bitácora funciona en **modo demostración**:
se guarda solo en ese navegador. Sirve para probar.

1. Entrar a <https://console.firebase.google.com> → **Crear un proyecto** (Analytics no hace falta).
2. **Authentication → Comenzar → Correo electrónico/contraseña → Habilitar.**
3. **Authentication → Usuarios → Agregar usuario**: un correo y contraseña por cada persona del personal.
4. **Firestore Database → Crear base de datos** → modo de producción → ubicación cercana (por ejemplo `nam5`).
5. **Firestore Database → Reglas**: pegar el contenido de `firestore.rules`, poner los correos del
   personal en la lista y **Publicar**.
6. **Configuración del proyecto** (engrane) → **Tus apps → Web (`</>`)** → registrar la app y copiar
   `apiKey`, `authDomain`, `projectId` y `appId` en `js/config.js` → `bitacora.firebase`.
7. **Authentication → Configuración → Dominios autorizados** → agregar `rebecaleu.github.io`.
8. Subir el cambio a `main`. Al abrir la bitácora pedirá correo y contraseña.

La `apiKey` de Firebase **no es secreta** (va en todas las páginas que usan Firebase): lo que protege
los datos son las cuentas y las reglas de `firestore.rules`.

**Dar acceso a alguien nuevo:** crear su usuario (paso 3), agregar su correo a las reglas (paso 5) y
compartirle los Excel con permiso de edición.

## Tarjetas NFC

Sirven etiquetas NFC comunes **NTAG213 o NTAG215**. Cada tarjeta puede llevar el **enlace** del equipo
(`https://rebecaleu.github.io/bitacora-simuladores/#EQ-347`), su **número de serie** guardado en la bitácora, o las dos cosas.
La página también entiende el enlace de los QR del inventario (`…/equipo/?id=EQ-347`).

| Dónde | Leer | Grabar el enlace |
|---|---|---|
| **Android con Chrome** | Directo en la página (botón "Leer tarjeta"; la primera vez pide permiso de NFC). | Ficha del equipo → **Tarjeta NFC → Grabar ahora**. |
| **iPhone** | Acercar el iPhone a la tarjeta grabada: abre la ficha del equipo. | Con una app como NFC Tools (la ficha muestra el enlace para copiar). |
| **Computadora** | Lector USB que escriba como teclado (modo HID / *keyboard emulation*). | — |

Si se lee una tarjeta que no está registrada, la página pregunta de qué equipo (o cuarto) es.

## Datos que cambian (`js/config.js` → `bitacora`)

| Dato | Para qué |
|---|---|
| `horasAlerta` | Después de cuántas horas fuera un equipo se marca en rojo. |
| `urlSitio` | Dirección del sitio; va en el enlace que se graba en las tarjetas. Si cambia, hay que volver a grabarlas. |
| `enPlano` | Qué equipos se dibujan en el plano (por Tipo, o por cómo empieza la Categoría: hoy "Simulador…"). |
| `firebase` | Datos del proyecto de Firebase. Vacío = modo demostración. |
| `microsoft` | Id. de la app registrada en Microsoft Entra. Vacío = no se conecta al Excel. |
| `excel` | Enlace, hoja y encabezados de cada Excel (inventario, bitácora y verificación), opciones de verificación y formato de fecha. |

## Archivos

```
index.html                        La página
vacio.html                        Página en blanco para la ventana de inicio de sesión de Microsoft
css/styles.css                    Estilos base (los mismos del sitio de solicitudes)
css/simuladores.css               Estilos de la bitácora
js/config.js                      Todo lo que cambia (un solo lugar)
js/simuladores-logica.js          Funciones puras: tarjetas, catálogo, validaciones, historial, CSV
js/simuladores-excel-logica.js    Funciones puras: encabezados, filas por ID, celdas que cambian en cada hoja
js/simuladores-excel.js           Microsoft Graph: leer el inventario y escribir en los Excel
js/simuladores-datos.js           Firebase o modo demostración (mismas funciones)
js/simuladores-nfc.js             Web NFC (Android) y lector USB
js/simuladores-ui.js              Todo lo que toca la pantalla
js/vendor/firebase/               Copia local de Firebase 12.19.0 (ver LEEME.md)
js/vendor/msal/                   Copia local de MSAL Browser 4.30.0 (inicio de sesión Microsoft)
firestore.rules                   Reglas de seguridad para pegar en Firebase
tests/                            Pruebas (node --test)
```

Para probarla en la computadora: `python3 -m http.server` en la raíz y abrir
`http://localhost:8000/` (modo demostración; "Probar con un inventario de ejemplo").

## Pruebas automáticas

Sin instalar nada (solo Node.js):

```
node --test
```

- `tests/simuladores.test.js`: tarjetas, catálogo, validaciones, historial y CSV.
- `tests/simuladores-excel.test.js`: lectura de hojas y celdas que se escriben, con hojas de la misma forma que las del CSC
  (datos inventados).

## Publicar en GitHub Pages

1. Los cambios deben estar en la rama **`main`**.
2. En GitHub: **Settings → Pages → Build and deployment → Deploy from a branch**, rama `main`, carpeta `/(root)`.
3. Esperar uno o dos minutos. La dirección es <https://rebecaleu.github.io/bitacora-simuladores/>.

El repositorio es **público** (GitHub Pages gratis lo requiere): cualquiera puede leer el código y `js/config.js`.
Por eso los vínculos de los Excel deben compartirse con **"Personas específicas"** o **"Personas de Tecnológico de Monterrey"**,
nunca con "Cualquier persona". Los datos del CSC no están en el repositorio: viven en Firebase y en los Excel.
