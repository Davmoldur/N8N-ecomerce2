# Checkpoint 4 – Integraciones Avanzadas e Interconexión de Sistemas

## AI Automation – Coderhouse

**Autor:** David Molina  
**Proyecto:** Sistema Multi-Agente para E-commerce  
**Módulo:** Integraciones Avanzadas  
**Plataforma de automatización:** n8n

---

## 📌 Descripción del proyecto

Este proyecto corresponde al Checkpoint 4 del curso de AI Automation de Coderhouse.

El objetivo es ampliar el sistema de atención al cliente desarrollado en los módulos anteriores, conectándolo con herramientas externas reales de una organización.

El workflow integra:

- Gmail como canal de entrada y gestión de correos.
- HubSpot como CRM para la gestión de contactos.
- Slack como canal de notificación para el equipo de operaciones.
- OpenAI para analizar las solicitudes de los clientes y generar respuestas.
- n8n como plataforma de automatización y orquestación del flujo.

El sistema está diseñado para procesar solicitudes de clientes de un e-commerce incorporando controles preventivos, validación de contactos y aprobación humana antes del envío de respuestas.

---

## 🏗️ Arquitectura general

El flujo principal implementado sigue esta estructura:

Gmail Trigger  
↓  
IF – Filtro anti auto-reply  
↓  
AI Agent  
↓  
Edit Fields / Limpieza de Payload  
↓  
HubSpot – Search Contacts  
↓  
IF – ¿Existe el contacto?  
↓  
Create or Update Contact  
↓  
Gmail – Create Draft  
↓  
Edit Fields – Preparar Log  
↓  
Slack – Notificación al equipo

---

## 🔧 Integraciones utilizadas

### Gmail

Gmail funciona como punto de entrada del workflow.

El `Gmail Trigger` detecta nuevos correos recibidos en la casilla de soporte.

También se utiliza Gmail al final del proceso mediante la operación **Create Draft**, de manera que la respuesta generada por IA no sea enviada automáticamente.

Esto implementa un mecanismo **Human-in-the-loop (HITL)**: una persona debe revisar y aprobar el borrador antes de enviarlo al cliente.

### HubSpot

HubSpot funciona como CRM del sistema.

Antes de registrar información del cliente se ejecuta una búsqueda mediante **Search Contacts** utilizando el correo electrónico.

Esta validación permite determinar si el contacto ya existe antes de realizar una operación de creación o actualización, reduciendo el riesgo de contactos duplicados.

### Slack

Slack funciona como canal de observabilidad y notificación para el equipo de operaciones.

Después de crear el borrador en Gmail, el workflow genera un payload reducido con la información relevante y envía una notificación al canal correspondiente.

---

## 🛡️ Guardrails implementados

### 1. Filtro anti auto-reply

Después del Gmail Trigger se utiliza un nodo `IF` para detectar mensajes que no deben continuar por el workflow.

El control permite detener correos automáticos como:

- Auto-reply
- Out of office
- Undeliverable
- Remitentes no-reply

Esto ayuda a prevenir bucles infinitos de respuestas automáticas.

### 2. Prevención de contactos duplicados

Antes de modificar el CRM se utiliza:

`HubSpot Search Contacts → IF → Create or Update Contact`

De esta manera se consulta primero la existencia del contacto antes de realizar la siguiente operación sobre el CRM.

### 3. Human-in-the-loop

El workflow **no envía automáticamente la respuesta generada por la IA**.

En su lugar utiliza:

`Gmail → Draft → Create`

El mensaje queda almacenado como borrador para que una persona pueda revisarlo antes de enviarlo.

### 4. Limpieza del payload

Se utilizan nodos `Edit Fields (Set)` para conservar únicamente los datos necesarios para cada integración.

Entre los campos procesados se encuentran:

- Email
- Asunto
- Mensaje
- Categoría
- Respuesta sugerida
- Estado del proceso

Esto evita transferir información innecesaria o payloads excesivamente grandes entre servicios.

---

## 🤖 Uso de Inteligencia Artificial

El AI Agent analiza el correo recibido y determina la categoría de la solicitud.

Para solicitudes de soporte puede generar una respuesta sugerida utilizando la información proporcionada por el cliente.

Ejemplo:

Entrada:

> "Hola, recibí mi iPhone 15 Pro Max dañado y quiero solicitar un reemplazo."

Clasificación:

`SOPORTE`

Posteriormente, el sistema genera una respuesta sugerida y la almacena como borrador en Gmail para revisión humana.

---

## 🧪 Pruebas realizadas

Se realizaron pruebas manuales del workflow para comprobar los principales escenarios.

### Caso 1 – Correo válido

Se envió una solicitud real de soporte relacionada con un pedido dañado.

Resultado:

`Gmail → AI Agent → HubSpot → Gmail Draft → Slack`

El workflow completó correctamente el procesamiento.

### Caso 2 – Correo que debe ser filtrado

Se probaron mensajes que cumplen las condiciones configuradas en el filtro de entrada.

Resultado:

El nodo `IF` detuvo correctamente la ejecución y el mensaje no llegó al agente ni a las integraciones posteriores.

### Caso 3 – Contacto existente

Se realizó una nueva ejecución utilizando un correo previamente registrado.

Resultado:

HubSpot encontró el contacto existente y el workflow tomó la ruta correspondiente, evitando crear un contacto duplicado.

---

## 🔐 Seguridad

Las integraciones externas fueron configuradas mediante credenciales autenticadas desde n8n.

El diseño aplica los siguientes principios:

- Mínimo privilegio.
- Validación antes de escritura.
- Prevención de duplicados.
- Filtrado de mensajes automáticos.
- Reducción del payload.
- Human-in-the-loop antes del envío de correos.

---

## 📂 Archivo entregable

El repositorio contiene el workflow exportado desde n8n:

`checkpoint4_david_molina.json`

El archivo puede importarse nuevamente en n8n para visualizar la arquitectura y configuración del workflow.

> Las credenciales y tokens de autenticación no se incluyen directamente en el repositorio. Cada integración debe autenticarse nuevamente en la instancia de n8n donde se importe el workflow.

---

## 🛠️ Tecnologías utilizadas

- n8n
- OpenAI
- Gmail
- HubSpot CRM
- Slack
- OAuth2

---

## 🎯 Resultado

El Checkpoint 4 extiende el sistema de atención al cliente hacia un ecosistema conectado con herramientas empresariales reales.

La solución incorpora los cuatro controles principales requeridos:

1. **IF anti auto-reply** para prevenir bucles.
2. **Lookup en CRM** antes de modificar contactos.
3. **Create Draft en Gmail** como mecanismo Human-in-the-loop.
4. **Edit Fields / Set** para limpiar y reducir el payload.

El resultado es un workflow capaz de recibir solicitudes por correo, analizarlas mediante IA, sincronizar información con el CRM, preparar una respuesta para aprobación humana y notificar al equipo de operaciones mediante Slack.

## Captura del workflow

<img width="2557" height="1321" alt="Captura de pantalla 2026-09-07 a la(s) 11 54 24 a m" src="https://github.com/user-attachments/assets/f4543104-1149-4d41-9f3d-03b7b5e09fd6" />
<img width="2566" height="1337" alt="Captura de pantalla 2026-09-07 a la(s) 11 54 14 a m" src="https://github.com/user-attachments/assets/9f9682e0-d5dd-4bbb-bdc8-f58c6ee839d7" />
<img width="2537" height="1302" alt="Captura de pantalla 2026-09-07 a la(s) 11 53 58 a m" src="https://github.com/user-attachments/assets/2f225c0c-13e1-4b65-880a-484ac41bd7b8" />


