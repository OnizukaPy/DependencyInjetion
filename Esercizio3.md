# La Factory Pattern in C# .NET

## Cos'è una Factory in C# .NET 8?

Una **Factory** (in italiano "fabbrica") è un *pattern di design creazionale* utilizzato nella programmazione orientata agli oggetti (OOP). Serve per **incapsulare la logica di creazione di un oggetto**, delegando la responsabilità di istanziare classi a un'entità separata invece di farlo direttamente nel codice che usa l'oggetto. In C#, una Factory può essere implementata come una classe, un metodo o una funzione (spesso una *lambda*), e in .NET 8 può sfruttare funzionalità moderne come le espressioni lambda e i *generics*.

## Definizione teorica

- **Pattern Creazionale**: Appartiene alla categoria dei pattern che si occupano di come creare oggetti.
- **Scopo**: Nascondere i dettagli di costruzione di un oggetto (es. quale classe concreta usare, quali parametri passare) e fornire un'interfaccia semplice per ottenerlo.
- **Vantaggi**:
  - Disaccoppia il codice che usa un oggetto da come viene creato.
  - Facilita la sostituzione di implementazioni senza modificare il codice chiamante.
  - Supporta la *Dependency Injection* (DI) fornendo un modo per creare oggetti in modo controllato.


In C#, una Factory può essere:

1. **Un metodo statico**: Un metodo in una classe che restituisce un'istanza.
2. **Una classe dedicata**: Una classe il cui unico scopo è creare oggetti.
3. **Una funzione/delegato**: Usando `Func<T>` o *lambda expressions*, molto comune nei container DI.

In .NET 8, grazie al supporto avanzato per i tipi generici e le espressioni lambda, le Factory sono spesso implementate in modo conciso e funzionale, specialmente in contesti come la DI.

## Esempio pratico

Supponiamo di voler creare oggetti di tipo `ILogger` (un'interfaccia per logging). Senza una Factory, potresti fare così:

```csharp
ILogger logger = new ConsoleLogger(); // Creazione diretta
logger.Log("Ciao!");
```

Ma questo lega il codice direttamente a `ConsoleLogger`. Con una `Factory`, **puoi astrarre la creazione**:

1. Factory come metodo statico

```csharp
// Definizione di un'interfaccia ILogger
// Teoria: Un'interfaccia è un contratto astratto che specifica i metodi che una classe deve implementare.
// Serve a garantire che tutte le implementazioni (es. ConsoleLogger, FileLogger) abbiano un comportamento uniforme.
// Definizione: Interfaccia è un tipo che definisce solo firme di metodi, proprietà o eventi, senza implementazione.
public interface ILogger {
    // Metodo astratto per registrare un messaggio
    // Teoria: Questo metodo sarà implementato diversamente da ogni classe che eredita l'interfaccia,
    // permettendo la sostituzione delle implementazioni senza modificare il codice chiamante.
    // Definizione: Metodo astratto è una firma senza corpo, che deve essere implementata dalle classi derivate.
    void Log(string message);
}

// Implementazione concreta di ILogger per il logging su console
// Teoria: ConsoleLogger è una classe concreta che fornisce un'implementazione specifica dell'interfaccia ILogger.
// Usa la console come destinazione dei log, rispettando il contratto definito dall'interfaccia.
// Definizione: Classe concreta è una classe che può essere istanziata direttamente, a differenza di un'interfaccia o classe astratta.
public class ConsoleLogger : ILogger {
    // Implementazione del metodo Log per scrivere sulla console
    // Teoria: Usa Console.WriteLine per output visivo immediato. La sintassi "=>" è un'espressione lambda
    // che rende il metodo più conciso, introdotta in C# 6.0 e supportata in .NET 8.
    // Definizione: Espressione lambda è una sintassi compatta per definire funzioni anonime, qui usata come corpo del metodo.
    public void Log(string message) => Console.WriteLine($"[Console] {message}");
}

// Implementazione concreta di ILogger per il logging su file (simulato)
// Teoria: FileLogger rappresenta un'altra variante di ILogger. Qui simula il logging su file scrivendo comunque
// sulla console per semplicità, ma in un caso reale userebbe operazioni di I/O su filesystem.
// Definizione: Simulazione è l'uso di un comportamento fittizio per rappresentare un'operazione reale senza implementarla completamente.
public class FileLogger : ILogger {
    // Implementazione del metodo Log per un'ipotetica scrittura su file
    // Teoria: Anche qui si usa la sintassi lambda per brevità. Il prefisso "[File]" distingue l'output da ConsoleLogger.
    // Nota: In un'applicazione reale, questo metodo potrebbe usare System.IO.File per scrivere su disco.
    public void Log(string message) => Console.WriteLine($"[File] {message}"); // Simulazione
}

// Classe Factory per creare istanze di ILogger
// Teoria: LoggerFactory è un esempio di "Factory Method Pattern", un pattern creazionale che incapsula
// la logica di creazione degli oggetti, permettendo di scegliere l'implementazione in base a un parametro.
// Definizione: Factory Method è un pattern che definisce un metodo per creare oggetti, delegando la scelta della classe concreta a una sottoclasse o logica interna.
// Definizione: Classe statica è una classe che non può essere istanziata e contiene solo membri statici, utile per utility o factory.
public static class LoggerFactory {
    // Metodo statico per creare un'istanza di ILogger
    // Teoria: Questo metodo usa un parametro "type" per decidere quale implementazione di ILogger restituire.
    // Usa l'espressione switch (introdotta in C# 8.0) per una sintassi moderna e leggibile.
    // Definizione: Espressione switch è una forma concisa di switch statement che restituisce direttamente un valore.
    public static ILogger CreateLogger(string type) {
        return type switch {
            // Caso "console": restituisce un'istanza di ConsoleLogger
            // Teoria: Qui la Factory sceglie ConsoleLogger, nascondendo la creazione al chiamante.
            "console" => new ConsoleLogger(),
            // Caso "file": restituisce un'istanza di FileLogger
            // Teoria: La Factory può facilmente aggiungere nuove implementazioni senza cambiare il codice chiamante.
            "file" => new FileLogger(),
            // Caso predefinito: lancia un'eccezione per tipi non validi
            // Teoria: Il caso "_" (wildcard) gestisce input non previsti, migliorando la robustezza del codice.
            // Definizione: Eccezione è un errore gestito che interrompe il flusso normale del programma, qui usata per segnalare un input invalido.
            _ => throw new ArgumentException("Tipo non supportato")
        };
    }
}

// Classe principale del programma
// Teoria: Program è il punto di ingresso dell'applicazione console in .NET. Qui dimostra l'uso pratico della Factory.
// Definizione: Punto di ingresso è il metodo (Main) che il runtime chiama per avviare l'esecuzione di un programma.
class Program {
    // Metodo Main: entry point dell'applicazione
    // Teoria: Main è statico perché viene chiamato dal runtime senza bisogno di istanziare Program.
    // Definizione: Metodo statico è un metodo che appartiene alla classe, non a un'istanza, e può essere chiamato senza creare un oggetto.
    static void Main() {
        // Creazione di un logger usando la Factory
        // Teoria: La Factory restituisce un'implementazione di ILogger (ConsoleLogger) senza che il codice
        // debba conoscere i dettagli di costruzione, rispettando il principio di disaccoppiamento.
        // Definizione: Disaccoppiamento è la separazione delle dipendenze tra componenti per rendere il codice più modulare e manutenibile.
        ILogger logger = LoggerFactory.CreateLogger("console");
        // Chiamata al metodo Log sul logger creato
        logger.Log("Ciao da console!");

        // Creazione di un secondo logger con un tipo diverso
        // Teoria: La stessa Factory viene riutilizzata per creare un'implementazione diversa (FileLogger),
        // dimostrando la flessibilità del pattern.
        ILogger fileLogger = LoggerFactory.CreateLogger("file");
        // Chiamata al metodo Log sul secondo logger
        fileLogger.Log("Ciao da file!");
    }
}
```

Risultato atteso

```bash
[Console] Ciao da console!
[File] Ciao da file!
```

**Spiegazione**:

`LoggerFactory.CreateLogger` decide quale implementazione di ILogger creare in base a un parametro.

Il codice chiamante (Main) non sa quale classe concreta viene usata, migliorando il disaccoppiamento.


2. Factory come delegato in DI (con .NET 8)

In un contesto di *Dependency Injection*, una Factory può essere una `Func<T>` (un delegato che restituisce un oggetto):

```csharp
// Container per la Dependency Injection (DI)
// Teoria: ServiceContainer è un esempio di "IoC Container" (Inversion of Control Container), un componente che gestisce
// la registrazione e la risoluzione delle dipendenze in un'applicazione. Implementa il pattern DI per disaccoppiare
// la creazione degli oggetti dal loro utilizzo.
// Definizione: Dependency Injection (DI) è un pattern di design che consente di fornire (o "iniettare") le dipendenze
// di una classe dall'esterno, invece di crearle internamente.
// Definizione: IoC (Inversion of Control) è un principio in cui il controllo della gestione degli oggetti viene invertito,
// delegandolo a un container o framework.
public class ServiceContainer {
    // Campo privato per memorizzare i servizi registrati
    // Teoria: Usa un Dictionary per associare tipi (es. ILogger) a factory che creano istanze. Il modificatore
    // "readonly" garantisce che il dizionario non venga riassegnato dopo l'inizializzazione, migliorando la sicurezza.
    // Definizione: Dictionary è una struttura dati che associa chiavi (Type) a valori (Func<object>) con accesso rapido O(1).
    // Definizione: Func<object> è un delegato generico che rappresenta una funzione senza parametri che restituisce un object.
    private readonly Dictionary<Type, Func<object>> _services = new();

    // Metodo per registrare un servizio con una factory
    // Teoria: Questo metodo permette di associare un tipo generico T a una factory che crea l'istanza corrispondente.
    // Usa un approccio "transient" implicito: ogni risoluzione crea una nuova istanza (a meno che la factory non restituisca un singleton).
    // Definizione: Generico (T) è un placeholder per un tipo, che rende il metodo riutilizzabile per qualsiasi classe o interfaccia.
    // Definizione: Factory è un'entità (qui una Func<T>) responsabile della creazione di oggetti, incapsulandone la logica.
    public void Register<T>(Func<T> factory) {
        // Registra la factory nel dizionario
        // Teoria: Converte la Func<T> in una Func<object> (usando una lambda) per uniformità nel dizionario.
        // La lambda "() => factory()" garantisce che la factory venga invocata solo al momento della risoluzione.
        // Definizione: Lambda è una funzione anonima definita inline, qui usata per creare un livello di indirezione.
        _services[typeof(T)] = () => factory();
    }

    // Metodo per risolvere un servizio
    // Teoria: La risoluzione è il processo di recuperare un'istanza di un servizio dal container. Questo metodo
    // cerca la factory associata al tipo T e la esegue per ottenere l'istanza.
    // Definizione: Risoluzione è l'atto di ottenere un oggetto dal container DI basato sul suo tipo registrato.
    public T Resolve<T>() {
        // Esegue la factory e restituisce l'istanza castata al tipo T
        // Teoria: Usa typeof(T) per cercare nel dizionario e invoca la factory con (). Il cast (T) è necessario
        // perché il dizionario restituisce un object.
        // Definizione: Cast è la conversione esplicita di un tipo (object) in un altro (T), richiesta dal compilatore.
        // Nota: Questo codice assume che T sia registrato, altrimenti solleverà un'eccezione runtime (KeyNotFoundException).
        return (T)_services[typeof(T)]();
    }
}

// Classe principale del programma
// Teoria: Program è il punto di ingresso dell'applicazione console in .NET. Qui dimostra come usare il container
// per registrare e risolvere dipendenze con una factory.
// Definizione: Punto di ingresso è il metodo (Main) che il runtime .NET chiama per avviare l'esecuzione del programma.
class Program {
    // Metodo Main: entry point dell'applicazione
    // Teoria: Main è statico perché viene invocato direttamente dal runtime senza istanziare la classe Program.
    // Definizione: Metodo statico è un metodo associato alla classe stessa, non a un'istanza, accessibile senza creare un oggetto.
    static void Main() {
        // Creazione di un'istanza del container
        // Teoria: Il container viene inizializzato per gestire le dipendenze dell'applicazione. È il "cuore" del sistema DI.
        var container = new ServiceContainer();

        // Registrazione di ILogger con una factory
        // Teoria: Qui si usa una lambda come factory per creare un'istanza di ConsoleLogger. La factory viene passata
        // al container, che la userà per generare l'istanza quando richiesto.
        // Definizione: ConsoleLogger è una classe concreta che implementa ILogger (non mostrata qui, ma assunta esistente).
        // Nota: La lambda "() => new ConsoleLogger()" è una factory che crea una nuova istanza ogni volta che viene invocata.
        container.Register<ILogger>(() => new ConsoleLogger()); // Factory come lambda

        // Risoluzione del servizio ILogger
        // Teoria: Il container restituisce un'istanza di ILogger creata dalla factory registrata. Il tipo ILogger
        // nasconde l'implementazione concreta (ConsoleLogger), rispettando il principio di disaccoppiamento.
        // Definizione: Disaccoppiamento è la separazione delle dipendenze tra componenti, rendendo il codice più flessibile e testabile.
        var logger = container.Resolve<ILogger>();

        // Uso del servizio risolto
        // Teoria: Una volta risolto, logger può essere usato senza sapere come è stato creato, dimostrando il potere della DI.
        // Nota: Si assume che ILogger abbia un metodo Log (non mostrato qui, ma implicito dall'esempio).
        logger.Log("Ciao da DI!");
    }
}
```

**Spiegazione**:

* La Factory è una `lambda (() => new ConsoleLogger())` passata al `container`.
* Il container usa la Factory per creare l'istanza solo quando necessario.
* In .NET 8, questo approccio è efficiente grazie all'ottimizzazione delle lambda e dei delegate.

## Perché usare una Factory?

* **Flessibilità:** Puoi cambiare l'implementazione (es. da `ConsoleLogger` a `FileLogger`) senza toccare il codice che usa `ILogger`.
* **Testabilità:** Nei test, puoi fornire una Factory che restituisce un mock.
* **Controllo:** La Factory può gestire parametri complessi o logica condizionale per la creazione.
* **Integrazione con DI:** È un pilastro per i container DI, come quello di Microsoft o il nostro esempio manuale.

## Factory in .NET 8: cosa c'è di nuovo?

Anche se il concetto di Factory è senza tempo, in .NET 8 puoi sfruttare:

* **Performance:** Le lambda e i delegate sono ottimizzati dal runtime.
* **Generics:** Puoi creare Factory fortemente tipizzate (`Func<T>`).
* **Minimal API:** Se lavori con ASP.NET Core, le Factory si integrano bene con la DI nativa, ma anche senza Microsoft puoi usarle come sopra.

## Conclusione

In C# con .NET 8, una Factory è un modo elegante per creare oggetti in modo controllato e disaccoppiato. Può essere semplice (un metodo statico) o sofisticata (una lambda in un container DI). Nel tuo caso, nel codice precedente, la Factory (`Func<T>`) è usata per registrare servizi transient o singleton, permettendo al container di generare istanze senza che il codice chiamante debba conoscerne i dettagli.

Se vuoi un esempio più specifico o un approfondimento su un aspetto (es. Factory Method vs Abstract Factory), fammi sapere!


# Esercizio con i service

Come implementare un service per la dependency injection (DI) in C# con .NET 8 senza utilizzare le librerie Microsoft preconfezionate, come `Microsoft.Extensions.DependencyInjection`. In questo caso, possiamo creare un sistema di DI manuale, semplice ma funzionale, usando solo le funzionalità base di C# e .NET 8. Ti guiderò passo passo per costruire un container di dependency injection personalizzato.

## Obiettivo

Creare un sistema di dependency injection senza dipendere da librerie Microsoft, permettendo la registrazione e la risoluzione di servizi.

## Passaggi per implementare un DI manuale

### 1. Struttura del progetto

Inizia creando un nuovo progetto console:

```bash
dotnet new console -n CustomDI
cd CustomDI
```

### 2. Implementazione del container DI

Ecco un esempio di codice che implementa un container di dependency injection basilare:

```csharp
using System;
using System.Collections.Generic;

// Container personalizzato per la Dependency Injection (DI)
// Teoria: La DI è un pattern di design che consente di invertire il controllo della creazione degli oggetti.
// Invece di creare direttamente le dipendenze all'interno di una classe, queste vengono "iniettate" dall'esterno.
// Il ServiceContainer funge da "IoC Container" (Inversion of Control Container), gestendo la registrazione e la risoluzione delle dipendenze.
// Definizione: IoC (Inversion of Control) è un principio in cui il controllo del flusso di un programma viene delegato a un'entità esterna (qui il container).
public class ServiceContainer {
    // Dizionario che associa tipi (interfacce o classi) a funzioni factory per creare istanze
    // Teoria: Usare un dizionario permette una lookup efficiente O(1) per risolvere i servizi.
    // La chiave è il tipo (es. ILogger), il valore è una funzione che produce l'istanza.
    // Definizione: Factory è un oggetto o una funzione che crea altre istanze di oggetti, incapsulando la logica di costruzione.
    // Definizione: Lookup si riferisce alla ricerca di un elemento in una struttura dati (qui il dizionario) usando una chiave.
    private readonly Dictionary<Type, Func<object>> _services;

    // Costruttore del container
    // Teoria: Inizializza il container come un oggetto immutabile per quanto riguarda la struttura interna,
    // garantendo che il dizionario sia pronto per la registrazione dei servizi.
    // Definizione: Costruttore è un metodo speciale di una classe che inizializza un'istanza quando viene creata.
    public ServiceContainer() {
        _services = new Dictionary<Type, Func<object>>();
    }

    // Registra un servizio come singleton
    // Teoria: Un singleton è un tipo di servizio di cui esiste una sola istanza condivisa nell'applicazione.
    // Questo metodo memorizza un'istanza esistente, che verrà restituita ogni volta che il servizio viene risolto.
    // Definizione: Singleton è un pattern di design che garantisce che una classe abbia una sola istanza e fornisce un punto di accesso globale a essa.
    public void RegisterSingleton<TService>(TService instance) {
        // La funzione factory restituisce sempre la stessa istanza (singleton)
        _services[typeof(TService)] = () => instance;
    }

    // Registra un servizio come transient
    // Teoria: Un servizio transient crea una nuova istanza ogni volta che viene richiesto.
    // Questo è utile per oggetti leggeri o che devono essere isolati per ogni utilizzo.
    // Definizione: Transient si riferisce a un lifetime in cui ogni richiesta genera una nuova istanza, opposto al singleton.
    public void RegisterTransient<TService>(Func<TService> factory) {
        // La factory viene chiamata ogni volta, generando una nuova istanza
        _services[typeof(TService)] = () => factory();
    }

    // Risoluzione di un servizio
    // Teoria: La risoluzione è il processo di ottenere un'istanza di un servizio dal container.
    // Questo metodo cerca il tipo nel dizionario e usa la factory associata per creare o restituire l'istanza.
    // Definizione: Risoluzione è l'atto di recuperare un'istanza di un servizio registrata in un container DI.
    public TService Resolve<TService>() {
        // Controlla se il servizio è stato registrato; altrimenti lancia un'eccezione
        if (!_services.TryGetValue(typeof(TService), out var factory)) {
            throw new InvalidOperationException($"Servizio {typeof(TService).Name} non registrato.");
        }
        // Esegue la factory e restituisce l'istanza castata al tipo richiesto
        // Definizione: Cast è la conversione esplicita di un tipo di dato in un altro (qui da object a TService).
        return (TService)factory();
    }

    // Risoluzione di un servizio con dipendenze
    // Teoria: Questo metodo implementa la "constructor injection", un tipo di DI in cui le dipendenze
    // vengono passate tramite il costruttore. Riflette sul tipo per identificare e risolvere automaticamente i parametri.
    // Definizione: Constructor Injection è una tecnica DI in cui le dipendenze sono fornite a una classe tramite il suo costruttore.
    public TService ResolveWithDependencies<TService>() {
        // Ottiene il tipo del servizio da costruire
        var type = typeof(TService);
        // Prende il primo costruttore (teoria: in un sistema reale si potrebbe scegliere il costruttore più adatto)
        var constructor = type.GetConstructors()[0];
        // Ottiene i parametri del costruttore (teoria: usa reflection per analizzare la firma del costruttore)
        // Definizione: Reflection è una funzionalità che permette di esaminare e manipolare i metadati di un tipo a runtime.
        var parameters = constructor.GetParameters();
        // Array per le istanze dei parametri risolti
        var resolvedParams = new object[parameters.Length];

        // Itera sui parametri per risolverli
        for (int i = 0; i < parameters.Length; i++) {
            // Tipo del parametro corrente
            var paramType = parameters[i].ParameterType;
            // Verifica se la dipendenza è registrata
            if (!_services.ContainsKey(paramType)) {
                throw new InvalidOperationException($"Dipendenza {paramType.Name} non registrata.");
            }
            // Risolvi la dipendenza usando la factory associata e memorizzala
            resolvedParams[i] = _services[paramType]();
        }

        // Crea l'istanza invocando il costruttore con i parametri risolti
        // Teoria: Usa reflection per chiamare dinamicamente il costruttore, un approccio comune nei container DI.
        return (TService)constructor.Invoke(resolvedParams);
    }
}

// Interfaccia per un logger
// Teoria: Le interfacce definiscono un contratto che le implementazioni devono seguire.
// Nella DI, le interfacce sono fondamentali per disaccoppiare i consumatori dalle implementazioni concrete.
// Definizione: Interfaccia è un tipo astratto che specifica un insieme di metodi che una classe deve implementare.
public interface ILogger {
    void Log(string message);
}

// Implementazione concreta del logger
// Teoria: Questa classe è una "dependency" concreta che soddisfa il contratto dell'interfaccia ILogger.
// Definizione: Dependency (dipendenza) è un oggetto o servizio richiesto da un altro oggetto per funzionare correttamente.
public class ConsoleLogger : ILogger {
    public void Log(string message) {
        Console.WriteLine($"[LOG] {message}");
    }
}

// Classe che dipende da ILogger
// Teoria: MyService è un consumatore di dipendenze. Usa la constructor injection per ricevere ILogger,
// rispettando il principio di inversione delle dipendenze (DIP) del SOLID.
// Definizione: SOLID è un acronimo per cinque principi di design orientato agli oggetti; DIP (Dependency Inversion Principle)
// afferma che i moduli di alto livello non dovrebbero dipendere da quelli di basso livello, ma entrambi da astrazioni.
public class MyService {
    // Campo privato per il logger, immutabile dopo l'inizializzazione
    private readonly ILogger _logger;

    // Costruttore con dependency injection
    // Teoria: Il costruttore è il punto di ingresso per le dipendenze, rendendo esplicita la relazione con ILogger.
    public MyService(ILogger logger) {
        _logger = logger;
    }

    // Metodo che usa la dipendenza
    public void DoWork() {
        _logger.Log("Sto lavorando!");
    }
}

class Program {
    static void Main(string[] args) {
        // Crea un nuovo container DI
        // Teoria: Il container è il cuore del sistema DI, orchestrando la gestione delle dipendenze.
        var container = new ServiceContainer();

        // Registra ILogger come singleton
        // Teoria: La registrazione associa un tipo astratto (ILogger) a un'implementazione concreta (ConsoleLogger).
        container.RegisterSingleton<ILogger>(new ConsoleLogger());
        // Registra MyService come transient
        // Teoria: Qui si usa una factory per creare MyService, risolvendo manualmente ILogger.
        container.RegisterTransient<MyService>(() => new MyService(container.Resolve<ILogger>()));

        // Risolvi e usa il servizio
        var service = container.Resolve<MyService>();
        service.DoWork();

        // Esempio con risoluzione automatica
        // Teoria: Dimostra come il container possa costruire oggetti complessi senza configurazioni manuali.
        container = new ServiceContainer();
        container.RegisterSingleton<ILogger>(new ConsoleLogger());
        var serviceWithDeps = container.ResolveWithDependencies<MyService>();
        serviceWithDeps.DoWork();
    }
}
```

### 3. Spiegazione del codice

* `ServiceContainer`:
    * Usa un `Dictionary<Type, Func<object>>` per memorizzare i servizi registrati.
    * Supporta due tipi di registrazione:
        * `Singleton`: L'istanza viene creata una volta e riutilizzata.
        * `Transient`: Una nuova istanza viene creata ogni volta che viene risolta.
    * Il metodo `Resolve<TService>` restituisce un'istanza del servizio richiesto.
    * `ResolveWithDependencies<TService>` gestisce la creazione di oggetti con dipendenze, risolvendo automaticamente i parametri del costruttore.
* Esempio pratico:
    * `ILogger` è un'interfaccia con un'implementazione (`ConsoleLogger`).
    * `MyService` dipende da `ILogger` e viene risolto tramite il container.
* Flessibilità:
    * Puoi estendere il container per supportare scope, lazy initialization o altre funzionalità, ma questo rimane un esempio minimale.

### 4. Compilazione ed esecuzione

Compila ed esegui:

```bash
dotnet build
dotnet run
```

Output atteso:

```
[LOG] Sto lavorando!
[LOG] Sto lavorando!
```

## Vantaggi e limiti

### Vantaggi:

* **IoC (Inversion of Control):**
    * Principio di design in cui il controllo del flusso di un programma viene delegato a un'entità esterna, come un container di Dependency Injection. Invece di avere oggetti che creano le proprie dipendenze, queste vengono fornite dall'esterno.
* **Factory:**
    * Un oggetto o una funzione che ha lo scopo di creare altri oggetti. Incapsula la logica di creazione degli oggetti, permettendo di astrarre il processo di istanziazione.
* **Lookup:**
    * Il processo di ricerca di un elemento specifico all'interno di una struttura dati, come un dizionario o una lista, utilizzando una chiave o un indice.
* **Costruttore:**
    * Un metodo speciale di una classe che viene invocato automaticamente quando viene creata una nuova istanza di quella classe. Il suo scopo principale è inizializzare l'oggetto.
* **Singleton:**
    * Un pattern di design che assicura che una classe abbia una sola istanza e fornisce un punto di accesso globale a essa. Viene utilizzato quando è necessario avere un'unica istanza di un oggetto condivisa tra diverse parti del sistema.
* **Transient:**
    * In ambito di Dependency Injection, si riferisce a un "lifetime" di un servizio in cui viene creata una nuova istanza del servizio ogni volta che viene richiesto.
* **Risoluzione:**
    * Il processo attraverso il quale un container di Dependency Injection fornisce un'istanza di un servizio richiesto. Il container "risolve" le dipendenze di un oggetto e fornisce le istanze necessarie.
* **Cast:**
    * La conversione esplicita di un valore da un tipo di dato a un altro. Ad esempio, convertire un oggetto di tipo "object" a un tipo più specifico.
* **Constructor Injection:**
    * Una tecnica di Dependency Injection in cui le dipendenze di un oggetto vengono fornite tramite il suo costruttore.
* **Reflection:**
    * Una funzionalità dei linguaggi di programmazione che consente di esaminare e manipolare i metadati di tipi, oggetti e assembly a runtime.
* **Interfaccia:**
    * Un tipo astratto che definisce un contratto, specificando un insieme di metodi che una classe deve implementare.
* **Dependency:**
    * Un oggetto o un servizio di cui un altro oggetto ha bisogno per funzionare correttamente.
* **SOLID/DIP:**
    * SOLID è un acronimo che rappresenta cinque principi di design orientato agli oggetti. DIP (Dependency Inversion Principle) è uno di questi principi, e afferma che i moduli di alto livello non dovrebbero dipendere da moduli di basso livello, ma entrambi dovrebbero dipendere da astrazioni.

### Limiti:

* Non gestisce automaticamente cicli di dipendenze o errori complessi.
* Manca il supporto nativo per scoped services o funzionalità avanzate senza ulteriori implementazioni.
* Richiede più codice rispetto all'uso di una libreria esistente.


