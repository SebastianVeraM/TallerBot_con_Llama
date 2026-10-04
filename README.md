# TallerBot — agente de WhatsApp para un taller mecánico

TallerBot es un prototipo educativo del Hackathon 2. Atiende preguntas frecuentes por WhatsApp, consulta órdenes ficticias y registra solicitudes de cita pendientes de confirmación humana. Se ejecuta en Google Colab para no depender de los recursos de la computadora local.

## Características

- WhatsApp Cloud API para recibir mensajes y enviar respuestas.
- Ollama con Llama 3.1 8B como modelo conversacional.
- Function calling con tres herramientas controladas: FAQ, consulta de órdenes demo y solicitud de cita.
- Historial de conversación en memoria por número de teléfono y exclusión mutua por usuario.
- FastAPI como webhook y ngrok para exponerlo por HTTPS durante la sesión de Colab.
- Diagnóstico del Phone Number ID y de la suscripción de la app a la WABA.
- Registro en pantalla de eventos recibidos, respuestas aceptadas, errores y estados de entrega de Meta.

## Contenido del repositorio

```text
.
├── README.md
└── TallerBot_Colab.ipynb
```

## Requisitos

- Una cuenta de Google con acceso a Google Colab. Se recomienda habilitar una GPU para cargar Llama 3.1 8B.
- Una app configurada en [Meta for Developers](https://developers.facebook.com/) con WhatsApp Cloud API.
- Un número de WhatsApp de prueba o un número configurado para producción.
- Una cuenta y token de ngrok.
- Un teléfono destinatario autorizado en Meta cuando se usa el número de prueba.

## Configuración en Meta

1. Configura el producto WhatsApp en tu app de Meta y ubica el **Phone Number ID** y el **WhatsApp Business Account ID (WABA ID)**.
2. Configura el webhook de la app para el campo `messages`.
3. La primera vez, deja que el cuaderno genere el túnel ngrok. En Meta, registra la URL que imprime el cuaderno, terminada en `/webhook`, junto con el mismo token de verificación.
4. Comprueba que la app esté suscrita a la WABA. La celda de diagnóstico consulta las apps suscritas. Si necesitas suscribir la app actual, verifica primero que el token pertenezca a esa app y tenga acceso a la WABA; luego cambia `SUSCRIBIR_APP_A_WABA = False` a `True` y ejecuta esa celda una vez.
5. Si envías desde un número de prueba de Meta, agrega y verifica el teléfono destinatario en la lista de destinatarios permitidos de Meta. De lo contrario, el envío puede fallar con el error `131030`.

## Secrets de Google Colab

Abre el panel **Secrets** (ícono de llave) y crea estas variables. Habilita el acceso al notebook para cada una.

| Secret | Uso |
| --- | --- |
| `WHATSAPP_ACCESS_TOKEN` | Token de acceso de la app de Meta que tiene acceso al número y a la WABA. |
| `PHONE_NUMBER_ID` | ID del número de WhatsApp conectado a Cloud API. |
| `NGROK_AUTHTOKEN` | Token de autenticación de ngrok. |
| `WHATSAPP_VERIFY_TOKEN` | Token elegido para verificar el webhook; debe coincidir con Meta. |
| `WHATSAPP_BUSINESS_ACCOUNT_ID` | ID de la WABA, necesario para revisar o solicitar la suscripción de la app. |
| `META_APP_ID` | ID de la app que debe aparecer suscrita a la WABA. |
| `WHATSAPP_TEST_RECIPIENT` | Número de prueba permitido en Meta; usado para filtrar los mensajes entrantes de prueba y para el envío saliente opcional. |

Los últimos tres Secrets se usan en diagnósticos o pruebas opcionales. El cuaderno no imprime los tokens de acceso. **No pegues credenciales en el código ni las subas a GitHub.** Si una credencial quedó expuesta, revócala desde Meta o ngrok y genera otra.

## Ejecutar en Google Colab

1. Sube `TallerBot_Colab.ipynb` a Colab o ábrelo desde el repositorio de GitHub.
2. Configura los Secrets indicados arriba.
3. Selecciona un entorno de ejecución con GPU si está disponible.
4. Ejecuta las celdas en orden. El cuaderno instala Ollama y dependencias, descarga `llama3.1:8b`, muestra diagnósticos de Meta y levanta FastAPI con ngrok.
5. Copia la URL HTTPS impresa por ngrok y configura en Meta el callback terminado en `/webhook`. El token de verificación debe coincidir y el campo `messages` debe estar suscrito.
6. Usa primero la prueba local `/demo`, que no envía mensajes reales. Luego, con el webhook activo, escribe desde un teléfono autorizado al número de WhatsApp de prueba.
7. Ejecuta la celda de registros para revisar eventos, respuestas y estados de entrega. La prueba saliente está desactivada de forma predeterminada; habilítala solo si quieres enviar un mensaje real y el destinatario está autorizado.
8. Ejecuta la celda de cierre de ngrok al terminar.

Cada sesión nueva de Colab requiere volver a ejecutar el notebook y registrar en Meta la nueva URL de ngrok si esta cambió. Mantén el runtime y el túnel activos mientras quieras que el bot responda.

## Ejemplos para la demostración

- «¿Cuál es el horario?»
- «¿Cómo va la orden ORD-1001?»
- «Quiero una cita para cambio de aceite el viernes por la mañana»
- «Quiero hablar con una persona»

Las solicitudes de cita quedan como `pendiente` y requieren confirmación humana.

## Alcance y limitaciones

- Horarios, dirección, órdenes y citas son datos ficticios para la demostración.
- El historial, las órdenes y las solicitudes viven en memoria y se pierden al reiniciar Colab.
- No hay conexión a inventario, agenda, pagos ni sistema real de órdenes.
- El túnel ngrok de esta configuración sirve para pruebas durante una sesión; no es un despliegue permanente.
- No uses este prototipo para datos reales de clientes ni lo presentes como un servicio de producción.

## Publicar en GitHub

Crea un repositorio vacío en GitHub y, desde esta carpeta, ejecuta estos comandos en PowerShell. Sustituye la URL por la del repositorio que creaste:

```powershell
git init
git add README.md TallerBot_Colab.ipynb
git commit -m "Documenta el prototipo TallerBot"
git branch -M main
git remote add origin https://github.com/USUARIO/REPOSITORIO.git
git push -u origin main
```

Antes de publicar, confirma que el notebook no contenga tokens ni credenciales en sus celdas de salida. Los Secrets se configuran dentro de Colab y no forman parte del repositorio.
