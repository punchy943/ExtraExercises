# Het Liskov Substitution Principle (LSP)

Het Liskov Substitution Principle is het derde van de vijf **SOLID**-principes voor objectgeoriënteerd ontwerp. Het werd geformuleerd door Barbara Liskov in 1987 en gaat over hoe overerving (inheritance) correct gebruikt moet worden.

## Definitie

> **Objecten van een subklasse moeten overal inzetbaar zijn waar objecten van de basisklasse verwacht worden, zonder dat de correctheid van het programma daardoor verandert.**

Met andere woorden: als klasse `B` overerft van klasse `A`, dan moet je overal waar je een object van type `A` gebruikt, dat object kunnen **vervangen door een object van type `B`**, zonder dat het gedrag van het programma onverwacht verandert of breekt.

Overerving mag dus niet enkel gebaseerd zijn op "lijkt logisch" (bv. "een vierkant is een soort rechthoek, dus laten we `Square` overerven van `Rectangle`"). Het moet ook gedrag-technisch kloppen: een subklasse mag geen beloftes (het "contract") van de basisklasse breken.

## De problemen die ontstaan als je LSP niet volgt

Wanneer een subklasse zich niet gedraagt zoals verwacht op basis van zijn basisklasse, ontstaan er een aantal concrete problemen:

1. **Onverwachte exceptions**
   Een klassiek symptoom van een LSP-schending is een subklasse die een methode overschrijft en daarin een `NotImplementedException` of vergelijkbare fout gooit, omdat die functionaliteit "eigenlijk niet van toepassing is" op dat specifieke geval. Code die met de basisklasse werkt, verwacht dat niet en crasht.

2. **Verborgen `if`-checks op het concrete type**
   Om de onverwachte exceptions te vermijden, ziet men vaak code zoals `if (obj is Penguin) { ... } else { obj.Fly(); }`. Dit ondermijnt net het hele idee van polymorfisme: de aanroepende code moet nu weten welke concrete subklassen er bestaan, in plaats van gewoon met de abstractie te werken.

3. **Breekt het vertrouwen in abstractie**
   Eén van de grote voordelen van overerving en interfaces is dat je code kan schrijven die werkt met de basisklasse, zonder je zorgen te moeten maken over welke concrete subklasse er precies gebruikt wordt. Zodra één subklasse zich niet aan het contract houdt, moet je bij élke plaats waar de basisklasse gebruikt wordt, nagaan of het wel veilig is — het vertrouwen in de abstractie valt weg.

4. **Moeilijker uit te breiden**
   Omdat ontwikkelaars niet meer blindelings kunnen vertrouwen op het gedrag van de basisklasse, wordt het toevoegen van een nieuwe subklasse riskant: je weet nooit zeker of jouw nieuwe klasse ergens een aanname breekt die impliciet in de bestaande code zit.

5. **Vaak ook een schending van OCP**
   Om rond een LSP-schending heen te werken, voegen ontwikkelaars vaak `if`/`switch`-checks toe op het concrete type in de aanroepende code. Dat is exact het patroon dat het Open/Closed Principle net probeert te vermijden — de twee principes hangen dus nauw samen.

## De oplossing: LSP toepassen

De oplossing bestaat erin om **overerving enkel te gebruiken wanneer het gedrag écht compatibel is**, en anders je hiërarchie te herdenken. Een aantal concrete technieken:

1. **Test het "is-a"-gevoel tegen gedrag, niet enkel tegen taalgebruik.** "Een vierkant is een rechthoek" klinkt logisch in het Nederlands, maar als het gedrag van `Rectangle` (breedte en hoogte onafhankelijk aanpasbaar) niet geldig is voor `Square` (breedte = hoogte), dan hoort dat niet in dezelfde overervingshiërarchie.
2. **Splits de basisklasse of interface op** wanneer niet elke subklasse alle mogelijke gedragingen kan ondersteunen (dit is trouwens ook waar het volgende SOLID-principe, Interface Segregation, over gaat).
3. **Gebruik samenstelling (compositie) in plaats van overerving** wanneer twee klassen wel gelijkenissen vertonen, maar niet volledig uitwisselbaar zijn.
4. **Zorg dat overschreven methodes het contract van de basisklasse respecteren**: geen striktere voorwaarden (preconditions) opleggen, geen zwakkere garanties (postconditions) bieden, en geen nieuwe exceptions gooien die de aanroeper niet verwacht.

### Waarom dit de bovenstaande problemen oplost

- **Geen onverwachte exceptions**: als een subklasse enkel bestaat wanneer ze het volledige gedrag van de basisklasse kan waarmaken, valt de noodzaak om ergens een `NotImplementedException` te gooien weg.
- **Geen verborgen type-checks nodig**: aanroepende code kan blindelings vertrouwen op de abstractie, zonder te moeten weten welke concrete subklasse erachter zit.
- **Abstractie blijft betrouwbaar**: iedereen die met de basisklasse werkt, kan erop vertrouwen dat elke subklasse zich hetzelfde gedraagt op het niveau van het contract.
- **Makkelijker uit te breiden**: nieuwe subklassen kunnen toegevoegd worden zonder angst om iets te breken, zolang ze het contract van de basisklasse respecteren.
- **Ondersteunt OCP**: doordat aanroepende code niet moet controleren "welk concreet type is dit", blijft het systeem open voor uitbreiding zonder wijziging.

## Voorbeelden

### Voorbeeld 1 — LSP overtreden

```csharp
public class Rectangle
{
    public virtual double Width { get; set; }
    public virtual double Height { get; set; }

    public double CalculateArea() => Width * Height;
}

public class Square : Rectangle
{
    public override double Width
    {
        get => base.Width;
        set
        {
            base.Width = value;
            base.Height = value; // een vierkant moet gelijke zijden hebben
        }
    }

    public override double Height
    {
        get => base.Height;
        set
        {
            base.Height = value;
            base.Width = value; // idem in de andere richting
        }
    }
}
```

```csharp
public static void ResizeAndPrintArea(Rectangle rectangle)
{
    rectangle.Width = 5;
    rectangle.Height = 10;

    // Voor een "gewone" Rectangle verwacht je hier 50 (5 x 10)
    Console.WriteLine($"Oppervlakte: {rectangle.CalculateArea()}");
}
```

Roep je `ResizeAndPrintArea` aan met een `Rectangle`, dan krijg je `50`, zoals verwacht. Roep je ze aan met een `Square`, dan krijg je plots `100` (10 x 10), omdat het instellen van `Height` ook `Width` overschrijft. De `Square` is dus **niet substitueerbaar** voor `Rectangle`: dezelfde code geeft een ander, onverwacht resultaat. Dit is een schoolvoorbeeld van een LSP-schending.

### Voorbeeld 2 — LSP toepassen

```csharp
public interface IShape
{
    double CalculateArea();
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double CalculateArea() => Width * Height;
}

public class Square : IShape
{
    public double Side { get; set; }

    public double CalculateArea() => Side * Side;
}
```

`Rectangle` en `Square` erven nu niet meer van elkaar — ze hebben elk hun eigen, correcte gedrag en delen enkel de `IShape`-abstractie. Er is geen enkele plaats meer waar het aanpassen van de ene eigenschap ongewild een andere eigenschap verandert. Beide klassen zijn volledig substitueerbaar overal waar een `IShape` verwacht wordt.

### Voorbeeld 3 — Een klein alledaags geval: vogels

```csharp
// Overtreedt LSP: niet elke Bird kan vliegen
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Ik vlieg weg!");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException("Pinguïns kunnen niet vliegen.");
    }
}
```

```csharp
public static void LetBirdFly(Bird bird)
{
    bird.Fly(); // werkt voor de meeste vogels, maar crasht voor Penguin
}
```

Elke keer dat ergens in de code een `Bird` doorgegeven wordt aan `LetBirdFly`, loop je het risico op een crash — maar enkel als het toevallig een `Penguin` is. Dat is exact het probleem: de aanroeper kan `Penguin` niet zomaar gebruiken waar een `Bird` verwacht wordt.

```csharp
// Volgt LSP: vliegen is een apart, optioneel gedrag
public abstract class Bird
{
    public abstract void Move();
}

public interface IFlyingBird
{
    void Fly();
}

public class Sparrow : Bird, IFlyingBird
{
    public override void Move() => Fly();

    public void Fly() => Console.WriteLine("Ik vlieg weg!");
}

public class Penguin : Bird
{
    public override void Move() => Console.WriteLine("Ik waggel voort!");
}
```

Nu belooft `Bird` enkel nog `Move()` — iets wat elke vogel effectief kan. Enkel vogels die ook echt kunnen vliegen, implementeren `IFlyingBird`. Er is geen enkele subklasse meer die een methode van haar basistype moet "weigeren" met een exception.

## Samenvatting

| Zonder LSP | Met LSP |
|---|---|
| Subklasse gooit onverwachte exceptions | Subklasse implementeert enkel gedrag dat ze ook echt waarmaakt |
| Aanroepende code doet verborgen `if (obj is ...)`-checks | Aanroepende code vertrouwt blindelings op de abstractie |
| Overerving gebaseerd op taalgebruik ("is een soort van") | Overerving gebaseerd op effectief compatibel gedrag |
| Nieuwe subklasse toevoegen is riskant | Nieuwe subklasse toevoegen is veilig zolang ze het contract respecteert |

LSP betekent niet dat overerving verboden is. Het betekent dat een subklasse zich moet gedragen als een **volwaardige, betrouwbare vervanger** van haar basisklasse — nooit als een uitzondering die extra voorzichtigheid vereist van elke aanroeper.