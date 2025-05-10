#
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "lead-webhook"
      },
      "name": "Webhook Entrada",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [300, 300]
    },
    {
      "parameters": {
        "functionCode": "const email = $json[\"email\"];\nif (!email.includes(\"@\")) throw new Error(\"Email inválido\");\nreturn [{json: {email}}];"
      },
      "name": "Validação de e-mail",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [540, 300]
    },
    {
      "parameters": {
        "operation": "append",
        "sheetId": "1hKqETC_fake_id", 
        "range": "Leads!A:C",
        "options": {}
      },
      "name": "Gravar no Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 2,
      "position": [780, 300]
    },
    {
      "parameters": {
        "authentication": "headerAuth",
        "requestMethod": "POST",
        "url": "https://api.hubspot.com/crm/v3/objects/contacts",
        "jsonParameters": true,
        "options": {},
        "bodyParametersJson": "{\"properties\":{\"email\":\"={{$json.email}}\"}}"
      },
      "name": "Criar contato HubSpot",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 2,
      "position": [1020, 300]
    }
  ],
  "connections": {
    "Webhook Entrada": { "main": [[{ "node": "Validação de e-mail", "type": "main", "index": 0 }]]},
    "Validação de e-mail": { "main": [[{ "node": "Gravar no Google Sheets", "type": "main", "index": 0 }]]},
    "Gravar no Google Sheets": { "main": [[{ "node": "Criar contato HubSpot", "type": "main", "index": 0 }]]}
  }
}
 automatic-octo-doodle
