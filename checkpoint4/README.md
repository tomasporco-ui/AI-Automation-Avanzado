# Checkpoint 4 – Integraciones (Tomás Porco)

Archivo del workflow: `checkpoint4_tomas_porco.json`

Seguí con el mismo agente de los checkpoints anteriores (Manager-Worker + memoria) y le sumé tres herramientas externas:

- **Gmail**: la casilla de soporte. Los mails entran por acá y las respuestas quedan como borrador.
- **Airtable**: lo uso como CRM (base "CRM Tienda"). Con HubSpot me pedían el DNI para registrarme, así que lo cambié con el OK de la profe.
- **Slack**: avisa en #operaciones cuando hay un borrador para revisar.

## Credenciales

![Credenciales en n8n](credenciales.png)

Gmail y Slack están conectados por OAuth2. Airtable, con un Personal Access Token.

## Cómo funciona

1. Entra un mail a Gmail.
2. Un IF revisa si es una respuesta automática y, si lo es, corta ahí.
3. Se limpia el mail (remitente, asunto y texto) y se valida el email.
4. El agente del M3 busca la memoria del cliente, el Router deriva al Worker que corresponde y se arma la respuesta.
5. Busca el contacto en Airtable: si existe lo actualiza y, si no, lo crea.
6. La respuesta queda como **borrador** en Gmail, dentro del mismo hilo.
7. Llega un aviso a Slack para que alguien la revise y la mande.

Como ahora la memoria se guarda por email del remitente, el agente se acuerda de cada cliente.

## Los 4 controles

1. **IF anti auto-reply**: está pegado al trigger. Corta los mails con asuntos como "Out of office", "Auto-reply", "Undeliverable" o "Respuesta automática", los que vienen de direcciones no-reply@ y los newsletters. Así se evita el bucle de respuestas automáticas.
2. **Look up antes de crear**: antes de crear un contacto en Airtable lo busca por email. Si ya existe, actualiza esa misma fila y le suma una interacción. Así no hay duplicados (error 409).
3. **Create Draft**: la IA nunca envía nada. Solo deja el borrador y una persona lo revisa antes de mandarlo (Human in the loop).
4. **Set de limpieza**: solo pasan el remitente, el asunto y el texto recortado, sin HTML ni adjuntos. Antes, un filtro descarta los mails vacíos o con el email mal formado (error 400). A Slack también le llega solo un resumen corto.

Además, el trigger solo lee mails no leídos de la bandeja Principal y que no sean míos, así no procesa promociones ni mis propios logs.

## Mínimo privilegio

- **Airtable**: el token tiene acceso solo a la base CRM Tienda, con permisos para leer y escribir registros y ver la estructura. Nada más.
- **Gmail**: lo único que se escribe son borradores y marcar como leído. No hay ningún nodo que envíe ni borre mails, y no se descargan adjuntos.
- **Slack**: el bot está solo en #operaciones y lo único que hace es mandar mensajes.

## Pruebas

| Prueba | Resultado |
|---|---|
| Mail con asunto "Out of office" | Corta en el IF ✅ |
| Consulta de un cliente nuevo | Lo crea en Airtable, deja el borrador y avisa en Slack ✅ |
| Segunda consulta del mismo cliente | Actualiza la misma fila (Interacciones = 2), sin duplicar ✅ |

![Ejecución completa](ejecucion.png)

![Auto-reply cortado en el IF](autoreply.png)

## Para importarlo

Importar el JSON en n8n y asignar las credenciales (Anthropic, Google Sheets, Gmail, Airtable y Slack). En Airtable hay que elegir la base y la tabla. Los Workers del Checkpoint 2 tienen que estar en la misma cuenta de n8n.
