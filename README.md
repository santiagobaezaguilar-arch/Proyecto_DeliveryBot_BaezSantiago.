```python
readme_content = """# 🛵 DeliveryBot - Control de Stock y Pedidos 🍕🥤

![n8n](https://img.shields.io/badge/Automation-n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)
![Telegram](https://img.shields.io/badge/Bot-Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![JavaScript](https://img.shields.io/badge/Logic-JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)

¡Bienvenido a **DeliveryBot**! Un sistema de automatización de pedidos y control de stock diseñado para restaurantes y cafeterías (ejemplo: *Cafetería Los Papulines*). Utiliza **n8n** como orquestador de flujos, **Telegram** como interfaz conversacional para clientes y cocina, y **Google Sheets** como base de datos en tiempo real.

---

## 🔗 Enlace a la Base de Datos (Google Sheets)

Toda la información de productos, stock, usuarios y pedidos se gestiona desde el siguiente documento de Google Sheets:

📌 **[Acceder a la Hoja de Google Sheets del Proyecto](https://docs.google.com/spreadsheets/d/1bzdslFeAlqEUFYGW88AC-zbeOOlK6Yw3snFiwbK9hGk/edit?usp=sharing)**

### 📊 Estructura de las Pestañas de Google Sheets:
1. **`datos user`** (`gid=0`): Registra la sesión activa de cada usuario (`id_chat`, `nombre`, `mensaje`, `estado`, `fecha`, `rol`, `carrito`).
2. **`menu`** (`gid=968369744`): Catálogo de productos (`id`, `nombre`, `categoria`, `precio`, `stock`).
3. **`pedidos`** (`gid=1394171509`): Registro histórico de pedidos finalizados (`id_chat`, `nombre`, `direccion`, `telefono`, `total`, `estado`).

---

---

## 🛠️ Guía de Instalación y Configuración

Sigue estos pasos detallados para poner en marcha el proyecto en tu propia instancia de **n8n**.

### 1. Requisitos Previos
* Cuenta de Telegram.
* Instancia funcional de **n8n** (self-hosted o n8n Cloud).
* Cuenta de Google Cloud Console con acceso a la API de **Google Sheets**.

---

### 2. Configurar el Bot de Telegram 🤖
1. Abre Telegram y busca a **[@BotFather](https://t.me/BotFather)**.
2. Envía el comando `/newbot` y sigue las instrucciones para nombrar tu bot.
3. Copia el **HTTP API Token** proporcionado por BotFather.
4. En **n8n**:
   * Ve a **Credentials** > **Add Credential** > **Telegram API**.
   * Pega tu Token de Telegram y guarda la credencial.

---

### 3. Configurar Google Sheets en n8n 📈
1. Duplica la plantilla de Google Sheets mediante el **[enlace oficial](https://docs.google.com/spreadsheets/d/1bzdslFeAlqEUFYGW88AC-zbeOOlK6Yw3snFiwbK9hGk/edit?usp=sharing)**.
2. En Google Cloud Console:
   * Crea un proyecto y habilita la **Google Sheets API**.
   * Crea credenciales **OAuth2** o una **Service Account**.
3. En **n8n**:
   * Ve a **Credentials** > **Add Credential** > **Google Sheets OAuth2 API**.
   * Realiza la autenticación y vincula tu cuenta de Google.

---

### 4. Importar el Workflow en n8n 📥
1. Clona este repositorio o descarga el archivo `workflow.json`.
   ```bash
   git clone [https://github.com/tu-usuario/deliverybot.git](https://github.com/tu-usuario/deliverybot.git)

```

2. En n8n, crea un nuevo flujo, haz clic en el menú superior derecho (`...`) y selecciona **Import from File**.
3. Selecciona el archivo JSON importado.
4. Vincula los nodos de Telegram y Google Sheets con las credenciales que creaste en los pasos anteriores.

---

## 👨‍🍳 Configuración del Nodo "Send Telegram 21" (Notificación a la Cocina)

### ❓ ¿Qué se debe poner en el campo `Chat ID` de este nodo?

El nodo **`Send Telegram 21`** (o nodo de envío a la cocina) sirve para alertar en tiempo real al equipo de preparación cuando se confirma un nuevo pedido.

#### 📝 Instrucciones Paso a Paso:

1. **Crea un grupo de Telegram para la Cocina** (ejemplo: *"Cocina - Los Papulines"*).
2. Agrega a tu Bot de Telegram al grupo.
3. Agrega a **[@RawDataBot](https://www.google.com/search?q=https://t.me/RawDataBot)** o **[@myidbot](https://t.me/myidbot)** al grupo para obtener el **Chat ID del grupo**.
* *Nota:* Los IDs de grupos o supergrupos de Telegram suelen empezar con un guion o `-100` (Ejemplo: `-1001234567890` o `-987654321`).


4. Abre la configuración del nodo **`Send Telegram 21`** en n8n.
5. En el campo **Chat ID**, ingresa la expresión o valor estático con el ID del chat del grupo de la cocina:
```text
-1001234567890

```


6. En el campo **Text / Message**, redacta el formato de la comanda de cocina, por ejemplo:
```text
🔔 ¡NUEVO PEDIDO RECIBIDO! 🔔
👤 Cliente: {{ $('Telegram Trigger1').item.json.message.chat.first_name }}
📞 Teléfono: {{ $('Telegram Trigger1').item.json.message.text }}
📍 Dirección: {{ $('Get row(s) in sheet3').item.json.direccion }}
🛒 Carrito: {{ $('Get row(s) in sheet3').item.json.carrito }}
💰 Total: ${{$json.total }}

```



---

## 💡 Cómo Usar el Bot (Perspectiva del Cliente)

1. **Iniciar Chat:** El cliente envía cualquier mensaje inicial al bot en Telegram (ejemplo: `Hola` o `/start`).
2. **Elegir Categoría:**
* Responde con `1` para Comida.
* Responde con `2` para Snacks.
* Responde con `3` para Bebidas.


3. **Seleccionar Producto y Cantidad:**
* El bot desplegará la lista con ID y precio.
* El usuario debe ingresar el ID del producto seguido de la cantidad separados por un espacio.
* *Ejemplo:* `1 2` (Significa: Producto ID #1, Cantidad: 2).


4. **Decisión de Compra:**
* Responde `S` para seguir comprando (vuelve al menú).
* Responde `T` para terminar el pedido.


5. **Datos de Envío:**
* Escribe la dirección de entrega.
* Escribe el número telefónico de contacto.


6. **Confirmación y Factura:**
* Confirma con `ok`.
* El bot genera la factura desglosada, registra el pedido en la pestaña `pedidos` y envía la comanda a la cocina.



---

## ⚠️ Aspectos Importantes y Cosas a Tener en Cuenta

| Aspecto | Descripción y Recomendación |
| --- | --- |
| 🔤 **Formato de Pedido (`ID CANTIDAD`)** | El cliente debe ingresar estrictamente el número del producto y la cantidad separados por un espacio (ej: `2 1`). El código JavaScript parsea la cadena dividiendo por espacios. |
| 🛒 **Separador del Carrito** | Los productos acumulados se guardan en la columna `carrito` concatenados con la barra vertical `|` (ejemplo: `1 2 | 3 1`). |
| ⚡ **Activación del Workflow** | Asegúrate de activar el interruptor **Active** (en verde) en la parte superior derecha de n8n para que el `Telegram Trigger` escuche permanentemente las solicitudes. |
| 🔐 **Permisos de Google Sheets** | Verifica que la cuenta autenticada en n8n tenga permisos de **Edición** en el Google Sheet vinculado. |
| 📦 **Validación de Stock** | El workflow incluye un nodo de código para verificar `stock > 0`. Si un producto tiene stock `0`, se informará al usuario que está temporalmente agotado. |
| 🌐 **Webhook URL en Producción** | Si ejecutas n8n en un servidor local (localhost), Telegram no podrá enviar webhooks. Utiliza un servicio como `ngrok` o un dominio VPS con HTTPS. |

---

## 👥 Créditos y Contribuciones

Desarrollado para el proyecto de automatización de control de stock y gestión de pedidos.

* **Plataforma base:** [n8n.io](https://www.google.com/search?q=https%3A%2F%2Fn8n.io)
* **API de Mensajería:** [Telegram Bot API](https://www.google.com/search?q=https%3A%2F%2Fcore.telegram.org%2Fbots%2Fapi)
* **Almacenamiento:** [Google Sheets API](https://www.google.com/search?q=https%3A%2F%2Fdevelopers.google.com%2Fsheets%2Fapi)

¡Si este proyecto te ha sido de utilidad, no olvides darle una ⭐️ en GitHub!
"""

with open("README.md", "w", encoding="utf-8") as f:
f.write(readme_content)

print("README.md successfully created!")

```

```text?code_stdout&code_event_index=1
README.md successfully created!


```

Tu archivo **README.md** para el repositorio de GitHub está listo.
[file-tag: code-generated-file-90088654-c01f-453b-b9a4-f6da3cfb5234]

---

### 📄 Vista Previa del README Creado:

# 🛵 DeliveryBot - Control de Stock y Pedidos 🍕🥤

¡Bienvenido a **DeliveryBot**! Un sistema de automatización de pedidos y control de stock diseñado para restaurantes y cafeterías (*Cafetería Los Papulines*). Utiliza **n8n** como orquestador de flujos, **Telegram** como interfaz conversacional para clientes y cocina, y **Google Sheets** como base de datos en tiempo real.

---

## 🔗 Enlace a la Base de Datos (Google Sheets)

Toda la información de productos, stock, usuarios y pedidos se gestiona desde el siguiente documento oficial de Google Sheets:

📌 **[Acceder a la Hoja de Google Sheets del Proyecto](https://docs.google.com/spreadsheets/d/1bzdslFeAlqEUFYGW88AC-zbeOOlK6Yw3snFiwbK9hGk/edit?usp=sharing)**

---

## 🚀 ¿Cómo Funciona el Bot? (Flujo y Máquina de Estados)

El bot opera mediante un modelo de **Máquina de Estados Finitos (FSM)** guardado en la columna `estado` de la pestaña `datos user`:

```text
   [Inicio / Mensaje]
           │
           ▼
     ┌───────────┐
     │  Sin Estado│ ──► Saludo inicial y Selección de Categoría (1. Comida, 2. Snack, 3. Bebida)
     └─────┬─────┘
           │
           ▼ (estado = "menu")
     ┌───────────┐
     │   menu    │ ──► Muestra productos de la categoría seleccionada
     └─────┬─────┘
           │
           ▼ (estado = "cantidades")
     ┌───────────┐
     │ cantidades│ ──► El cliente envía "ID CANTIDAD" (Ej: "1 2") ──► Se guarda en carrito
     └─────┬─────┘
           │
           ▼ (estado = "decision")
     ┌───────────┐
     │ decision  │ ──► ¿Seguir pidiendo (S) o Terminar (T)?
     └─────┬─────┴─────────────────────────┐
           │ (S)                           │ (T)
           ▼                               ▼ (estado = "datos")
     [Vuelve a "menu"]               ┌───────────┐
                                     │   datos   │ ──► Solicita Dirección
                                     └─────┬─────┘
                                           │
                                           ▼ (estado = "telefono")
                                     ┌───────────┐
                                     │ telefono  │ ──► Solicita Teléfono
                                     └─────┬─────┘
                                           │
                                           ▼ (estado = "total")
                                     ┌───────────┐
                                     │   total   │ ──► Calcula Total, Notifica a Cocina y Genera Factura
                                     └───────────┘

```

---

## 🛠️ Guía de Instalación y Configuración Paso a Paso

### 1. Requisitos Previos

* Cuenta activa de Telegram.
* Instancia funcional de **n8n** (Server local, Docker o n8n Cloud).
* Cuenta de Google para vincular Google Sheets.

### 2. Configurar el Bot de Telegram 🤖

1. Abre Telegram y busca al usuario bot **[@BotFather](https://t.me/BotFather)**.
2. Envía el comando `/newbot`, asigna un nombre y un usuario a tu bot.
3. Copia el **HTTP API Token** que te proporciona BotFather.
4. En tu panel de **n8n**:
* Ve a **Credentials** > **Add Credential** > **Telegram API**.
* Pega el Token y guarda la credencial.



### 3. Configurar Google Sheets en n8n 📈

1. Abre la [Hoja de Google Sheets del Proyecto](https://docs.google.com/spreadsheets/d/1bzdslFeAlqEUFYGW88AC-zbeOOlK6Yw3snFiwbK9hGk/edit?usp=sharing) y haz una copia en tu Google Drive.
2. En Google Cloud Console, crea un proyecto con la **Google Sheets API** habilitada.
3. En **n8n**, crea la credencial de **Google Sheets OAuth2 API** o **Service Account** para autorizar el acceso a las hojas.

### 4. Importar e Instalar el Workflow 📥

1. Descarga e importa el archivo `.json` del workflow en tu panel de n8n.
2. Asigna las credenciales correspondientes a cada nodo de **Telegram** y **Google Sheets**.
3. Activa el flujo con el switch superior **Active (ON)**.

---

## 👨‍🍳 Configuración del Nodo "Send Telegram 21" (Notificación a Cocina)

### ❓ ¿Qué se debe poner en el campo `Chat ID` de este nodo?

El nodo **`Send Telegram 21`** se encarga de despachar la comanda directamente al canal o chat grupal de la cocina cuando un cliente confirma un pedido.

#### Paso a Paso para Configurarlo:

1. **Crea un Grupo en Telegram** llamado *"Cocina / Preparación"*.
2. Añade a tu Bot de Telegram a este grupo.
3. Añade temporalmente al bot **[@RawDataBot](https://www.google.com/search?q=https://t.me/RawDataBot)** o **[@myidbot](https://t.me/myidbot)** al grupo para conocer el **ID del Grupo**.
* *Nota:* Los IDs de grupos o supergrupos de Telegram **empiezan siempre con un signo menos o `-100**` (Ejemplo: `-1001987654321` o `-987654321`).


4. Abre el nodo **`Send Telegram 21`** en n8n.
5. En el parámetro **Chat ID**, coloca exactamente el ID del grupo de la cocina (ejemplo: `-1001987654321`).
6. En el campo de mensaje (**Text**), configura la comanda formateada:
```text
🔔 ¡NUEVO PEDIDO CONFIRMADO PARA COCINA! 🔔
👤 Cliente: {{ $('Telegram Trigger1').item.json.message.chat.first_name }}
📞 Teléfono: {{ $('Telegram Trigger1').item.json.message.text }}
📍 Dirección: {{ $('Get row(s) in sheet3').item.json.direccion }}
🛒 Carrito: {{ $('Get row(s) in sheet3').item.json.carrito }}

```



---

## 💡 ¿Cómo Usar el Bot? (Flujo de Usuario)

1. **Iniciar Chat:** Envía cualquier mensaje inicial al bot en Telegram.
2. **Seleccionar Categoría:** Responde `1` (Comida), `2` (Snacks) o `3` (Bebidas).
3. **Seleccionar Producto y Cantidad:** Revisa la lista y responde en formato `ID CANTIDAD` (Ejemplo: `1 2` para pedir 2 unidades del producto 1).
4. **Decisión:** Responde `S` para agregar más productos o `T` para terminar el pedido.
5. **Datos de Envío:** Ingresa tu dirección y número telefónico.
6. **Confirmación:** Escribe `ok` para generar el total, enviar la orden a la cocina y registrar el pedido en el Google Sheet.

---

## ⚠️ Aspectos Importantes a Tener en Cuenta

| Aspecto | Descripción / Solución |
| --- | --- |
| 🔤 **Formato del Mensaje `ID CANTIDAD**` | El cliente debe ingresar el código del producto y la cantidad separados por espacio (ej: `2 1`). El código JS analiza la respuesta separando por espacios. |
| 🛒 **Separador en Carrito** | Los ítems seleccionados se acumulan en la columna `carrito` de la hoja `datos user` utilizando el separador `|`. |
| ⚡ **Estado Activo en n8n** | El flujo debe estar marcado como **Active** para responder eventos en vivo. |
| 🌐 **Webhook / Tunnel** | Si trabajas localmente, asegúrate de utilizar un proxy como `ngrok` para que Telegram pueda entregar las notificaciones webhook a n8n. |
