# Esercizio sulla dependency injection (DI) in C# .NET 8:

**Esercizio: Simulazione di un sistema di prenotazione**

**Descrizione**

Creerai una piccola applicazione console che simula un sistema di prenotazione di biglietti. Implementerai la dependency injection manualmente per gestire le dipendenze tra classi.

**Passaggi**

1.  **Definisci un'interfaccia per il servizio di prenotazione**
    * Crea un'interfaccia `ITicketService` che definisce un metodo per prenotare un biglietto.
2.  **Implementa il servizio**
    * Crea una classe concreta `TicketService` che implementa `ITicketService` e simula la prenotazione.
3.  **Crea un container DI manuale**
    * Implementa una classe `ServiceContainer` che funge da registro per le dipendenze e risolve le istanze.
4.  **Usa la DI nel programma principale**
    * Configura e usa il container nel `Program.cs` per iniettare il servizio in una classe consumatrice.

**Codice:**

**Soluzione**

1.  **Interfaccia `ITicketService`**

    ```csharp
    public interface ITicketService{
        void BookTicket(string userName, string eventName);
    }
    ```

2.  **Implementazione `TicketService`**

    ```csharp
    public class TicketService : ITicketService{
        public void BookTicket(string userName, string eventName) {
            Console.WriteLine($"Biglietto prenotato per {userName} all'evento '{eventName}'");
        }
    }
    ```

3.  **Container DI manuale `ServiceContainer`**

    ```csharp
    public class ServiceContainer {
        private readonly Dictionary<Type, Func<object>> _services;
        public ServiceContainer() {
            _services = new Dictionary<Type, Func<object>>();
        }
        // Registra un servizio con la sua implementazione
        public void Register<TInterface, TImplementation>() where TImplementation : TInterface, new() {
            _services[typeof(TInterface)] = () => new TImplementation();
        }
        // Risolvi un'istanza del servizio
        public T GetService<T>() {
            if (_services.TryGetValue(typeof(T), out var factory)) {
                return (T)factory();
            }
            throw new InvalidOperationException($"Nessun servizio registrato per il tipo {typeof(T).Name}");
        }
    }
    ```

4.  **Classe consumatrice `BookingManager`**

    ```csharp
    public class BookingManager{
        private readonly ITicketService _ticketService;
        // Il servizio viene iniettato tramite il costruttore
        public BookingManager(ITicketService ticketService) {
            _ticketService = ticketService ?? throw new ArgumentNullException(nameof(ticketService));
        }
        public void ProcessBooking(string userName, string eventName) {
            _ticketService.BookTicket(userName, eventName);
        }
    }
    ```

5.  **Programma principale `Program.cs`**

    ```csharp
    class Program{
        static void Main(string[] args) {
            // Crea il container DI
            var container = new ServiceContainer();
            // Registra il servizio
            container.Register<ITicketService, TicketService>();
            // Risolvi il servizio e crea il BookingManager
            var ticketService = container.GetService<ITicketService>();
            var bookingManager = new BookingManager(ticketService);
            // Usa il sistema
            bookingManager.ProcessBooking("Mario Rossi", "Concerto di primavera");
        }
    }
    ```

**Spiegazione**

* **Interfaccia e Implementazione**: `ITicketService` e `TicketService` definiscono il contratto e la logica del servizio.
* **Container DI**: `ServiceContainer` è una semplice implementazione manuale che:
    * Registra i tipi con `Register`.
    * Rende disponibili le istanze con `GetService`.
* **Iniezione tramite costruttore**: `BookingManager` riceve il servizio tramite il costruttore, rispettando il principio di inversione delle dipendenze.
* **Flessibilità**: Puoi facilmente sostituire `TicketService` con un'altra implementazione (es. per test) modificando solo la registrazione.

**Sfida aggiuntiva**

* Aggiungi un secondo servizio, ad esempio `IEmailService`, per inviare una conferma via email dopo la prenotazione.
* Modifica `BookingManager` per dipendere sia da `ITicketService` che da `IEmailService`.
* Aggiorna il container per supportare più dipendenze.

