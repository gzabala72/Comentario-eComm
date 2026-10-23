# Comentario eComm

Workflows de [n8n](https://n8n.io) que reciben por Outlook los comentarios del formulario de e-commerce, los clasifican con IA y los derivan por mail al área correspondiente. Además envían un acuse de recibo al cliente.

## Archivos

| Archivo | Descripción |
|---|---|
| `Llega mail y clasifica v8.json` | Workflow principal: lectura, parseo, clasificación, derivación y acuse. |
| `Error - Llega mail y clasifica.json` | Error workflow: avisa por mail cuando falla una ejecución del principal. |

## Cómo funciona

1. **Entrada**: un Outlook Trigger lee cada minuto los mails nuevos de la carpeta Ecommerce.
2. **Anti-loop**: se descartan los mails generados por el propio workflow (asunto `Ecommerce:` o `Farmar: hemos recibido`).
3. **Parseo**: se extraen los campos del formulario (`Etiqueta: valor`) y se arman el HTML para el equipo y el del acuse al cliente.
4. **Arrepentimiento de compra**: tiene categoría fija, sin pasar por la IA.
5. **Clasificación con IA**: si el comentario no está vacío, un Text Classifier con Anthropic lo asigna a una de 12 categorías (más `other`). Si el comentario está vacío, o si el modelo falla tras 3 intentos, va a `revision_humana`.
6. **Derivación**: cada categoría tiene su nodo de destino y su mail de envío.
7. **Acuse de recibo**: si el email del cliente es válido, se le envía una confirmación (excepto en `spam_irrelevante`).

### Categorías

`arrepentimiento_compra`, `postulacion_laboral`, `queja_reclamo`, `reclamo_salud_seguridad`, `oferta_comercial`, `agradecimiento`, `devolucion_arrepentimiento`, `consulta_general`, `spam_irrelevante`, `revision_humana`, `problema_web_app`, `estado_pedido`, `franquicia_expansion`.

## Puesta en marcha

1. En n8n, importar ambos JSON (**Import from File**).
2. Configurar las credenciales: Microsoft Outlook OAuth2 y Anthropic API.
3. En el Outlook Trigger, elegir la carpeta en **Filters → Folders to Include**.
4. Revisar los destinatarios de cada nodo `Enviar mail - ...`.
5. En el workflow principal, ir a **Settings → Error Workflow** y seleccionar `Error - Llega mail y clasifica`.
6. Activar el workflow.

## Limitaciones

- Si Outlook está caído, el aviso de error también falla. En ese caso hay que revisar **Executions** en n8n.
