# Esercizio per approfondire la Dependency Injection (DI) in C# .NET 8, questa volta con un focus sulla gestione di configurazioni e logging:

**Esercizio: Sistema di Notifiche Personalizzabili**

**Descrizione**

Creerai un'applicazione console che invia notifiche agli utenti. L'obiettivo è implementare la DI per gestire le diverse modalità di notifica (email, SMS) e le configurazioni personalizzate.

**Passaggi**

1.  **Definisci le interfacce per i servizi di notifica**
    * Crea un'interfaccia `INotificationService` con un metodo `SendNotification(string message, string recipient)`.
    * Crea un'interfaccia `IConfigurationService` per gestire le configurazioni.
    * Crea un'interfaccia `ILoggingService` per il logging.
2.  **Implementa i servizi di notifica**
    * Crea classi `EmailNotificationService` e `SmsNotificationService` che implementano `INotificationService`.
3.  **Implementa i servizi di configurazione e logging**
    * Crea una classe `ConfigurationService` che implementa `IConfigurationService` e legge le configurazioni da un file JSON (o da un dizionario in memoria per semplicità).
    * Crea una classe `ConsoleLoggingService` che implementa `ILoggingService` e scrive i log sulla console.
4.  **Crea un container DI manuale**
    * Implementa una classe `ServiceContainer` come nell'esercizio precedente.
5.  **Usa la DI nel programma principale**
    * Configura e usa il container nel `Program.cs` per iniettare i servizi in una classe `NotificationManager`.

**Codice:**

**Soluzione**

1.  **Interfacce**

    ```csharp
    public interface INotificationService {
        void SendNotification(string message, string recipient);
    }

    public interface IConfigurationService {
        string GetSetting(string key);
    }

    public interface ILoggingService {
        void Log(string message);
    }
    ```

2.  **Implementazioni dei servizi di notifica**

    ```csharp
    public class EmailNotificationService : INotificationService {
        private readonly ILoggingService _logger;
        private readonly IConfigurationService _config;

        public EmailNotificationService(ILoggingService logger, IConfigurationService config) {
            _logger = logger;
            _config = config;
        }

        public void SendNotification(string message, string recipient) {
            _logger.Log($"Invio email a {recipient}: {message}");
            Console.WriteLine($"Simulazione invio email a {recipient}: {message}");
            // Logica di invio email reale
        }
    }

    public class SmsNotificationService : INotificationService {
          private readonly ILoggingService _logger;

        public SmsNotificationService(ILoggingService logger) {
            _logger = logger;
        }
        public void SendNotification(string message, string recipient) {
           _logger.Log($"Invio SMS a {recipient}: {message}");
            Console.WriteLine($"Simulazione invio SMS a {recipient}: {message}");
            // Logica di invio SMS reale
        }
    }
    ```

3.  **Implementazioni dei servizi di configurazione e logging**

    ```csharp
    public class ConfigurationService : IConfigurationService {
        private readonly Dictionary<string, string> _settings;

        public ConfigurationService() {
            // Semplice configurazione in memoria
            _settings = new Dictionary<string, string> {
                { "EmailSender", "no-reply@example.com" },
                { "SmsGateway", "https://api.smsgateway.com" }
            };
        }

        public string GetSetting(string key) {
            return _settings.TryGetValue(key, out var value) ? value : null;
        }
    }

    public class ConsoleLoggingService : ILoggingService {
        public void Log(string message) {
            Console.WriteLine($"[LOG] {DateTime.Now}: {message}");
        }
    }
    ```

4.  **Classe `NotificationManager`**

    ```csharp
    public class NotificationManager {
        private readonly INotificationService _notificationService;

        public NotificationManager(INotificationService notificationService) {
            _notificationService = notificationService;
        }

        public void Send(string message, string recipient) {
            _notificationService.SendNotification(message, recipient);
        }
    }
    ```

5.  **Programma principale `Program.cs`**

    ```csharp
    class Program {
        static void Main(string[] args) {
            var container = new ServiceContainer();
            container.Register<ILoggingService, ConsoleLoggingService>();
            container.Register<IConfigurationService, ConfigurationService>();
            container.Register<INotificationService, EmailNotificationService>();

            var notificationManager = new NotificationManager(container.GetService<INotificationService>());
            notificationManager.Send("Ciao, questo è un test!", "utente@example.com");
        }
    }
    ```

**Sfide aggiuntive**

* Aggiungi un'implementazione di `INotificationService` per le notifiche push.
* Implementa la lettura delle configurazioni da un file JSON.
* Aggiungi la gestione degli errori e il logging dettagliato.
* Crea una classe astratta base per i servizi di notifica.
