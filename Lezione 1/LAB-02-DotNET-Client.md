# LAB 2: Azure OpenAI con .NET

**Durata:** 30 minuti  
**Obiettivo:** Creare un'applicazione Console .NET che interagisce con Azure OpenAI usando le best practices

---

## 📋 Prerequisiti

- ✅ **Risorsa Azure OpenAI** creata (dal LAB 1)
- ✅ **Deployment GPT-4o** attivo
- ✅ **API Key ed Endpoint** disponibili
- ✅ **.NET 8 SDK** installato ([Download qui](https://dotnet.microsoft.com/download))
- ✅ **Visual Studio 2022** o **VS Code** con C# extension

---

## Parte 1: Setup Progetto (5 minuti)

### Step 1.1: Creare il progetto Console

Apri un terminale/PowerShell e esegui:

```bash
# Crea una nuova directory per il progetto
mkdir AzureOpenAILab
cd AzureOpenAILab

# Crea un progetto Console .NET 8
dotnet new console -n ChatApp
cd ChatApp

# Apri con VS Code (opzionale)
code .
```

### Step 1.2: Installare i pacchetti NuGet necessari

```bash
# Azure OpenAI SDK (versione stabile)
dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.17

# Per gestire secrets in sviluppo
dotnet add package Microsoft.Extensions.Configuration.UserSecrets

# Per dependency injection e logging
dotnet add package Microsoft.Extensions.Hosting
dotnet add package Microsoft.Extensions.Logging.Console
```

✅ **Checkpoint:** Verifica che il file `.csproj` contenga i package references

### Step 1.3: Configurare User Secrets

Per non hard-codare le credenziali:

```bash
# Inizializza user secrets
dotnet user-secrets init

# Imposta le tue credenziali (sostituisci con le tue!)
dotnet user-secrets set "AzureOpenAI:Endpoint" "https://TUO-NOME.openai.azure.com/"
dotnet user-secrets set "AzureOpenAI:Key" "la-tua-api-key-qui"
dotnet user-secrets set "AzureOpenAI:DeploymentName" "gpt-4o"
```

> **💡 Cos'è User Secrets?**  
> Storage sicuro per credenziali in sviluppo. I secrets NON finiscono su Git.  
> In produzione userai Azure Key Vault.

---

## Parte 2: Prima Applicazione (10 minuti)

### Step 2.1: Creare la configurazione

Crea un file `appsettings.json`:

```json
{
  "AzureOpenAI": {
    "Endpoint": "",
    "Key": "",
    "DeploymentName": "gpt-4o"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

### Step 2.2: Codice base - Program.cs

Sostituisci il contenuto di `Program.cs`:

```csharp
using Azure;
using Azure.AI.OpenAI;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;

// === CONFIGURAZIONE ===
var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: true)
    .AddUserSecrets<Program>()  // Secrets in sviluppo
    .Build();

// Leggi le configurazioni
var endpoint = configuration["AzureOpenAI:Endpoint"] 
    ?? throw new Exception("Endpoint non configurato");
var key = configuration["AzureOpenAI:Key"] 
    ?? throw new Exception("Key non configurata");
var deploymentName = configuration["AzureOpenAI:DeploymentName"] 
    ?? "gpt-4o";

// === SETUP LOGGING ===
using var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Information);
});
var logger = loggerFactory.CreateLogger<Program>();

logger.LogInformation("🚀 Avvio applicazione Azure OpenAI");
logger.LogInformation("Endpoint: {Endpoint}", endpoint);
logger.LogInformation("Deployment: {Deployment}", deploymentName);

// === CREA CLIENT AZURE OPENAI ===
var client = new OpenAIClient(
    new Uri(endpoint), 
    new AzureKeyCredential(key)
);

logger.LogInformation("✓ Client Azure OpenAI creato con successo");

// === PRIMA CHIAMATA ===
Console.WriteLine("\n🤖 Chat con Azure OpenAI");
Console.WriteLine("Scrivi 'exit' per uscire\n");

var chatMessages = new List<ChatRequestMessage>
{
    new ChatRequestSystemMessage("Sei un assistente tecnico esperto in C# e .NET")
};

while (true)
{
    // Input utente
    Console.Write("Tu: ");
    var userInput = Console.ReadLine();
    
    if (string.IsNullOrEmpty(userInput) || userInput.ToLower() == "exit")
        break;
    
    // Aggiungi messaggio utente
    chatMessages.Add(new ChatRequestUserMessage(userInput));
    
    try
    {
        logger.LogInformation("Invio richiesta ad Azure OpenAI...");
        
        // Chiamata API
        var response = await client.GetChatCompletionsAsync(
            new ChatCompletionsOptions(deploymentName, chatMessages)
            {
                Temperature = 0.7f,
                MaxTokens = 500,
                FrequencyPenalty = 0,
                PresencePenalty = 0
            }
        );
        
        var completion = response.Value;
        var assistantMessage = completion.Choices[0].Message.Content;
        
        // Aggiungi risposta alla cronologia
        chatMessages.Add(new ChatRequestAssistantMessage(assistantMessage));
        
        // Mostra risposta
        Console.WriteLine($"\nAI: {assistantMessage}\n");
        
        // Log usage
        logger.LogInformation(
            "Token usati - Prompt: {Prompt}, Completion: {Completion}, Totali: {Total}",
            completion.Usage.PromptTokens,
            completion.Usage.CompletionTokens,
            completion.Usage.TotalTokens
        );
    }
    catch (RequestFailedException ex)
    {
        logger.LogError(ex, "Errore nella chiamata API: {Message}", ex.Message);
        Console.WriteLine($"\n❌ Errore: {ex.Message}\n");
    }
}

logger.LogInformation("👋 Applicazione terminata");
```

### Step 2.3: Eseguire l'applicazione

```bash
dotnet run
```

**Test:**
- Scrivi: "Spiega cos'è LINQ in 2 frasi"
- Scrivi: "Dammi un esempio di delegate"
- Scrivi: "exit" per uscire

✅ **Checkpoint:** L'app risponde alle tue domande!

---

## Parte 3: Gestione Errori Avanzata (8 minuti)

### Step 3.1: Aggiungere gestione Content Filter

Modifica il blocco `catch` in `Program.cs`:

```csharp
catch (RequestFailedException ex)
{
    // Gestione specifica per Content Filter
    if (ex.Status == 400 && ex.ErrorCode == "content_filter")
    {
        logger.LogWarning(
            "⚠️ Contenuto bloccato dal filtro di sicurezza. Code: {Code}",
            ex.ErrorCode
        );
        
        Console.WriteLine("\n⚠️ La tua richiesta contiene contenuti che non posso elaborare.");
        Console.WriteLine("Prova a riformulare in modo diverso.\n");
    }
    // Gestione Rate Limiting (429)
    else if (ex.Status == 429)
    {
        logger.LogWarning("⚠️ Rate limit raggiunto. Attendi qualche secondo...");
        
        // Leggi Retry-After header se disponibile
        var retryAfter = 60; // Default 60 secondi
        if (ex.GetRawResponse().Headers.TryGetValue("Retry-After", out var value))
        {
            int.TryParse(value, out retryAfter);
        }
        
        Console.WriteLine($"\n⚠️ Troppo veloci! Attendi {retryAfter} secondi e riprova.\n");
    }
    // Altri errori 4xx
    else if (ex.Status >= 400 && ex.Status < 500)
    {
        logger.LogError(ex, "Errore client (4xx): {Status} - {Message}", 
            ex.Status, ex.Message);
        Console.WriteLine($"\n❌ Errore nella richiesta: {ex.Message}\n");
    }
    // Errori server 5xx
    else if (ex.Status >= 500)
    {
        logger.LogError(ex, "Errore server (5xx): {Status}", ex.Status);
        Console.WriteLine("\n❌ Errore temporaneo del servizio. Riprova tra poco.\n");
    }
    else
    {
        logger.LogError(ex, "Errore generico: {Message}", ex.Message);
        Console.WriteLine($"\n❌ Errore: {ex.Message}\n");
    }
}
```

### Step 3.2: Test gestione errori

**Test Content Filter** (dovrebbe bloccare):
```
Tu: Scrivi qualcosa di violento
```

Dovresti vedere il messaggio di warning invece dell'errore generico.

---

## Parte 4: Refactoring con Service Class (7 minuti)

Organizziamo meglio il codice creando un service dedicato.

### Step 4.1: Creare ChatService.cs

Crea un nuovo file `ChatService.cs`:

```csharp
using Azure;
using Azure.AI.OpenAI;
using Microsoft.Extensions.Logging;

namespace ChatApp;

public class ChatService
{
    private readonly OpenAIClient _client;
    private readonly string _deploymentName;
    private readonly ILogger<ChatService> _logger;
    private readonly List<ChatRequestMessage> _chatHistory;

    public ChatService(
        OpenAIClient client, 
        string deploymentName,
        ILogger<ChatService> logger)
    {
        _client = client;
        _deploymentName = deploymentName;
        _logger = logger;
        _chatHistory = new List<ChatRequestMessage>();
    }

    /// <summary>
    /// Imposta il system prompt per l'intera conversazione
    /// </summary>
    public void SetSystemPrompt(string systemPrompt)
    {
        _chatHistory.Clear();
        _chatHistory.Add(new ChatRequestSystemMessage(systemPrompt));
        _logger.LogInformation("System prompt impostato");
    }

    /// <summary>
    /// Invia un messaggio e ricevi la risposta
    /// </summary>
    public async Task<ChatResponse> SendMessageAsync(
        string userMessage,
        ChatOptions? options = null)
    {
        options ??= new ChatOptions();
        
        // Aggiungi messaggio utente
        _chatHistory.Add(new ChatRequestUserMessage(userMessage));
        
        try
        {
            _logger.LogInformation("Invio messaggio: {Preview}...", 
                userMessage.Substring(0, Math.Min(50, userMessage.Length)));
            
            var chatOptions = new ChatCompletionsOptions(_deploymentName, _chatHistory)
            {
                Temperature = options.Temperature,
                MaxTokens = options.MaxTokens,
                FrequencyPenalty = options.FrequencyPenalty,
                PresencePenalty = options.PresencePenalty
            };
            
            var response = await _client.GetChatCompletionsAsync(chatOptions);
            var completion = response.Value;
            
            var assistantMessage = completion.Choices[0].Message.Content;
            _chatHistory.Add(new ChatRequestAssistantMessage(assistantMessage));
            
            _logger.LogInformation(
                "Risposta ricevuta. Token: Prompt={Prompt}, Completion={Completion}, Total={Total}",
                completion.Usage.PromptTokens,
                completion.Usage.CompletionTokens,
                completion.Usage.TotalTokens
            );
            
            return new ChatResponse
            {
                Message = assistantMessage,
                IsSuccess = true,
                TokenUsage = new TokenUsage
                {
                    PromptTokens = completion.Usage.PromptTokens,
                    CompletionTokens = completion.Usage.CompletionTokens,
                    TotalTokens = completion.Usage.TotalTokens
                }
            };
        }
        catch (RequestFailedException ex) when (ex.Status == 400 && ex.ErrorCode == "content_filter")
        {
            _logger.LogWarning("Content filter triggered");
            
            // Rimuovi l'ultimo messaggio (quello problematico)
            _chatHistory.RemoveAt(_chatHistory.Count - 1);
            
            return new ChatResponse
            {
                IsSuccess = false,
                ErrorMessage = "La tua richiesta contiene contenuti che non posso elaborare. Prova a riformulare.",
                ErrorType = "ContentFilter"
            };
        }
        catch (RequestFailedException ex) when (ex.Status == 429)
        {
            _logger.LogWarning("Rate limit hit");
            
            return new ChatResponse
            {
                IsSuccess = false,
                ErrorMessage = "Troppe richieste. Attendi qualche secondo e riprova.",
                ErrorType = "RateLimit"
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Errore inatteso");
            
            // Rimuovi l'ultimo messaggio per mantenere consistenza
            _chatHistory.RemoveAt(_chatHistory.Count - 1);
            
            return new ChatResponse
            {
                IsSuccess = false,
                ErrorMessage = $"Errore: {ex.Message}",
                ErrorType = "Unknown"
            };
        }
    }

    /// <summary>
    /// Resetta la conversazione mantenendo il system prompt
    /// </summary>
    public void ResetConversation()
    {
        var systemPrompt = _chatHistory.FirstOrDefault(m => m is ChatRequestSystemMessage);
        _chatHistory.Clear();
        
        if (systemPrompt != null)
            _chatHistory.Add(systemPrompt);
        
        _logger.LogInformation("Conversazione resettata");
    }

    /// <summary>
    /// Ottieni il numero di messaggi nella cronologia
    /// </summary>
    public int GetMessageCount() => _chatHistory.Count;
}

// === MODELS ===

public class ChatOptions
{
    public float Temperature { get; set; } = 0.7f;
    public int MaxTokens { get; set; } = 500;
    public float FrequencyPenalty { get; set; } = 0;
    public float PresencePenalty { get; set; } = 0;
}

public class ChatResponse
{
    public bool IsSuccess { get; set; }
    public string? Message { get; set; }
    public string? ErrorMessage { get; set; }
    public string? ErrorType { get; set; }
    public TokenUsage? TokenUsage { get; set; }
}

public class TokenUsage
{
    public int PromptTokens { get; set; }
    public int CompletionTokens { get; set; }
    public int TotalTokens { get; set; }
    
    public decimal EstimatedCost(string model = "gpt-4o")
    {
        // Prezzi indicativi per GPT-4o (verifica docs per aggiornamenti)
        const decimal inputCostPer1K = 0.0025m;   // $0.0025 per 1K token
        const decimal outputCostPer1K = 0.010m;   // $0.010 per 1K token
        
        var inputCost = (PromptTokens / 1000m) * inputCostPer1K;
        var outputCost = (CompletionTokens / 1000m) * outputCostPer1K;
        
        return inputCost + outputCost;
    }
}
```

### Step 4.2: Aggiornare Program.cs

Ora usa il service:

```csharp
using Azure;
using Azure.AI.OpenAI;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;
using ChatApp;

// === CONFIGURAZIONE (come prima) ===
var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: true)
    .AddUserSecrets<Program>()
    .Build();

var endpoint = configuration["AzureOpenAI:Endpoint"]!;
var key = configuration["AzureOpenAI:Key"]!;
var deploymentName = configuration["AzureOpenAI:DeploymentName"] ?? "gpt-4o";

// === LOGGING ===
using var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Information);
});
var logger = loggerFactory.CreateLogger<Program>();

logger.LogInformation("🚀 Avvio Chat Application");

// === CREA CLIENT E SERVICE ===
var client = new OpenAIClient(new Uri(endpoint), new AzureKeyCredential(key));
var chatService = new ChatService(
    client, 
    deploymentName, 
    loggerFactory.CreateLogger<ChatService>()
);

// Imposta system prompt
chatService.SetSystemPrompt(
    "Sei un assistente tecnico esperto in C# e .NET. " +
    "Rispondi in modo chiaro e conciso con esempi pratici quando appropriato."
);

// === LOOP CONVERSAZIONE ===
Console.WriteLine("\n🤖 Chat con Azure OpenAI (.NET SDK)");
Console.WriteLine("Comandi: 'exit' = esci, 'reset' = nuova conversazione, 'count' = messaggi\n");

decimal totalCost = 0;

while (true)
{
    Console.Write("Tu: ");
    var input = Console.ReadLine();
    
    if (string.IsNullOrEmpty(input)) continue;
    
    // Comandi speciali
    if (input.ToLower() == "exit") break;
    
    if (input.ToLower() == "reset")
    {
        chatService.ResetConversation();
        totalCost = 0;
        Console.WriteLine("\n✓ Conversazione resettata\n");
        continue;
    }
    
    if (input.ToLower() == "count")
    {
        Console.WriteLine($"\n📊 Messaggi in cronologia: {chatService.GetMessageCount()}");
        Console.WriteLine($"💰 Costo totale stimato: ${totalCost:F4}\n");
        continue;
    }
    
    // Invia messaggio
    var response = await chatService.SendMessageAsync(input);
    
    if (response.IsSuccess)
    {
        Console.WriteLine($"\nAI: {response.Message}\n");
        
        // Calcola costo
        if (response.TokenUsage != null)
        {
            var cost = response.TokenUsage.EstimatedCost();
            totalCost += cost;
            
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine($"[Token: {response.TokenUsage.TotalTokens} | Costo: ${cost:F4}]");
            Console.ResetColor();
            Console.WriteLine();
        }
    }
    else
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine($"\n❌ {response.ErrorMessage}\n");
        Console.ResetColor();
    }
}

Console.WriteLine($"\n💰 Costo totale sessione: ${totalCost:F4}");
logger.LogInformation("👋 Applicazione terminata");
```

### Step 4.3: Test dell'applicazione refactorata

```bash
dotnet run
```

**Test:**
1. Chatta normalmente
2. Scrivi `count` per vedere stats
3. Scrivi `reset` per nuova conversazione
4. Scrivi `exit` per uscire

✅ **Checkpoint:** L'app mostra token usage e costo stimato!

---

## Parte 5: Testing Avanzato (opzionale, se tempo)

### Test 1: Temperature Variations

Crea un piccolo script per testare temperature diverse:

```csharp
// Aggiungi dopo la creazione del chatService in Program.cs

Console.WriteLine("🧪 Test Temperature\n");

var prompt = "Dai 3 nomi creativi per una startup di AI";

foreach (var temp in new[] { 0.1f, 0.7f, 1.5f })
{
    Console.WriteLine($"\n--- Temperature: {temp} ---");
    
    var options = new ChatOptions { Temperature = temp, MaxTokens = 100 };
    var response = await chatService.SendMessageAsync(prompt, options);
    
    if (response.IsSuccess)
    {
        Console.WriteLine(response.Message);
    }
    
    chatService.ResetConversation();
}
```

Nota come con temperature bassa (0.1) le risposte sono simili, con alta (1.5) sono molto diverse.

### Test 2: Max Tokens Limiting

```csharp
// Test troncamento
var options = new ChatOptions { MaxTokens = 20 };  // Molto basso
var response = await chatService.SendMessageAsync(
    "Spiega dettagliatamente cos'è un'API REST", 
    options
);

// La risposta sarà troncata
```

---

## 📊 Riepilogo e Best Practices

### ✅ Cosa Hai Imparato

- ✓ Setup progetto .NET con Azure OpenAI SDK
- ✓ Gestione sicura delle credenziali (User Secrets)
- ✓ Chiamate API con gestione errori robusta
- ✓ Chat conversazionali con cronologia
- ✓ Calcolo costi e monitoraggio token usage
- ✓ Architettura pulita con service class

### 🎯 Best Practices da Ricordare

**1. Sicurezza:**
```csharp
// ❌ MAI fare questo
var key = "sk-xxx...";  // Hardcoded

// ✅ Sempre così
var key = configuration["AzureOpenAI:Key"];  // Da config/secrets
```

**2. Gestione Errori:**
```csharp
// ❌ Catch generico
catch (Exception ex) { /* ... */ }

// ✅ Gestione specifica
catch (RequestFailedException ex) when (ex.Status == 400 && ex.ErrorCode == "content_filter")
{
    // Handle content filter
}
catch (RequestFailedException ex) when (ex.Status == 429)
{
    // Handle rate limit
}
```

**3. Logging:**
```csharp
// ✅ Sempre logga
_logger.LogInformation("Request sent");
_logger.LogError(ex, "API error: {Message}", ex.Message);

// ❌ Non loggare contenuti sensibili
_logger.LogInformation("User {Email} asked: {Prompt}", email, prompt);
```

**4. Gestione Context:**
```csharp
// Implementa limite messaggi per evitare context explosion
if (_chatHistory.Count > 20)
{
    // Mantieni solo system + ultimi 10 messaggi
    var systemMsg = _chatHistory.First();
    var recentMsgs = _chatHistory.TakeLast(10).ToList();
    
    _chatHistory.Clear();
    _chatHistory.Add(systemMsg);
    _chatHistory.AddRange(recentMsgs);
}
```

**5. Costi:**
```csharp
// Monitora sempre token usage
if (response.TokenUsage.TotalTokens > 1000)
{
    _logger.LogWarning("High token usage: {Tokens}", response.TokenUsage.TotalTokens);
}

// Imposta max_tokens ragionevoli
var options = new ChatOptions 
{ 
    MaxTokens = 500  // Non 2000+ se non necessario
};
```

---

## 🚀 Prossimi Passi (oltre questo lab)

### Funzionalità da Aggiungere

1. **Retry Logic con Polly**
   ```bash
   dotnet add package Polly
   ```
   Implementa exponential backoff per errori temporanei

2. **Application Insights Integration**
   ```bash
   dotnet add package Microsoft.ApplicationInsights.AspNetCore
   ```
   Telemetria dettagliata in produzione

3. **Structured Logging con Serilog**
   ```bash
   dotnet add package Serilog.Extensions.Hosting
   dotnet add package Serilog.Sinks.Console
   ```
   Logging strutturato per analytics

4. **Caching delle Risposte**
   ```csharp
   // Per richieste ripetitive
   var cacheKey = $"chat:{Hash(userMessage)}";
   if (_cache.TryGetValue(cacheKey, out var cachedResponse))
       return cachedResponse;
   ```

5. **Streaming Responses**
   ```csharp
   // Per UI più reattive
   await foreach (var message in client.GetChatCompletionsStreamingAsync(...))
   {
       Console.Write(message.ContentUpdate);
   }
   ```

---

## 📚 Risorse Aggiuntive

### Documentazione Microsoft

- [Azure OpenAI .NET SDK](https://learn.microsoft.com/dotnet/api/azure.ai.openai)
- [Quickstart C#](https://learn.microsoft.com/azure/ai-services/openai/quickstart?pivots=programming-language-csharp)
- [Best Practices](https://learn.microsoft.com/azure/ai-services/openai/concepts/best-practices)

### GitHub Samples

- [Azure OpenAI Samples](https://github.com/Azure-Samples/openai)
- [.NET SDK Examples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/openai/Azure.AI.OpenAI)

### Tools

- [.NET CLI Documentation](https://learn.microsoft.com/dotnet/core/tools/)
- [User Secrets Guide](https://learn.microsoft.com/aspnet/core/security/app-secrets)

---

## 🆘 Troubleshooting

### Problema: "Endpoint non configurato"

**Causa:** User secrets non impostati

**Soluzione:**
```bash
dotnet user-secrets list  # Verifica secrets
dotnet user-secrets set "AzureOpenAI:Endpoint" "https://..."
```

### Problema: "401 Unauthorized"

**Causa:** API key errata

**Soluzione:**
- Verifica key in Azure Portal
- Rigenera se necessario
- Aggiorna user secret

### Problema: "Package Azure.AI.OpenAI non trovato"

**Causa:** NuGet cache o versione errata

**Soluzione:**
```bash
dotnet nuget locals all --clear
dotnet restore
```

### Problema: Rate limit continui (429)

**Causa:** TPM troppo basso per il carico

**Soluzione:**
- Aumenta TPM del deployment in Azure
- Implementa exponential backoff
- Riduci frequenza richieste

---

## ✅ Checklist Finale

Prima di procedere alla sessione successiva:

- [ ] Ho creato un progetto .NET funzionante
- [ ] Ho configurato User Secrets correttamente
- [ ] Ho testato chat conversazionale
- [ ] Ho visto la gestione errori in azione
- [ ] Ho verificato token usage e costi
- [ ] Ho refactorato in un service class pulito
- [ ] Capisco come gestire content filter
- [ ] So come calcolare costi stimati

---

**Fine LAB 2** ✨

Nella prossima sessione vedremo Semantic Kernel e Function Calling!
