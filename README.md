# Elyora · Landing page + Sophie (chatbot de ventas con n8n)

Landing page de **Elyora · Joyas y accesorios** lista para GitHub Pages, con **Sophie**, la asesora virtual que recomienda productos, suma el pedido, lo registra en Google Sheets y genera el resumen para el WhatsApp **933 504 129**. El pago **nunca** se hace en el chat.

```
elyora-landing/
├── index.html                      ← la web completa (sube esto a GitHub)
├── assets/img/brand/               ← logo (fondo transparente), isotipo, favicon, fotos de portada
├── assets/img/products/            ← fotos de los 41 productos (de tus catálogos Canva)
├── n8n/elyora-sophie-workflow.json ← flujo para importar en n8n
├── data/productos.json             ← catálogo (fuente única de productos y precios)
├── data/plantilla-hoja-pedidos.csv ← encabezados de la hoja de Google
└── tools/                          ← plantilla + script para regenerar si cambias precios
```

## 1. Configurar n8n

1. En n8n: **Workflows → Import from file** → `n8n/elyora-sophie-workflow.json`.
2. Nodo **Claude (modelo)** → crea una credencial *Anthropic* con tu API key. Viene con `claude-sonnet-5-5`; si quieres reducir costos, cámbialo por `claude-haiku-5-5`. También puedes reemplazarlo por un nodo *OpenAI Chat Model* conectado al mismo agente.
3. **Google Sheets**
   - Crea una hoja en tu Drive con una pestaña llamada **`Pedidos`**.
   - Pega en la fila 1 los encabezados de `data/plantilla-hoja-pedidos.csv`: `Fecha · ID Pedido · Cliente · Celular · Entrega · Distrito / Ciudad · Dirección / Agencia · Productos · N° piezas · Subtotal (S/) · Envío · Total (S/) · Notas · Estado · Sesión`.
   - En el nodo **Registrar pedido en Google Sheets**, conecta tu cuenta de Google (OAuth2) y pega la URL de la hoja en *Document*.
4. **Activa** el flujo (switch *Active*) y copia la **Production URL** del nodo **Webhook Sophie** (termina en `/webhook/elyora-sophie`).

### ¿Cómo llega el pedido al WhatsApp 933 504 129?
- **Por defecto (sin costo):** cuando la clienta confirma, Sophie le muestra el botón verde **«Enviar resumen por WhatsApp»**, que abre WhatsApp con el pedido completo ya escrito, dirigido al 51 933 504 129. Así la clienta inicia la conversación para coordinar el pago.
- **Opcional (aviso automático):** el nodo **Notificar a Elyora (WhatsApp API)** viene desactivado. Actívalo si tienes *WhatsApp Business Cloud API* (Meta): agrega la credencial y el *Phone Number ID*. Meta solo permite mensajes de texto libre si tu número escribió al número de la API en las últimas 24 h; si no, debes usar una plantilla aprobada.

## 2. Conectar la web con n8n

En `index.html`, arriba del todo, edita:

```js
window.ELYORA_CONFIG = {
  n8nWebhookUrl: "https://TU-INSTANCIA-N8N.com/webhook/elyora-sophie",  // ← tu Production URL
  whatsapp: "51933504129",
  ...
};
```

Si la URL no está configurada o n8n no responde, Sophie avisa y ofrece el botón de WhatsApp, y la bolsa siempre permite enviar el pedido directo por WhatsApp.

**CORS:** el webhook acepta cualquier origen (`*`). Cuando publiques, puedes restringirlo en *Webhook Sophie → Options → Allowed Origins* a `https://TU-USUARIO.github.io`.

## 3. Publicar en GitHub Pages

1. Crea un repositorio (por ejemplo `elyora`) y sube **todo el contenido** de esta carpeta (no solo el `index.html`, porque las fotos están en `assets/`).
2. En el repositorio: **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `(root)`**.
3. En 1–2 minutos tu web estará en `https://TU-USUARIO.github.io/elyora/`.

## 4. Cómo trabaja Sophie

- Solo responde sobre Elyora (productos, precios, materiales, envíos, compra). Cualquier otro tema lo rechaza con amabilidad.
- Solo conoce los 41 productos de tus catálogos **Noviembre** y **Diciembre**; no inventa productos, tallas, stock, tarifas de envío ni medios de pago.
- Cuando menciona un producto escribe su código (`[AR06]`) y la web muestra automáticamente su **foto, precio y botón «Agregar»**.
- La sumatoria la hace la herramienta `calcular_pedido` (código, no la IA), con los precios de oferta y las reglas de envío gratis: Lima > S/ 100 en los 14 distritos del catálogo; provincia > S/ 200 (sin Iquitos).
- Al confirmar, el nodo **Procesar respuesta** vuelve a validar los productos y los precios, crea el ID (`ELY-AAMMDD-XXXX`), escribe la fila en Sheets (Estado: *Pendiente de pago*) y arma el mensaje de WhatsApp.

## 5. Cambiar productos o precios

1. Edita `data/productos.json` (para fotos nuevas, agrégalas en `assets/img/products/` como `codigo-1.webp` y `codigo-modelo.webp`).
2. Ejecuta `python tools/build.py` (necesitas Python 3). Esto regenera `index.html` **y** `n8n/elyora-sophie-workflow.json`.
3. Vuelve a importar el flujo en n8n (o copia el *system message* y los nodos de código) y sube los cambios a GitHub. Recuerda volver a poner tu `n8nWebhookUrl` en `index.html`.

## Notas
- Reseñas: la web **no** incluye reseñas inventadas. Hay un bloque comentado en `index.html` (sección `#resenas`) para pegar reseñas reales de tus clientas, con su permiso.
- Algunas piezas plateadas del catálogo de Noviembre (Bone Snake Plateada, Soft Love, Pop Cupido y Clover plateados, Tree of Life Plateada) dicen «Acero bañado/enchapado en oro de 18 k», tal como aparece en tu Canva. Corrígelo en `data/productos.json` si el material es otro.
