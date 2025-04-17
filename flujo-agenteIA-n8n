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
        "

