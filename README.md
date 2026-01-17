# Gmail Task Order — Documentación

**ID del flujo:** `yhhlC3PXdaZxaGqws8Tpc`

**Resumen:**
Este workflow convierte correos entrantes de Gmail en tareas accionables dentro de Trello. Cada nuevo email detectado se lee, se transforma en una tarjeta (usando el asunto como título y el cuerpo como descripción) y finalmente se marca como leído para evitar reprocesos. Ideal para equipos que gestionan pedidos, solicitudes o tareas directamente desde su bandeja de entrada.

---

## Tabla de contenidos

1. Objetivo
2. Casos de uso
3. Requisitos previos
4. Diagrama lógico del flujo
5. Descripción nodo a nodo
6. Mapeo de datos (Email → Trello)
7. Ejemplo de ejecución
8. Configuración de credenciales
9. Uso, pruebas y despliegue
10. Manejo de errores y consideraciones
11. Mejoras recomendadas

---

## 1) Objetivo

Automatizar la creación de tareas en Trello a partir de correos electrónicos entrantes, reduciendo trabajo manual, evitando olvidos y asegurando que cada email relevante termine convertido en una tarjeta organizada.

## 2) Casos de uso

* Pedidos que llegan por correo y deben convertirse en tareas.
* Solicitudes de clientes enviadas por email.
* Incidencias o tickets recibidos en una bandeja compartida.
* Automatización de inbox para equipos de soporte u operaciones.

## 3) Requisitos previos

* Instancia de n8n activa.
* Cuenta de Gmail con OAuth2 configurado en n8n.
* Cuenta de Trello con acceso a la lista destino.
* ID de la lista de Trello (`listId`) donde se crearán las tarjetas.
* Permisos OAuth válidos para Gmail (lectura y modificación de mensajes).

## 4) Diagrama lógico del flujo

* `Gmail Trigger` (nuevo email) → `Get a message` (leer contenido completo) → `Edit Fields` (mapear datos) → `Create a card` (Trello) → `Mark a message as read`

## 5) Descripción nodo a nodo

### 5.1 Gmail Trigger

* **Tipo:** `gmailTrigger` (v1.3)
* **Frecuencia:** `everyMinute`
* **Función:** Detecta nuevos correos entrantes en la cuenta de Gmail configurada.
* **Salida principal:** ID del mensaje (`$json.id`).
* **Credencial:** `gmailOAuth2`.

**Nota:** Se pueden añadir filtros (por remitente, asunto o etiquetas) para limitar qué correos disparan el flujo.

### 5.2 Get a message

* **Tipo:** `gmail` (operation: `get`)
* **Entrada:** `messageId = {{ $json.id }}`
* **Modo:** `simple = false` (obtiene contenido completo del email).
* **Función:** Recupera el asunto, cuerpo del mensaje y metadatos del correo.

### 5.3 Edit Fields

* **Tipo:** `set`
* **Campos definidos:**

  * `Label` → `{{ $json.subject }}`
  * `Content` → `{{ $json.text }}`
* **Función:** Normaliza los datos del email para que puedan ser reutilizados fácilmente en Trello.

### 5.4 Create a card

* **Tipo:** `trello`
* **Operación:** Crear tarjeta
* **Parámetros:**

  * `listId`: `6827326d537f44b4b514ad8f`
  * `name`: `{{ $json.Label }}` (asunto del email)
  * `description`: `{{ $json.Content }}` (contenido del email)
* **Función:** Crea una nueva tarjeta en la lista indicada de Trello.
* **Credencial:** `trelloApi`.

### 5.5 Mark a message as read

* **Tipo:** `gmail`
* **Operación:** `markAsRead`
* **Entrada:** `messageId = {{ $('Gmail Trigger').item.json.id }}`
* **Función:** Marca el correo como leído una vez procesado, evitando que vuelva a disparar el flujo.

## 6) Mapeo de datos (Email → Trello)

| Email (Gmail) | Trello           |
| ------------- | ---------------- |
| Subject       | Card name        |
| Body (text)   | Card description |

## 7) Ejemplo de ejecución

**Correo entrante:**

* Asunto: `Pedido #245 - Nuevo cliente`
* Cuerpo: `Hola, necesito cotizar 10 unidades...`

**Resultado en Trello:**

* Tarjeta creada con título **Pedido #245 - Nuevo cliente**
* Descripción contiene el texto completo del correo.
* El email queda marcado como leído en Gmail.

## 8) Configuración de credenciales

### Gmail

1. Crear credenciales OAuth2 en Google Cloud.
2. Autorizar scopes para lectura y modificación de correos.
3. Configurar `gmailOAuth2` en n8n.

### Trello

1. Generar API Key y Token desde Trello.
2. Configurar la credencial `trelloApi` en n8n.
3. Obtener el `listId` de la lista destino.

## 9) Uso, pruebas y despliegue

* Enviar un correo de prueba a la cuenta configurada.
* Verificar la ejecución en el historial de n8n.
* Confirmar creación de la tarjeta en Trello.
* Confirmar que el correo se marca como leído.

## 10) Manejo de errores y consideraciones

* Si falla Trello, el correo no se marcará como leído (permite reintento).
* Correos muy largos pueden truncarse según límites de Trello.
* HTML no se procesa; se usa texto plano (`$json.text`).
* Considerar rate limits si llegan muchos correos por minuto.

## 11) Mejoras recomendadas

* **Filtros inteligentes:** Procesar solo correos con cierto asunto o remitente.
* **Etiquetas Gmail:** Aplicar una label tipo `Processed` además de marcar como leído.
* **Adjuntos:** Descargar y adjuntar archivos del email a la tarjeta de Trello.
* **Asignación automática:** Asignar miembros o etiquetas en Trello según reglas.
* **Prioridades:** Detectar palabras clave (URGENTE, ALTA) y asignar labels.

