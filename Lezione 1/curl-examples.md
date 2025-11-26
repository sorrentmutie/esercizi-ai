# Script CURL per Azure OpenAI - LAB 1

Questi script possono essere eseguiti da terminale (Linux/Mac) o Git Bash (Windows).

## Setup Variabili Ambiente

Prima di eseguire gli script, imposta le variabili ambiente:

```bash
# Imposta le tue credenziali
export AZURE_OPENAI_ENDPOINT="https://TUO-NOME-RISORSA.openai.azure.com"
export AZURE_OPENAI_KEY="la-tua-api-key-qui"
export DEPLOYMENT_NAME="gpt-4o"
export API_VERSION="2024-08-01-preview"
```

**Windows (PowerShell):**
```powershell
$env:AZURE_OPENAI_ENDPOINT = "https://TUO-NOME-RISORSA.openai.azure.com"
$env:AZURE_OPENAI_KEY = "la-tua-api-key-qui"
$env:DEPLOYMENT_NAME = "gpt-4o"
$env:API_VERSION = "2024-08-01-preview"
```

---

## 1. Chiamata Base

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Ciao! Spiegami in una frase cos'\''è Azure OpenAI."
      }
    ],
    "max_tokens": 100,
    "temperature": 0.7
  }'
```

---

## 2. Con System Prompt

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "system",
        "content": "Sei un esperto C# che spiega concetti usando analogie semplici."
      },
      {
        "role": "user",
        "content": "Cos'\''è un delegate in C#?"
      }
    ],
    "max_tokens": 150,
    "temperature": 0.7
  }'
```

---

## 3. Temperature Bassa (Deterministico)

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Scrivi uno slogan per una pizzeria"
      }
    ],
    "max_tokens": 50,
    "temperature": 0.1
  }'
```

---

## 4. Temperature Alta (Creativo)

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Scrivi uno slogan per una pizzeria"
      }
    ],
    "max_tokens": 50,
    "temperature": 1.8
  }'
```

---

## 5. Conversazione Multi-turno

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "system",
        "content": "Sei un assistente tecnico per sviluppatori C#."
      },
      {
        "role": "user",
        "content": "Come creo una lista generica in C#?"
      },
      {
        "role": "assistant",
        "content": "Puoi creare una lista generica usando List<T>. Esempio: List<int> numeri = new List<int>();"
      },
      {
        "role": "user",
        "content": "E come aggiungo elementi?"
      }
    ],
    "max_tokens": 100,
    "temperature": 0.7
  }'
```

---

## 6. Richiesta JSON

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "system",
        "content": "Rispondi sempre con JSON valido."
      },
      {
        "role": "user",
        "content": "Dammi 3 linguaggi di programmazione con loro caratteristiche"
      }
    ],
    "max_tokens": 300,
    "temperature": 0.3,
    "response_format": {
      "type": "json_object"
    }
  }'
```

---

## 7. Generazione Codice

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "system",
        "content": "Sei un esperto sviluppatore C#. Scrivi codice pulito e commentato."
      },
      {
        "role": "user",
        "content": "Scrivi una classe C# per gestire un carrello della spesa"
      }
    ],
    "max_tokens": 500,
    "temperature": 0.2
  }'
```

---

## Parsing della Risposta con jq

Se hai `jq` installato, puoi parsare facilmente la risposta:

```bash
# Estrarre solo il contenuto della risposta
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Ciao!"}],
    "max_tokens": 100
  }' | jq -r '.choices[0].message.content'

# Estrarre anche i token usati
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Ciao!"}],
    "max_tokens": 100
  }' | jq '{content: .choices[0].message.content, tokens: .usage.total_tokens}'
```

---

## Salvare Risposta in File

```bash
curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Scrivi un tutorial su Azure Functions"
      }
    ],
    "max_tokens": 1000
  }' > risposta.json

# Visualizzare solo il contenuto
cat risposta.json | jq -r '.choices[0].message.content'
```

---

## Gestione Errori

```bash
# Cattura codice di stato HTTP
HTTP_CODE=$(curl -X POST "${AZURE_OPENAI_ENDPOINT}/openai/deployments/${DEPLOYMENT_NAME}/chat/completions?api-version=${API_VERSION}" \
  -H "api-key: ${AZURE_OPENAI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Test"}], "max_tokens": 50}' \
  -w "%{http_code}" \
  -o response.json \
  -s)

if [ "$HTTP_CODE" -eq 200 ]; then
  echo "✓ Successo!"
  cat response.json | jq -r '.choices[0].message.content'
else
  echo "✗ Errore HTTP $HTTP_CODE"
  cat response.json | jq '.'
fi
```

---

## Note Importanti

### Caratteri Speciali
In bash, l'apostrofo nelle stringhe JSON va escapato: `'\''`

Esempio:
```bash
"content": "Cos'\''è un delegate?"
```

### Windows PowerShell
Per eseguire curl in PowerShell, i comandi sono leggermente diversi:

```powershell
curl -X POST "$env:AZURE_OPENAI_ENDPOINT/openai/deployments/$env:DEPLOYMENT_NAME/chat/completions?api-version=$env:API_VERSION" `
  -H "api-key: $env:AZURE_OPENAI_KEY" `
  -H "Content-Type: application/json" `
  -d '{\"messages\": [{\"role\": \"user\", \"content\": \"Ciao!\"}], \"max_tokens\": 100}'
```

### Rate Limiting
Se ricevi errore 429 (Too Many Requests):
- Attendi 60 secondi
- Riduci la frequenza delle richieste
- Aumenta TPM del deployment se necessario

---

## Troubleshooting

**Errore: "curl: command not found"**
- Linux: `sudo apt-get install curl`
- Mac: curl è preinstallato
- Windows: usa Git Bash o PowerShell

**Errore 401 Unauthorized**
- Verifica che `AZURE_OPENAI_KEY` sia impostata correttamente
- Controlla che non ci siano spazi extra
- Rigenera la key se necessario

**Errore 404 Not Found**
- Verifica `DEPLOYMENT_NAME` corrisponda al tuo deployment
- Controlla `AZURE_OPENAI_ENDPOINT` sia corretto
- Assicurati che il deployment sia completato (Succeeded)

**Risposta vuota o malformata**
- Verifica la sintassi JSON del body
- Controlla le virgole e le parentesi
- Usa un validatore JSON online

---

**Fine Script CURL** ✨
