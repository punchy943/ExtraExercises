# Het Dependency Inversion Principle (DIP)

Het Dependency Inversion Principle is het vijfde en laatste van de vijf **SOLID**-principes voor objectgeoriënteerd ontwerp. Het gaat over hoe klassen van elkaar zouden moeten afhangen.

## Definitie

> **Hooggeplaatste modules mogen niet afhangen van laaggeplaatste modules. Beide moeten afhangen van abstracties. Abstracties mogen niet afhangen van details; details moeten afhangen van abstracties.**

Dit klinkt abstract, dus laten we het opsplitsen:

- **Hooggeplaatste modules** ("high-level modules") zijn klassen die de business logica of het overkoepelende proces bevatten (bv. "een bestelling verwerken").
- **Laaggeplaatste modules** ("low-level modules") zijn klassen die concrete, technische details afhandelen (bv. "een e-mail versturen via SMTP", "gegevens opslaan in SQL Server").
- Normaal gezien zou je verwachten dat de hooggeplaatste module rechtstreeks afhangt van de laaggeplaatste module (het proces "gebruikt" de techniek). DIP draait dit om: **beide moeten afhangen van een abstractie** (een interface) **in plaats van van elkaar**.

Merk op dat de naam letterlijk "**inversie** van afhankelijkheid" betekent: in plaats van dat de business logica rechtstreeks afhangt van een concrete technische implementatie, hangt de technische implementatie af van een abstractie die de business logica zelf definieert.

## De problemen die ontstaan als je DIP niet volgt

Wanneer hooggeplaatste klassen rechtstreeks concrete, laaggeplaatste klassen aanmaken en gebruiken (bijvoorbeeld via `new SqlDatabase()` binnenin een business-klasse), ontstaan er een aantal concrete problemen:

1. **Sterke koppeling aan een specifieke technologie**
   Als je business logica rechtstreeks een concrete klasse zoals `SqlDatabase` gebruikt, wordt overstappen naar een andere technologie (bv. een NoSQL-database, of gewoon een andere provider) een pijnlijke operatie: je moet de business logica zelf aanpassen, ook al is de eigenlijke business-regel niet veranderd.

2. **Moeilijk of onmogelijk te unit testen**
   Als een klasse intern zelf `new SqlDatabase()` aanmaakt, kan je die database in een unit test niet vervangen door een nepversie (een "mock" of "stub"). Je test raakt dan gedwongen verbonden met een echte database, wat testen traag, fragiel en soms zelfs onmogelijk maakt zonder een echte database-omgeving.

3. **Weinig flexibiliteit**
   Als je bijvoorbeeld tijdelijk wil overschakelen naar een andere manier van notificaties versturen (bv. SMS in plaats van e-mail), moet je de business klasse zelf aanpassen, in plaats van gewoon een andere implementatie van dezelfde abstractie door te geven.

4. **Schendt vaak ook OCP**
   Omdat de hooggeplaatste klasse rechtstreeks weet welke concrete klasse ze gebruikt, moet je die hooggeplaatste klasse aanpassen telkens er een nieuwe technische implementatie bijkomt — exact het probleem dat het Open/Closed Principle net probeert te vermijden.

5. **Business logica en technische details raken vermengd**
   Zonder DIP weet je business klasse plots ook "hoe" iets technisch gebeurt (bv. de exacte manier waarop een e-mail verstuurd wordt), in plaats van enkel "dat" het moet gebeuren. Dit maakt de klasse moeilijker te begrijpen en botst met het Single Responsibility Principle.

## De oplossing: DIP toepassen

De oplossing bestaat erin om **abstracties (interfaces) te definiëren vanuit het perspectief van de hooggeplaatste module**, en de laaggeplaatste, technische klassen die abstractie te laten implementeren. De hooggeplaatste klasse krijgt de concrete implementatie dan **van buitenaf aangereikt** (bijvoorbeeld via de constructor), in plaats van ze zelf aan te maken. Dit heet **dependency injection**, en is de meest gebruikte techniek om DIP in de praktijk toe te passen.

Typisch patroon:

1. Definieer een interface die beschrijft **wat** de hooggeplaatste module nodig heeft (bv. `INotificationSender` met een methode `Send`), zonder iets te zeggen over **hoe** dat precies gebeurt.
2. Laat de hooggeplaatste klasse enkel afhangen van deze interface, nooit van een concrete klasse.
3. Laat elke concrete, technische implementatie (bv. `EmailNotificationSender`, `SmsNotificationSender`) deze interface implementeren.
4. Geef de gewenste concrete implementatie door aan de hooggeplaatste klasse via de constructor (of een andere vorm van injectie), in plaats van ze intern met `new` aan te maken.

### Waarom dit de bovenstaande problemen oplost

- **Losse koppeling aan technologie**: de business klasse kent enkel de interface, niet de concrete technologie erachter. Een nieuwe technologie toevoegen of een bestaande vervangen raakt de business klasse niet.
- **Makkelijk te unit testen**: in een test kan je een nepimplementatie (mock) van de interface doorgeven, zodat je de business logica volledig geïsoleerd kan testen, zonder een echte database, e-mailserver, ...
- **Flexibel**: welke concrete implementatie gebruikt wordt, kan op één centrale plek bepaald worden (vaak bij het opstarten van de applicatie), zonder de business logica zelf aan te passen.
- **Ondersteunt OCP**: een nieuwe implementatie toevoegen (bv. een nieuwe manier van notificaties versturen) vereist geen wijziging aan de hooggeplaatste klasse — enkel een nieuwe klasse die de interface implementeert.
- **Business logica blijft puur**: de hooggeplaatste klasse weet enkel **dat** ze iets moet laten gebeuren via de interface, niet **hoe** dat precies technisch in zijn werk gaat.

## Voorbeelden

### Voorbeeld 1 — DIP overtreden

```csharp
public class EmailSender
{
    public void SendEmail(string to, string message)
    {
        Console.WriteLine($"E-mail verzonden naar {to}: {message}");
    }
}

public class OrderService
{
    private readonly EmailSender _emailSender = new EmailSender();

    public void PlaceOrder(string customerEmail, string orderDetails)
    {
        // ... bestelling verwerken ...

        _emailSender.SendEmail(customerEmail, $"Uw bestelling is geplaatst: {orderDetails}");
    }
}
```

`OrderService` (de hooggeplaatste module, verantwoordelijk voor het bestelproces) maakt hier zelf een concrete `EmailSender` (de laaggeplaatste, technische module) aan met `new`. Wil je ooit overschakelen naar SMS-notificaties, of `OrderService` unit testen zonder echt een e-mail te versturen? Dan zit je vast, want `OrderService` is rechtstreeks gekoppeld aan `EmailSender`.

### Voorbeeld 2 — DIP toepassen

```csharp
public interface INotificationSender
{
    void Send(string to, string message);
}

public class EmailSender : INotificationSender
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"E-mail verzonden naar {to}: {message}");
    }
}

public class SmsSender : INotificationSender
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"SMS verzonden naar {to}: {message}");
    }
}

public class OrderService
{
    private readonly INotificationSender _notificationSender;

    public OrderService(INotificationSender notificationSender)
    {
        _notificationSender = notificationSender;
    }

    public void PlaceOrder(string customerContact, string orderDetails)
    {
        // ... bestelling verwerken ...

        _notificationSender.Send(customerContact, $"Uw bestelling is geplaatst: {orderDetails}");
    }
}
```

`OrderService` hangt nu enkel af van de abstractie `INotificationSender`, niet van een concrete technologie. Welke implementatie gebruikt wordt, wordt van buitenaf bepaald:

```csharp
var orderService = new OrderService(new EmailSender());
// of eenvoudig omwisselen naar:
var orderServiceViaSms = new OrderService(new SmsSender());
```

`OrderService` moet hiervoor **niet** aangepast worden, en kan in een unit test moeiteloos gecombineerd worden met een nepimplementatie van `INotificationSender` die niets echt verstuurt.

### Voorbeeld 3 — Een klein alledaags geval: logging

```csharp
// Overtreedt DIP: ReportGenerator is rechtstreeks gekoppeld aan FileLogger
public class FileLogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[Bestand] {message}");
    }
}

public class ReportGenerator
{
    private readonly FileLogger _logger = new FileLogger();

    public void GenerateReport()
    {
        _logger.Log("Rapport wordt gegenereerd...");
        // ... rapportlogica ...
    }
}
```

```csharp
// Volgt DIP: ReportGenerator hangt af van een abstractie
public interface ILogger
{
    void Log(string message);
}

public class FileLogger : ILogger
{
    public void Log(string message) => Console.WriteLine($"[Bestand] {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine($"[Console] {message}");
}

public class ReportGenerator
{
    private readonly ILogger _logger;

    public ReportGenerator(ILogger logger)
    {
        _logger = logger;
    }

    public void GenerateReport()
    {
        _logger.Log("Rapport wordt gegenereerd...");
        // ... rapportlogica ...
    }
}
```

`ReportGenerator` weet nu enkel dat er **iets** gelogd moet worden, niet **hoe** of **waar** precies. Welke `ILogger`-implementatie gebruikt wordt, kan vrij gekozen (en gewijzigd) worden zonder `ReportGenerator` aan te raken.

## Samenvatting

| Zonder DIP | Met DIP |
|---|---|
| Hooggeplaatste klasse maakt zelf `new ConcreteClass()` aan | Hooggeplaatste klasse hangt af van een interface |
| Sterke koppeling aan een specifieke technologie | Technologie kan vrij gewisseld worden |
| Moeilijk of onmogelijk te unit testen | Makkelijk te testen met een nepimplementatie |
| Business logica en technische details vermengd | Business logica kent enkel het "wat", niet het "hoe" |

DIP betekent niet dat je overal interfaces moet maken, ook waar dat geen meerwaarde biedt. Het betekent dat de **richting van afhankelijkheid** omgekeerd moet worden op de plekken waar business logica en technische details elkaar raken: beide kanten moeten afhangen van een abstractie, in plaats van dat de business logica rechtstreeks vastgekoppeld zit aan een technische implementatie.