# Het Interface Segregation Principle (ISP)

Het Interface Segregation Principle is het vierde van de vijf **SOLID**-principes voor objectgeoriënteerd ontwerp. Het werd, net als de andere SOLID-principes, beschreven door Robert C. Martin.

## Definitie

> **Een klasse mag niet gedwongen worden om af te hangen van methodes die ze niet gebruikt.**

Met andere woorden: **grote, "alles-in-één"-interfaces zijn een probleem.** In plaats van één brede interface te maken met veel methodes, is het beter om meerdere **kleine, specifieke interfaces** te maken. Klassen implementeren dan enkel de interfaces die ze écht nodig hebben, in plaats van gedwongen te worden om ook methodes te implementeren die voor hen niet relevant zijn.

ISP hangt nauw samen met LSP: een veelgebruikte manier om een LSP-schending op te lossen (zoals in het vorige document, met `Bird` en `Penguin`) is net door een brede interface of basisklasse op te splitsen in kleinere, meer specifieke stukken — en dát opsplitsen is precies waar ISP over gaat.

## De problemen die ontstaan als je ISP niet volgt

Wanneer een interface te veel methodes bevat die niet voor elke implementatie relevant zijn, ontstaan er een aantal concrete problemen:

1. **Gedwongen "lege" of foutgevende implementaties**
   Een klasse die de brede interface implementeert, maar bepaalde methodes niet kan ondersteunen, is verplicht om die methodes toch te implementeren — vaak met een lege body, of erger, met een `NotImplementedException` of `NotSupportedException`. Dit is exact hetzelfde symptoom dat we zagen bij LSP-schendingen.

2. **Onnodige afhankelijkheden**
   Een klasse die maar één van de vijf methodes van een interface nodig heeft, is toch afhankelijk van alle vijf. Wijzigt er iets aan een methode die deze klasse niet eens gebruikt, dan kan haar code (of op zijn minst haar compilatie) daar toch door beïnvloed worden.

3. **Interfaces worden moeilijk te implementeren**
   Hoe meer methodes een interface bevat, hoe meer werk (en hoe meer kans op fouten) het is om ze correct te implementeren — zelfs voor het deel van het gedrag dat een klasse eigenlijk niet nodig heeft.

4. **Onduidelijke intentie**
   Als een klasse een brede interface implementeert, is het voor iemand die de code leest niet meteen duidelijk welke methodes ze écht zinvol ondersteunt en welke enkel "verplicht aanwezig" zijn. Dit maakt de code moeilijker te begrijpen.

5. **Client-code wordt fragieler**
   Code die met de brede interface werkt, kan methodes aanroepen die voor het concrete object eigenlijk niet van toepassing zijn, met een crash tot gevolg — net zoals bij een LSP-schending.

## De oplossing: ISP toepassen

De oplossing bestaat erin om **grote interfaces op te splitsen in kleinere, meer specifieke interfaces**, elk gericht op één samenhangend stukje gedrag. Een klasse implementeert dan enkel de interfaces die voor haar daadwerkelijk relevant zijn.

Typisch patroon:

1. Kijk naar een brede interface en groepeer de methodes op basis van welke soort klassen ze effectief nodig hebben.
2. Splits de interface op in meerdere kleinere interfaces, één per groep.
3. Laat elke klasse enkel de interfaces implementeren die ze werkelijk kan waarmaken.
4. Code die met deze objecten werkt, vraagt enkel naar de interface die ze nodig heeft (bv. enkel `IPrintable`, niet de volledige oude interface).

### Waarom dit de bovenstaande problemen oplost

- **Geen gedwongen lege/foutgevende implementaties meer**: een klasse implementeert enkel interfaces waarvan ze elke methode zinvol kan waarmaken.
- **Minder onnodige afhankelijkheden**: een klasse hangt enkel af van de kleine interface die ze echt gebruikt, niet van een brede interface vol ongebruikte methodes.
- **Makkelijker te implementeren**: kleinere interfaces betekenen minder methodes om te implementeren per klasse, en dus minder kans op fouten.
- **Duidelijkere intentie**: als een klasse `IPrintable` implementeert maar niet `IScannable`, is meteen duidelijk wat ze wel en niet kan.
- **Robuustere client-code**: code die enkel met `IPrintable` werkt, kan nooit per ongeluk een niet-ondersteunde methode aanroepen, want die methode bestaat gewoon niet in die interface.

## Voorbeelden

### Voorbeeld 1 — ISP overtreden

```csharp
public interface IMultiFunctionDevice
{
    void Print(string document);
    void Scan(string document);
    void Fax(string document);
}

public class ModernPrinter : IMultiFunctionDevice
{
    public void Print(string document) => Console.WriteLine($"Printen: {document}");
    public void Scan(string document) => Console.WriteLine($"Scannen: {document}");
    public void Fax(string document) => Console.WriteLine($"Faxen: {document}");
}

public class BasicPrinter : IMultiFunctionDevice
{
    public void Print(string document) => Console.WriteLine($"Printen: {document}");

    public void Scan(string document)
    {
        throw new NotSupportedException("Deze printer kan niet scannen.");
    }

    public void Fax(string document)
    {
        throw new NotSupportedException("Deze printer kan niet faxen.");
    }
}
```

`BasicPrinter` is een eenvoudige printer die enkel kan printen. Toch is ze, door de brede interface `IMultiFunctionDevice`, verplicht om ook `Scan` en `Fax` te implementeren — methodes die ze niet kan waarmaken. Elke keer dat iemand `Scan` of `Fax` aanroept op een `BasicPrinter` (wat perfect toegestaan is volgens het type `IMultiFunctionDevice`), crasht het programma.

### Voorbeeld 2 — ISP toepassen

```csharp
public interface IPrintable
{
    void Print(string document);
}

public interface IScannable
{
    void Scan(string document);
}

public interface IFaxable
{
    void Fax(string document);
}

public class ModernPrinter : IPrintable, IScannable, IFaxable
{
    public void Print(string document) => Console.WriteLine($"Printen: {document}");
    public void Scan(string document) => Console.WriteLine($"Scannen: {document}");
    public void Fax(string document) => Console.WriteLine($"Faxen: {document}");
}

public class BasicPrinter : IPrintable
{
    public void Print(string document) => Console.WriteLine($"Printen: {document}");
}
```

Nu implementeert `BasicPrinter` enkel `IPrintable` — de enige interface die ze effectief kan waarmaken. Er is geen enkele methode meer die ze moet "weigeren". Code die enkel iets wil printen, vraagt gewoon naar een `IPrintable`, en kan zowel een `ModernPrinter` als een `BasicPrinter` gebruiken zonder risico op een crash.

### Voorbeeld 3 — Een klein alledaags geval: werknemers

```csharp
// Overtreedt ISP: niet elke werknemer overwerkt
public interface IEmployee
{
    void DoRegularWork();
    void DoOvertimeWork();
}

public class PartTimeEmployee : IEmployee
{
    public void DoRegularWork() => Console.WriteLine("Regulier werk uitvoeren...");

    public void DoOvertimeWork()
    {
        throw new NotSupportedException("Deeltijdse werknemers doen geen overwerk.");
    }
}
```

```csharp
// Volgt ISP: overwerk is een apart, optioneel gedrag
public interface IEmployee
{
    void DoRegularWork();
}

public interface IOvertimeEligible
{
    void DoOvertimeWork();
}

public class PartTimeEmployee : IEmployee
{
    public void DoRegularWork() => Console.WriteLine("Regulier werk uitvoeren...");
}

public class FullTimeEmployee : IEmployee, IOvertimeEligible
{
    public void DoRegularWork() => Console.WriteLine("Regulier werk uitvoeren...");
    public void DoOvertimeWork() => Console.WriteLine("Overwerk uitvoeren...");
}
```

`PartTimeEmployee` implementeert nu enkel `IEmployee`, en moet dus nergens meer een methode "weigeren" die eigenlijk niet op haar van toepassing is.

## Samenvatting

| Zonder ISP | Met ISP |
|---|---|
| Eén brede interface met veel methodes | Meerdere kleine, specifieke interfaces |
| Klassen implementeren methodes die ze niet gebruiken | Klassen implementeren enkel wat ze echt nodig hebben |
| Lege of foutgevende implementaties | Elke geïmplementeerde methode is zinvol |
| Onduidelijk wat een klasse écht ondersteunt | Interfaces maken meteen duidelijk wat een klasse kan |

ISP betekent niet dat elke interface maar één methode mag hebben. Het betekent dat een interface enkel methodes mag bundelen die **werkelijk samenhoren**, zodat geen enkele klasse gedwongen wordt om gedrag te implementeren dat voor haar niet van toepassing is.