# 🤖 Asistente Virtual para Tienda (Telegram + AI Agent en n8n)

Este repositorio (o flujo de n8n) contiene un asistente virtual inteligente conectado a Telegram, diseñado para gestionar de forma automatizada la atención a Clientes, consultas de Empleados y administración de inventario por parte del Administrador.

El flujo utiliza el nodo **AI Agent** (LangChain) de n8n impulsado por OpenAI, el cual es capaz de tomar decisiones, leer contextos y ejecutar herramientas de Google Sheets y Google Drive de manera dinámica según el perfil del usuario.

---

## ✨ Características Principales

El comportamiento del bot cambia dinámicamente gracias a un proceso de Identificación Inicial (Nombre y Código Personal). Dependiendo del perfil, el bot habilita diferentes funciones:

### 👤 1. Perfil Cliente (Usuarios sin código o no registrados)
* **Compras:** Pueden visualizar el catálogo de productos (Solo columnas públicas: índice, descripción, marca, categoría y precio), armar un "carrito" de compras, calcular su total y programar una hora de recolección en tienda. El pedido se guarda automáticamente.
* **Comentarios:** Sistema para dejar retroalimentación (🔴 Malo, 🟡 Bueno, 🟢 Excelente) junto con comentarios de texto.
* **Soporte:** Consulta de políticas de reembolso, devoluciones y preguntas frecuentes.

### 🧑‍💼 2. Perfil Empleado (Registrados en `Lista_de_empleados`)
* **Consulta de Inventario:** Acceso de solo lectura a la base de datos de inventario (`Stok`).
* **Documentación Interna:** Acceso de lectura a documentos almacenados en Google Drive:
  * `PREGUNTAS_FRECUENTES` (txt)
  * `Reglamento_Interno_de_Trabajo` (txt)
  * `Terminos_y_Condiciones_Reembolso` (pdf/txt)

### 👑 3. Perfil Administrador (Exclusivo: "Jocelyn" - Código "3110")
Tiene los permisos del Empleado, más la capacidad de gestionar bases de datos restringidas:

* **Gestión de Inventario (`Stok`):**
  * **Agregar a inventario:** Suma piezas al `stock_actual` de un producto existente.
  * **Descontar del inventario:** Resta piezas al `stock_actual` tras una venta o merma.
  * **Agregar producto nuevo:** Genera un nuevo índice y registra todos los datos de un nuevo artículo.
* **Control de Personal y Ventas:** Capacidad de visualizar la tabla de `PEDIDOS` y consultar la `Lista_de_empleados`.

---

## 🛠️ Arquitectura del Flujo (Nodos n8n)

El flujo está construido con los siguientes componentes principales:

* **Telegram Trigger:** Escucha cada mensaje entrante de los usuarios.
* **AI Agent (`@n8n/n8n-nodes-langchain.agent`):** El "cerebro" del bot. Orquesta la conversación utilizando un modelo LLM (`gpt-5-mini` / equivalente en OpenAI).
* **Window Buffer Memory:** Mantiene el contexto de los últimos 20 mensajes de la conversación por chat (basado en el `chat.id` de Telegram).
* **Tools (Herramientas del Agente):**
  * `consultar_lista_empleados` (Google Sheets)
  * `consultar_stock` (Google Sheets)
  * `actualizar_stock_actual` (Google Sheets - Update)
  * `agregar_producto_nuevo` (Google Sheets - Append)
  * `registrar_pedido` (Google Sheets - Append)
  * `registrar_comentario` (Google Sheets - Append)
  * `leer_preguntas_frecuentes` (Google Drive - Download)
  * `leer_reglamento_interno` (Google Drive - Download)
  * `leer_terminos_reembolso` (Google Drive - Download)
* **Telegram Send Reply:** Envía la respuesta procesada por la IA de vuelta al usuario.

---

## 📋 Requisitos Previos (Prerequisites)

Para ejecutar este flujo en tu instancia de n8n, necesitas las siguientes credenciales configuradas:

* **Telegram API:** Un bot creado en BotFather y su Token de acceso.
* **OpenAI API:** Una clave de API (API Key) con saldo disponible para usar los modelos de chat.
* **Google Drive OAuth2 / Service Account:** Permisos para leer los archivos de texto y PDF.
* **Google Sheets OAuth2 / Service Account:** Permisos de lectura/escritura para interactuar con los libros de cálculo.

---

## 📁 Estructura de Datos Requerida (Google Workspace)

Para que las herramientas del agente funcionen correctamente, los documentos deben estar estructurados de la siguiente manera:

### Google Sheets

* **Documento 1: Empleados**
  * **Hoja:** `Lista_de_empleados`
  * **Columnas:** `nombre`, `codigo`
* **Documento 2: Inventario**
  * **Hoja:** `Stok`
  * **Columnas:** `indice`, `descripcion`, `categoria`, `marca`, `stock_inicial`, `stock_actual`, `precio_unitario`, `precio_al_publico`, `ganancia`
* **Documento 3: Registros Tienda**
  * **Hoja 1:** `PEDIDOS` (Columnas: `nombre_cliente`, `hora_entrega`, `carrito`, `total_a_pagar`)
  * **Hoja 2:** `COMENTARIOS` (Columnas: `fecha`, `calificacion`, `comentario`)

### Google Drive (Archivos)

Deben existir y estar accesibles para la credencial de Google Drive los siguientes archivos por sus respectivos IDs:

* `faq.txt` (`PREGUNTAS_FRECUENTES`)
* `reglamentointerno.txt` (`Reglamento_Interno_de_Trabajo`)
* `Terminos_y_Condiciones_Reembolso_Abarrotera (1).pdf`

---

## 🚀 Instalación y Despliegue

1. Copia el código JSON del flujo proporcionado.
2. En tu panel de n8n, ve a **Workflows** y selecciona **Import from File** o pega directamente el JSON en el lienzo (Canvas).
3. Abre cada nodo que requiera credenciales (Telegram Trigger, OpenAI Model, Google Drive, Google Sheets) y selecciona tus cuentas preconfiguradas.
4. Activa el flujo (Toggle superior derecho a **Active**).
5. ¡Inicia un chat con tu bot en Telegram! Inicia enviando un simple "Hola".
