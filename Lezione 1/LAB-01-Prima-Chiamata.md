# LAB 1: Primo Deployment e Chiamata Azure OpenAI

**Durata:** 30 minuti  
**Obiettivo:** Creare la prima risorsa Azure OpenAI, deployare GPT-4o e fare la prima chiamata REST

---

## 📋 Prerequisiti

Prima di iniziare, assicurati di avere:

- ✅ **Subscription Azure attiva** con crediti disponibili
- ✅ **Permessi adeguati** (Owner, Contributor o Cognitive Services Contributor)
- ✅ **Accesso Azure OpenAI approvato** ([Richiedi qui](https://aka.ms/oai/access) se non lo hai ancora)
- ✅ **Postman installato** o usa la [versione web](https://www.postman.com/downloads/)

---

## ⚠️ IMPORTANTE: Gestione Costi

Azure OpenAI è un servizio pay-per-use. Per questo LAB:

- **Costo stimato:** €0.10 - €0.50 per tutto il laboratorio
- **Budget raccomandato:** Imposteremo un alert a €5
- **Best practice:** Cancella le risorse al termine se non ti servono più

---

## Parte 1: Creare la Risorsa Azure OpenAI (10 minuti)

### Step 1.1: Accedere al portale Azure

1. Apri il browser e vai su [https://portal.azure.com](https://portal.azure.com)
2. Effettua il login con il tuo account Azure

### Step 1.2: Cercare il servizio Azure OpenAI

1. Nella barra di ricerca in alto, digita: **Azure OpenAI**
2. Nei risultati, clicca su **Azure OpenAI** (icona con il logo OpenAI)
3. Clicca sul pulsante **+ Create** oppure **Create Azure OpenAI**

### Step 1.3: Configurare la risorsa

Nella schermata "Create Azure OpenAI", compila i campi:

#### **Tab: Basics**

| Campo | Valore | Note |
|-------|--------|------|
| **Subscription** | Seleziona la tua subscription | Quella con crediti disponibili |
| **Resource Group** | Crea nuovo: `rg-azureopenai-lab` | Oppure usa uno esistente |
| **Region** | **Sweden Central** | ⚠️ Consigliato: ha GPT-4o disponibile |
| **Name** | `aoai-lab-<tuonome>` | Es: `aoai-lab-salvatore` |
| **Pricing tier** | **Standard S0** | L'unico disponibile per OpenAI |

> **💡 Importante sulla Region:**  
> Non tutte le region hanno tutti i modelli. Sweden Central e East US hanno la migliore disponibilità.  
> [Verifica qui](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#model-summary-table-and-region-availability) l'elenco aggiornato.

#### **Tab: Network** (opzionale, per questo lab lascia default)

- **Connectivity method:** All networks (default)

> In produzione, useresti **Private endpoint** per maggiore sicurezza.

#### **Tab: Tags** (opzionale ma raccomandato)

Aggiungi tags per organizzare meglio le risorse:

| Tag Name | Tag Value |
|----------|-----------|
| Environment | Lab |
| Project | Corso-AzureAI |

#### **Tab: Review + create**

1. Rivedi la configurazione
2. Clicca **Create**
3. Attendi 1-2 minuti per il deployment

✅ **Checkpoint:** Dovresti vedere "Your deployment is complete"

---

### Step 1.4: Configurare Budget Alert

Per evitare costi imprevisti, impostiamo un alert:

1. Dal menu laterale del portale Azure, clicca su **Cost Management + Billing**
2. Seleziona **Cost Management** → **Budgets**
3. Clicca **+ Add**
4. Configura:
   - **Name:** `Budget-AzureOpenAI-Lab`
   - **Amount:** `5` EUR (o USD)
   - **Reset period:** Monthly
   - **Alert conditions:** 
     - 50% (€2.50) → Email warning
     - 80% (€4.00) → Email alert
     - 100% (€5.00) → Email critical alert
5. Inserisci la tua email
6. Clicca **Create**

✅ **Checkpoint:** Riceverai una email di conferma

---

## Parte 2: Deployare il Modello GPT-4o (5 minuti)

### Step 2.1: Aprire la risorsa Azure OpenAI

1. Nel portale Azure, vai alla risorsa appena creata
   - Metodo 1: Clicca su "Go to resource" dalla notifica di deployment
   - Metodo 2: Home → Cerca il nome della tua risorsa
2. Dovresti vedere la pagina Overview della risorsa

### Step 2.2: Accedere a Azure OpenAI Studio

1. Nella pagina Overview, cerca il pulsante **"Go to Azure OpenAI Studio"**
2. Clicca sul pulsante (si apre una nuova tab)
3. Oppure vai direttamente su [https://oai.azure.com/portal](https://oai.azure.com/portal)

### Step 2.3: Creare un Deployment

1. Nel menu laterale sinistro, clicca su **Deployments**
2. Clicca sul pulsante **+ Create new deployment**
3. Nella finestra "Deploy model", compila:

| Campo | Valore | Note |
|-------|--------|------|
| **Select a model** | `gpt-4o` | Il modello più recente |
| **Model version** | Seleziona l'ultima disponibile | Es: `2024-08-06` o `latest` |
| **Deployment name** | `gpt-4o` | ⚠️ Userai QUESTO nome nelle API |
| **Deployment type** | Standard | Default |
| **Tokens per Minute Rate Limit (thousands)** | `10` | 10K TPM è sufficiente per test |

4. Clicca **Create**
5. Attendi 10-30 secondi

✅ **Checkpoint:** Il deployment appare nella lista con stato "Succeeded"

> **💡 Nota sul Deployment Name:**  
> Il nome che dai qui (es: `gpt-4o`) è quello che userai nell'URL delle API.  
> Puoi chiamarlo come vuoi, non deve essere necessariamente "gpt-4o".

---

## Parte 3: Ottenere Keys ed Endpoint (2 minuti)

### Step 3.1: Trovare Keys ed Endpoint

1. Torna al portale Azure (tab principale)
2. Nella tua risorsa Azure OpenAI, menu laterale sinistro
3. Clicca su **Keys and Endpoint**

Dovresti vedere:

- **KEY 1:** Una stringa lunga tipo `a1b2c3d4e5f6...`
- **KEY 2:** Un'altra stringa
- **Endpoint:** Un URL tipo `https://aoai-lab-salvatore.openai.azure.com/`
- **Location:** La region (es: `swedencentral`)

### Step 3.2: Copiare le credenziali

1. **Copia KEY 1** (clicca sull'icona copia)
2. **Copia l'Endpoint** (clicca sull'icona copia)
3. Salvali in un file temporaneo (li useremo tra poco)

> **🔒 Sicurezza:**  
> Queste keys danno accesso completo alla tua risorsa.  
> - ❌ Non committarle su Git
> - ❌ Non condividerle pubblicamente
> - ✅ Usale solo in locale per questo lab

### Step 3.3: Annotare il Deployment Name

Ricordati anche il nome del deployment che hai creato:
- Nel nostro caso: `gpt-4o`
- Lo useremo nell'URL API

✅ **Checkpoint:** Hai copiato:
- API Key
- Endpoint URL  
- Deployment Name

---

## Parte 4: Prima Chiamata con Postman (10 minuti)

### Step 4.1: Importare la Collection Postman

1. Apri Postman (desktop o web: [postman.com/downloads](https://www.postman.com/downloads/))
2. Clicca su **Import** (in alto a sinistra)
3. Seleziona il file `Azure-OpenAI-Lab.postman_collection.json` fornito
4. Clicca **Import**

### Step 4.2: Configurare l'Environment

1. In Postman, clicca sull'icona **Environments** (a sinistra, icona ingranaggio)
2. Clicca **+ Create Environment**  
3. Nominalo: `Azure OpenAI Lab`
4. Aggiungi queste variabili:

| Variable | Type | Initial Value | Current Value |
|----------|------|---------------|---------------|
| `azure_openai_endpoint` | default | `https://TUO_NOME.openai.azure.com` | Il tuo Endpoint |
| `azure_openai_key` | secret | (lascia vuoto) | La tua KEY 1 |
| `deployment_name` | default | `gpt-4o` | Il tuo Deployment Name |
| `api_version` | default | `2024-08-01-preview` | (lascia così) |

5. Clicca **Save**
6. Seleziona questo environment dal dropdown in alto a destra

> **💡 Nota sui Current Values:**  
> - **Initial Value:** visibile a tutti se condividi
> - **Current Value:** solo locale, usalo per secrets

### Step 4.3: Testare la prima chiamata

1. Nella collection importata, apri **Chat Completion - Simple**
2. Verifica l'URL (dovrebbe usare le variabili):
   ```
   {{azure_openai_endpoint}}/openai/deployments/{{deployment_name}}/chat/completions?api-version={{api_version}}
   ```

3. Verifica gli Headers:
   ```
   api-key: {{azure_openai_key}}
   Content-Type: application/json
   ```

4. Verifica il Body (tab Body → raw → JSON):
   ```json
   {
     "messages": [
       {
         "role": "user",
         "content": "Ciao! Spiegami in una frase cos'è Azure OpenAI."
       }
     ],
     "max_tokens": 100,
     "temperature": 0.7
   }
   ```

5. Clicca **Send**

### Step 4.4: Interpretare la risposta

Se tutto funziona, dovresti ricevere una risposta simile:

```json
{
  "id": "chatcmpl-xxxxx",
  "object": "chat.completion",
  "created": 1707825602,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Azure OpenAI è un servizio Microsoft che offre accesso ai modelli OpenAI attraverso l'infrastruttura Azure, con sicurezza enterprise e compliance."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 28,
    "total_tokens": 43
  }
}
```

**Campi importanti:**
- `choices[0].message.content`: La risposta generata
- `finish_reason`: "stop" = completato, "length" = troncato
- `usage.total_tokens`: Token usati (43 nel nostro caso)

✅ **Checkpoint:** Hai ricevuto una risposta valida dall'AI!

### Possibili Errori e Soluzioni

| Errore | Causa | Soluzione |
|--------|-------|-----------|
| `401 Unauthorized` | API key errata | Verifica di aver copiato KEY 1 correttamente |
| `404 Not Found` | Deployment name sbagliato | Verifica il nome del deployment |
| `429 Too Many Requests` | Rate limit superato | Attendi 60 secondi e riprova |
| `InvalidRequestError` | Body malformato | Verifica il JSON (virgole, parentesi) |

---

## Parte 5: Sperimentare con i Parametri (10 minuti)

Ora che la chiamata base funziona, sperimentiamo con i parametri!

### Esperimento 1: Temperature

La `temperature` controlla la "creatività" delle risposte (0-2):
- **0:** Deterministico, sempre uguale
- **0.7:** Bilanciato (default)
- **1.5-2:** Molto creativo/casuale

**Test:**

1. Duplica la request (tasto destro → Duplicate)
2. Rinomina in "Chat - Temperature Low"
3. Modifica il body:
   ```json
   {
     "messages": [
       {
         "role": "user",
         "content": "Scrivi uno slogan per una pizzeria"
       }
     ],
     "max_tokens": 50,
     "temperature": 0.1
   }
   ```
4. Invia più volte → nota che le risposte sono molto simili

5. Duplica di nuovo, rinomina "Chat - Temperature High"
6. Cambia `temperature: 1.8`
7. Invia più volte → nota risposte molto diverse e creative

**💡 Quando usare:**
- Temperature bassa (0-0.3): Documentazione, risposte precise, codice
- Temperature media (0.7): Chat generali, spiegazioni
- Temperature alta (1.2-2): Brainstorming, contenuti creativi

### Esperimento 2: Max Tokens

`max_tokens` limita la lunghezza della risposta:

1. Duplica la request originale
2. Rinomina "Chat - Max Tokens Test"
3. Body:
   ```json
   {
     "messages": [
       {
         "role": "user",
         "content": "Spiega dettagliatamente cos'è un'API REST"
       }
     ],
     "max_tokens": 20,
     "temperature": 0.7
   }
   ```
4. Invia → nota che la risposta si tronca
5. Guarda il `finish_reason`: sarà "length" invece di "stop"
6. Aumenta a `max_tokens: 200` e riprova

**💡 Best practice:**
- Stima sempre più del necessario
- Monitora `finish_reason` per rilevare troncamenti
- Costo: paghi solo i token effettivamente generati

### Esperimento 3: System Prompt

Il ruolo `system` definisce il comportamento del modello:

1. Crea nuova request: "Chat - System Prompt"
2. Body:
   ```json
   {
     "messages": [
       {
         "role": "system",
         "content": "Sei un esperto C# che spiega concetti in modo semplice usando analogie dal mondo reale."
       },
       {
         "role": "user",
         "content": "Cos'è un delegate in C#?"
       }
     ],
     "max_tokens": 150,
     "temperature": 0.7
   }
   ```
3. Invia e nota come la risposta usa analogie

4. Cambia il system prompt:
   ```json
   "content": "Sei un tutor paziente che parla a uno studente di 10 anni. Usa esempi semplici e incoraggiamento."
   ```
5. Riprova la stessa domanda → nota il cambio di tono

**💡 System prompts utili:**
- "Rispondi sempre in JSON valido"
- "Sei un assistente conciso: massimo 2 frasi"
- "Sei un code reviewer: trova bug e suggerisci fix"

### Esperimento 4: Conversazione Multi-turno

Le chat funzionano inviando TUTTA la cronologia ogni volta:

1. Crea nuova request: "Chat - Conversation"
2. Body:
   ```json
   {
     "messages": [
       {
         "role": "system",
         "content": "Sei un assistente tecnico"
       },
       {
         "role": "user",
         "content": "Come creo una lista in C#?"
       },
       {
         "role": "assistant",
         "content": "Puoi creare una lista così: List<int> numeri = new List<int>();"
       },
       {
         "role": "user",
         "content": "E come aggiungo elementi?"
       }
     ],
     "max_tokens": 100
   }
   ```
3. Invia → nota che l'AI sa di cosa parli ("elementi alla lista")

**💡 Attenzione ai costi:**
- Ogni messaggio ri-invia TUTTA la cronologia
- Conversazione lunga = molti token prompt
- In produzione: riassumi o tronca vecchi messaggi

---

## Parte 6: (Opzionale) Chiamata con curl

Se preferisci CLI, ecco un esempio curl:

```bash
curl -X POST "https://TUO_ENDPOINT.openai.azure.com/openai/deployments/TUO_DEPLOYMENT/chat/completions?api-version=2024-08-01-preview" \
  -H "api-key: TUA_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Ciao!"
      }
    ],
    "max_tokens": 100
  }'
```

**Sostituisci:**
- `TUO_ENDPOINT`: il tuo endpoint (senza https://)
- `TUO_DEPLOYMENT`: il nome del deployment
- `TUA_KEY`: la tua API key

---

## 📊 Monitorare i Costi

### Controllare l'utilizzo

1. Torna al portale Azure
2. Vai alla tua risorsa Azure OpenAI
3. Menu laterale → **Metrics**
4. Aggiungi metric: **Total Tokens**
5. Puoi vedere:
   - Token usati per periodo
   - Trend di utilizzo
   - Picchi di consumo

### Stimare i costi

Per GPT-4o (prezzi indicativi, verificare su Azure):
- Input: ~$0.005 per 1K token
- Output: ~$0.015 per 1K token

**Esempio calcolo:**
- Hai usato 1000 token in questo lab
- Prompt: 400 token
- Completion: 600 token
- Costo: (400 * 0.005 / 1000) + (600 * 0.015 / 1000) = $0.011 (~€0.01)

### 🗑️ Pulizia Risorse

Se non ti serve più la risorsa:

1. Portale Azure → Resource Groups
2. Seleziona `rg-azureopenai-lab`
3. Clicca **Delete resource group**
4. Conferma digitando il nome del RG
5. Click **Delete**

> Questo cancellerà TUTTO nel resource group.

---

## ✅ Recap: Cosa Hai Imparato

Congratulazioni! Hai:

✅ Creato una risorsa Azure OpenAI  
✅ Configurato budget alert per controllare i costi  
✅ Deployato il modello GPT-4o  
✅ Ottenuto keys ed endpoint  
✅ Fatto la tua prima chiamata REST  
✅ Sperimentato con temperatura, max_tokens, system prompts  
✅ Capito come funzionano le conversazioni multi-turno  

---

## 📚 Risorse Aggiuntive

- [Documentazione Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/)
- [Playground Azure OpenAI Studio](https://oai.azure.com/portal)
- [Prezzi Azure OpenAI](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/)
- [Tokenizer OpenAI](https://platform.openai.com/tokenizer) - per testare tokenizzazione

---

## 🆘 Hai Problemi?

**Problema: Non riesco a creare la risorsa**
- Verifica di avere accesso OpenAI approvato
- Controlla i permessi sulla subscription
- Prova una region diversa

**Problema: 401 Unauthorized**
- Verifica API key copiata correttamente (nessuno spazio extra)
- Rigenera la key se necessario

**Problema: Deployment non trovato**
- Verifica il nome del deployment nell'URL
- Aspetta 30-60 secondi dopo la creazione

**Problema: Rate limit**
- Stai facendo troppe richieste
- Attendi 60 secondi
- Aumenta TPM del deployment se necessario

---

## 🎯 Prossimi Passi

Nella prossima sessione vedremo:
- Content filtering in dettaglio
- Best practices per prompt engineering
- Come integrare Azure OpenAI in applicazioni .NET
- Monitoring e logging avanzati

---

**Fine del LAB 1** ✨
