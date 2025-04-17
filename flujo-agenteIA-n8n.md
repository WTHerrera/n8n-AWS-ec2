Flujo de Trabajo de Ejemplo en n8n para Agentes IA
Este flujo de trabajo en n8n muestra cómo un agente de IA puede procesar una solicitud de usuario, generar una respuesta usando una API de IA (OpenAI GPT), y enviar el resultado por correo electrónico. Es un ejemplo simple pero extensible para tareas como chatbots, análisis de texto, o automatización de respuestas.
Descripción del Flujo

Objetivo: Recibir una pregunta o texto a través de un webhook, procesarlo con OpenAI GPT para generar una respuesta, y enviar la respuesta por correo electrónico.
Nodos Utilizados:
Webhook: Recibe la entrada del usuario (e.g., una pregunta).
OpenAI: Envía el texto a la API de OpenAI para generar una respuesta.
Send Email: Envía la respuesta generada por correo.


Caso de Uso: Automatizar respuestas a preguntas frecuentes, generar contenido dinámico, o analizar texto con IA.

Prerrequisitos

Una instancia de n8n configurada (puede ser la de AWS EC2 del tutorial anterior).
Una API Key de OpenAI (obténla en platform.openai.com).
Una cuenta de correo con acceso SMTP (e.g., Gmail) para enviar emails.

Paso a Paso: Configuración del Flujo
Paso 1: Crear el Flujo

Inicia sesión en tu instancia de n8n (e.g., https://<TU_IP_O_DOMINIO>).
Haz clic en New Workflow para crear un flujo nuevo.

Paso 2: Configurar el Nodo Webhook

Añade un nodo Webhook (búscalo en el panel de nodos).
Configura:
HTTP Method: POST.
Path: ai-agent (o personalízalo).
Response Code: 200.


Copia la URL del webhook (e.g., https://<TU_DOMINIO>/webhook/ai-agent) para pruebas.
Activa el flujo (Activate) para que el webhook esté activo.

Paso 3: Configurar el Nodo OpenAI

Añade un nodo OpenAI y conéctalo al Webhook.
Configura:
Credentials: Añade tu API Key de OpenAI.
Resource: Chat.
Model: gpt-4o-mini (o el modelo disponible en tu plan).
Prompt: Usa una expresión para pasar el texto del webhook:{{ $json.body.question }}

Esto asume que el webhook recibe un JSON con un campo question.
Max Tokens: 150 (ajusta según necesidad).


Opcional: Añade un sistema de prompt inicial para personalizar el tono (e.g., “Actúa como un asistente útil”).

Paso 4: Configurar el Nodo Send Email

Añade un nodo Send Email y conéctalo al nodo OpenAI.
Configura:
Credentials: Configura SMTP (e.g., Gmail: smtp.gmail.com, puerto 465, SSL).
From Email: Tu dirección de correo.
To Email: El destinatario (puedes usar {{ $json.body.email }} si el webhook lo incluye).
Subject: “Respuesta de tu Agente IA”.
Text: Usa la respuesta de OpenAI:{{ $node["OpenAI"].json.choices[0].message.content }}




Prueba enviando un email de prueba.

Paso 5: Probar el Flujo

Usa una herramienta como Postman o curl para enviar un POST al webhook:curl -X POST https://<TU_DOMINIO>/webhook/ai-agent \
-H "Content-Type: application/json" \
-d '{"question": "Explica qué es IA en 50 palabras", "email": "tu@correo.com"}'


Verifica:
La respuesta de OpenAI en el nodo OpenAI.
El correo recibido con la respuesta generada.



Paso 6: Guardar y Escalar

Guarda el flujo (Save).
Escala según necesidades:
Añade nodos para manejar errores (e.g., Error Trigger).
Integra más APIs de IA (e.g., Hugging Face para otros modelos).
Usa nodos como HTTP Request para conectar con otras herramientas.



JSON del Flujo de Trabajo
Copia este JSON y pégalo en n8n (Workflow > Import > From JSON) para importar el flujo:
{
  "nodes": [
    {
      "parameters": {
        "path": "ai-agent",
        "responseCode": 200
      },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [240, 300]
    },
    {
      "parameters": {
        "resource": "chat",
        "model": "gpt-4o-mini",
        "prompt": "{{ $json.body.question }}",
        "maxTokens": 150
      },
      "name": "OpenAI",
      "type": "n8n-nodes-base.openAI",
      "typeVersion": 1,
      "position": [460, 300],
      "credentials": {
        "openAiApi": {
          "apiKey": "TU_API_KEY"
        }
      }
    },
    {
      "parameters": {
        "fromEmail": "tu@correo.com",
        "toEmail": "{{ $json.body.email }}",
        "subject": "Respuesta de tu Agente IA",
        "text": "{{ $node[\"OpenAI\"].json.choices[0].message.content }}"
      },
      "name": "Send Email",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 1,
      "position": [680, 300],
      "credentials": {
        "smtp": {
          "host": "smtp.gmail.com",
          "port": 465,
          "secure": true,
          "user": "tu@correo.com",
          "password": "TU_CONTRASEÑA_O_APP_PASSWORD"
        }
      }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "OpenAI",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI": {
      "main": [
        [
          {
            "node": "Send Email",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}

Notas:

Reemplaza TU_API_KEY, tu@correo.com, y TU_CONTRASEÑA_O_APP_PASSWORD con tus credenciales.
Si usas Gmail, genera una App Password en tu cuenta de Google para SMTP.

Infografía: Resumen del Flujo de Trabajo
┌────────────────────────────────────────────────────────────┐
│       🧠 Flujo de Trabajo n8n para Agentes IA               │
└────────────────────────────────────────────────────────────┘
[Background: Light blue (#E6F0FA), title in white box, font: Montserrat Bold, 20pt]

┌────────────────────┐ ┌────────────────────┐ ┌────────────────────┐
│ 🌐 1. Webhook      │ │ 🤖 2. OpenAI       │ │ 📧 3. Send Email   │
├────────────────────┤ ├────────────────────┤ ├────────────────────┤
│ • Método: POST     │ │ • API Key OpenAI   │ │ • SMTP (e.g., Gmail)│
│ • Path: ai-agent   │ │ • Modelo: gpt-4o-mini │ │ • De: tu@correo.com │
│ • Recibe pregunta  │ │ • Prompt: {{question}}│ │ • Para: {{email}}  │
│                    │ │ • Máx. tokens: 150 │ │ • Texto: {{respuesta}}│
└────────────────────┘ └────────────────────┘ └────────────────────┘
[Three boxes, side-by-side, white background, blue border (#4A90E2), font: Roboto, 12pt]

┌────────────────────────────────────────────────────────────┐
│ 🚀 ¡Automatiza Respuestas IA y Envía por Correo! 🎉        │
└────────────────────────────────────────────────────────────┘
[Footer: Green background (#2ECC71), white text, font: Montserrat Bold, 14pt]

Instrucciones para Crear la Infografía

Herramienta:

Canva: Crea un diseño de 600x400px.
PowerPoint: Usa un lienzo de 6x4 pulgadas.
Illustrator: Configura un lienzo de 600x400px.


Pasos:

Fondo: Azul claro (#E6F0FA).
Título: Caja blanca, texto negro, Montserrat Bold, 20pt.
Cajas de Nodos:
3 cajas (180x150px), borde azul (#4A90E2), fondo blanco.
Texto en Roboto, 12pt, negro (#333333).
Íconos: Busca “webhook”, “robot”, “email” en Canva o usa emojis (🌐, 🤖, 📧).


Pie: Caja verde (#2ECC71), texto blanco, Montserrat Bold, 14pt.
Espaciado: 15px entre cajas, 20px entre secciones.


Exportar:

Canva: Descarga como PNG/PDF.
PowerPoint: Guarda como PDF.
Illustrator: Exporta como PNG.



Notas

Seguridad: Asegúrate de que tu webhook sea accesible (revisa reglas de seguridad en AWS si usas EC2).
Escalabilidad: Añade nodos para manejar múltiples APIs de IA o destinos (e.g., Slack, Discord).
Pruebas: Usa un entorno de prueba para evitar costos de API innecesarios.
Integración con Tutorial Anterior: Este flujo puede ejecutarse en la instancia de n8n configurada en AWS EC2 (t3.micro).

¿Necesitas ayuda para importar el flujo en n8n, configurar las credenciales, o extender el flujo (e.g., añadir más nodos para otras APIs de IA)? También puedo proporcionar una versión del JSON adaptada a otro modelo o destino.
