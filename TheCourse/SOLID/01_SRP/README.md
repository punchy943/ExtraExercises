# Het Single Responsibility Principle (SRP)

Het Single Responsibility Principle is het eerste van de vijf **SOLID**-principes voor objectgeoriënteerd ontwerp. SOLID is een acroniem geïntroduceerd door Robert C. Martin ("Uncle Bob"), en SRP wordt algemeen beschouwd als de basis waarop de andere vier principes verder bouwen.

## Definitie

> **Een klasse mag maar één reden hebben om te veranderen.**

Met andere woorden: een klasse zou precies **één taak of verantwoordelijkheid** moeten hebben. Als een klasse meer dan één verantwoordelijkheid op zich neemt, raken die verantwoordelijkheden met elkaar verweven — en kan een wijziging aan de ene verantwoordelijkheid de andere beïnvloeden of zelfs breken, ook al hebben ze conceptueel niets met elkaar te maken.

Belangrijk om op te merken: "verantwoordelijkheid" betekent hier niet "één methode" of "één regel code". Het betekent "één *reden om te veranderen*" — één concern (business of technisch) waarvoor de klasse verantwoordelijk is. Een klasse mag dus best meerdere methodes bevatten, zolang al die methodes dezelfde verantwoordelijkheid dienen.

## De problemen die ontstaan als je SRP niet volgt

Wanneer een klasse meerdere verantwoordelijkheden combineert, duiken er naarmate de codebase groeit een aantal concrete problemen op:

1. **Sterke koppeling tussen niet-verwante zaken**
   Als een klasse business data berekent, deze opslaat in een database, én ze opmaakt voor weergave, dan zorgt een wijziging in de databasetechnologie (bijvoorbeeld van een bestand naar SQL Server) ervoor dat je dezelfde klasse moet aanpassen als bij een wijziging in een business regel. Twee ontwikkelaars die aan niet-gerelateerde features werken, kunnen zo in hetzelfde bestand terechtkomen en merge conflicts veroorzaken.

2. **Moeilijker te testen**
   Een klasse met meerdere verantwoordelijkheden heeft meestal ook meerdere afhankelijkheden (bv. bestandstoegang, opmaaklogica en business regels allemaal samen). Om enkel de business logica te unit testen, ben je verplicht om ook met bestands-I/O of console-output om te gaan, wat tests trager, fragieler en moeilijker te schrijven maakt.

3. **Moeilijker herbruikbaar**
   Als de logica om loon te berekenen verstopt zit in een klasse die ook weet hoe ze een loonbriefje moet afdrukken, kan je die berekeningslogica niet ergens anders hergebruiken (bijvoorbeeld in een rapport of een API) zonder ook de afdruklogica mee te slepen.

4. **Moeilijker te begrijpen**
   Een klasse die veel dingen doet, is moeilijker te lezen en te doorgronden. Nieuwe ontwikkelaars (of jijzelf, later) moeten *alle* verantwoordelijkheden begrijpen, ook al willen ze maar een kleine wijziging aan één ervan aanbrengen.

5. **Zorgt voor een "rimpeleffect"**
   Eén wijziging in de requirements (bv. "loonbriefjes moeten nu gemaild worden in plaats van afgedrukt") verspreidt zich zo doorheen een klasse die ook niet-gerelateerde logica bevat, wat het risico verhoogt dat je per ongeluk iets breekt dat niets met de oorspronkelijke wijziging te maken had.

## De oplossing: SRP toepassen

De oplossing bestaat erin om **verantwoordelijkheden op te splitsen in aparte klassen**, elk met één duidelijk afgebakende taak. In plaats van één klasse die berekening, opslag én weergave doet, krijg je bijvoorbeeld:

- Een klasse verantwoordelijk voor de **business logica** (bv. loon berekenen)
- Een klasse verantwoordelijk voor de **opslag** (bv. gegevens opslaan in een bestand of database)
- Een klasse verantwoordelijk voor de **weergave** (bv. een rapport opmaken/afdrukken)

Elke klasse heeft nu precies één reden om te veranderen, en ze worden met elkaar verbonden (vaak via interfaces) door een hoger gelegen stuk code dat ze coördineert.

### Waarom dit de bovenstaande problemen oplost

- **Losse koppeling**: een wijziging in hoe gegevens worden opgeslagen raakt de business logica niet meer, en omgekeerd. Elke klasse kan onafhankelijk evolueren.
- **Makkelijker te testen**: de klasse met business logica kan geïsoleerd unit getest worden, zonder bestandssysteem of console nodig te hebben.
- **Herbruikbaar**: de berekeningslogica kan nu overal hergebruikt worden (API, rapport, achtergrondtaak) zonder afdruk- of opslagcode mee te slepen.
- **Makkelijker te begrijpen**: elke klasse kan op zichzelf gelezen en begrepen worden, aangezien ze maar één ding doet.
- **Lokale wijzigingen**: een nieuwe vereiste over *hoe* gegevens getoond worden, raakt enkel de weergaveklasse — de rest van het systeem blijft ongewijzigd.

## Voorbeelden

### Voorbeeld 1 — SRP overtreden

```csharp
public class Report
{
    public string Title { get; set; }
    public string Content { get; set; }

    // Verantwoordelijkheid 1: de inhoud van het rapport opbouwen
    public string GenerateReport()
    {
        return $"{Title}\n{Content}";
    }

    // Verantwoordelijkheid 2: het rapport opslaan op schijf
    public void SaveToFile(string path)
    {
        System.IO.File.WriteAllText(path, GenerateReport());
    }
}
```

Hier heeft `Report` twee redenen om te veranderen: hoe de inhoud van een rapport gegenereerd wordt, en hoe/waar het opgeslagen wordt. Als rapporten morgen in een database opgeslagen moeten worden in plaats van in een bestand, moet deze klasse aangepast worden — ook al hoefde de logica om het rapport te genereren helemaal niet te veranderen.

### Voorbeeld 2 — SRP toepassen

```csharp
// Enkel verantwoordelijk voor het voorstellen en genereren van de rapportinhoud
public class Report
{
    public string Title { get; set; }
    public string Content { get; set; }

    public string GenerateReport()
    {
        return $"{Title}\n{Content}";
    }
}

// Enkel verantwoordelijk voor het opslaan van tekst in een bestand
public class FileSaver
{
    public void Save(string path, string content)
    {
        System.IO.File.WriteAllText(path, content);
    }
}

// Coördineert de twee, zonder zelf het werk te doen
public class ReportService
{
    private readonly FileSaver _fileSaver;

    public ReportService(FileSaver fileSaver)
    {
        _fileSaver = fileSaver;
    }

    public void ExportReport(Report report, string path)
    {
        string content = report.GenerateReport();
        _fileSaver.Save(path, content);
    }
}
```

Nu weet `Report` enkel hoe het zijn eigen inhoud moet voorstellen en genereren. `FileSaver` weet enkel hoe tekst naar schijf geschreven wordt — het weet zelfs niet dat het met een "rapport" te maken heeft. `ReportService` is de enige klasse die de twee met elkaar verbindt, en dat doet ze zonder zelf business- of opslaglogica te bevatten.

Als de manier van opslaan moet veranderen (bv. opslaan in een database in plaats van een bestand), moet enkel `FileSaver` (of een nieuwe klasse die hetzelfde idee implementeert) aangepast worden. `Report` blijft volledig ongewijzigd.

### Voorbeeld 3 — Een klein alledaags geval

```csharp
// Overtreedt SRP: validatie en opslag zitten samen in één klasse
public class UserRegistration
{
    public bool RegisterUser(string email, string password)
    {
        if (!email.Contains("@"))
        {
            return false; // ongeldig e-mailadres
        }

        // Doet alsof dit de gebruiker opslaat in een database
        Console.WriteLine($"Gebruiker {email} opslaan in database...");
        return true;
    }
}
```

```csharp
// Volgt SRP: validatie en opslag zijn gescheiden
public class EmailValidator
{
    public bool IsValid(string email) => email.Contains("@");
}

public class UserRepository
{
    public void Save(string email, string password)
    {
        Console.WriteLine($"Gebruiker {email} opslaan in database...");
    }
}

public class UserRegistrationService
{
    private readonly EmailValidator _validator;
    private readonly UserRepository _repository;

    public UserRegistrationService(EmailValidator validator, UserRepository repository)
    {
        _validator = validator;
        _repository = repository;
    }

    public bool RegisterUser(string email, string password)
    {
        if (!_validator.IsValid(email))
        {
            return false;
        }

        _repository.Save(email, password);
        return true;
    }
}
```

Elke klasse beantwoordt nu precies één vraag:
- `EmailValidator` → "Is dit e-mailadres geldig?"
- `UserRepository` → "Hoe sla ik een gebruiker op?"
- `UserRegistrationService` → "Wat zijn de stappen om een gebruiker te registreren?"

### Voorbeelden in de cursus

BookTracker project: 
- `BookPermissions` → "Mag deze gebruiker boeken beheren?"
- `BookTitle` & `AuthorName` → "Is dit een geldige titel/naam?"
- `CreateBookCommandHandler` → "Hoe wordt een boek aangemaakt?"
- `UpdateBookCommandHandler` → "Hoe wordt een boek aangepast?"
- `DeleteBookCommandHandler` → "Hoe wordt een boek verwijderd?"
- `GetBookDetailsQueryHandler` & `GetBookSummariesQueryHandler` → "Hoe wordt een boek getoond?" 
- `BookRepository` → "Hoe sla ik een boek op?"

## Samenvatting

| Zonder SRP | Met SRP |
|---|---|
| Eén klasse, veel redenen om te veranderen | Meerdere klassen, elk met één reden om te veranderen |
| Moeilijk geïsoleerd te testen | Elke verantwoordelijkheid makkelijk apart te unit testen |
| Logica is moeilijk herbruikbaar | Logica kan onafhankelijk hergebruikt worden |
| Wijzigingen hebben een onvoorspelbaar rimpeleffect | Wijzigingen zijn lokaal en voorspelbaar |

SRP betekent niet "elke klasse mag maar één methode hebben". Het betekent dat elke klasse verantwoordelijk moet zijn voor **één samenhangend concern**, zodat ze maar om **één reden** ooit moet veranderen.