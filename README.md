{
  "name": "Agendamento Unhas - Gemini + Sheets + Calendar",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "whatsapp-unhas",
        "options": {}
      },
      "id": "768340d9-7607-4e01-9a91-4c281358b534",
      "name": "Webhook WhatsApp",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [0, 200]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Você é uma assistente de agendamento para um salão de unhas.\n\n### REGRAS DE ATENDIMENTO:\n- Terça a Sexta: 14h às 20h.\n- Sábado: 09h às 17h.\n- Duração do serviço: 1 hora e 40 minutos.\n\n### FLUXO DE TRABALHO:\n1. Se o cliente pedir um horário, verifique se está dentro das regras acima. Se estiver, salve na planilha como 'PENDENTE' e pergunte: 'Tenho disponível o dia {{ $now.plus({days:1}).format('dd/MM') }} às [Hora]. Posso confirmar?'\n2. Se o cliente disser 'SIM', 'PODE' ou confirmar, busque o registro na planilha, altere para 'CONFIRMADO' e use a ferramenta para criar DOIS eventos no calendário: o atual e outro exatamente 15 dias depois no mesmo horário.\n3. Se o cliente quiser CANCELAR, localize o agendamento e remova do calendário.\n\nData e Hora atual: {{ $now.setZone('America/Sao_Paulo').toString() }}",
        "options": {
          "systemMessage": "Você gerencia agendamentos de manutenção de unhas. Seja educada e precisa com horários."
        }
      },
      "id": "e3a89e83-7d72-404f-956e-8f1523c52402",
      "name": "AI Agent Gemini",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.6,
      "position": [220, 200]
    },
    {
      "parameters": {
        "modelName": "models/gemini-1.5-flash",
        "options": {
          "temperature": 0.2
        }
      },
      "id": "b7890f12-3456-4789-a123-bcde4567890f",
      "name": "Google Gemini",
      "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
      "typeVersion": 1,
      "position": [150, 420],
      "credentials": {
        "googlePalmApi": {
          "id": "SUA_API_KEY_GEMINI"
        }
      }
    },
    {
      "parameters": {
        "operation": "append",
        "documentId": {
          "__rl": true,
          "value": "ID_DA_SUA_PLANILHA",
          "mode": "id"
        },
        "sheetName": "Agendamentos",
        "columns": {
          "mappingMode": "defineColumns",
          "value": {
            "Nome": "={{ $json.query }}",
            "Status": "PENDENTE",
            "Data": "={{ $now }}"
          }
        }
      },
      "id": "f1234567-890a-4b2c-c3d4-e5f6a7b8c9d0",
      "name": "Google Sheets Tool",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4.5,
      "position": [450, 420],
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "SUA_CREDENCIAL_SHEETS"
        }
      }
    },
    {
      "parameters": {
        "calendar": {
          "__rl": true,
          "value": "primary",
          "mode": "list"
        },
        "start": "={{ $json.data_agendamento }}",
        "end": "={{ $json.data_agendamento.plus({minutes: 100}) }}",
        "summary": "={{ $json.cliente_nome }} - Unhas",
        "additionalFields": {
          "description": "Agendamento Principal"
        }
      },
      "id": "a1b2c3d4-e5f6-4g7h-8i9j-k0l1m2n3o4p5",
      "name": "Google Calendar (Hoje)",
      "type": "n8n-nodes-base.googleCalendar",
      "typeVersion": 1.1,
      "position": [680, 100],
      "credentials": {
        "googleCalendarOAuth2Api": {
          "id": "SUA_CREDENCIAL_CALENDAR"
        }
      }
    },
    {
      "parameters": {
        "calendar": {
          "__rl": true,
          "value": "primary",
          "mode": "list"
        },
        "start": "={{ $json.data_agendamento.plus({days: 15}) }}",
        "end": "={{ $json.data_agendamento.plus({days: 15, minutes: 100}) }}",
        "summary": "={{ $json.cliente_nome }} - Manutenção (15 dias)",
        "additionalFields": {
          "description": "Retorno Automático de 15 dias"
        }
      },
      "id": "z9y8x7w6-v5u4-t3s2-r1q0-p9o8n7m6l5k4",
      "name": "Google Calendar (15 Dias)",
      "type": "n8n-nodes-base.googleCalendar",
      "typeVersion": 1.1,
      "position": [680, 300],
      "credentials": {
        "googleCalendarOAuth2Api": {
          "id": "SUA_CREDENCIAL_CALENDAR"
        }
      }
    }
  ],
  "connections": {
    "Webhook WhatsApp": {
      "main": [
        [
          {
            "node": "AI Agent Gemini",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Google Gemini": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent Gemini",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Google Sheets Tool": {
      "main": [
        [
          {
            "node": "AI Agent Gemini",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Google Calendar (Hoje)": {
      "main": [
        [
          {
            "node": "Google Calendar (15 Dias)",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
