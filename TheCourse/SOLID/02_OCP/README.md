# Het Open/Closed Principle (OCP)

Het Open/Closed Principle is het tweede van de vijf **SOLID**-principes voor objectgeoriënteerd ontwerp. Het werd oorspronkelijk beschreven door Bertrand Meyer en later populair gemaakt binnen SOLID door Robert C. Martin.

## Definitie

> **Software-entiteiten (klassen, modules, functies, ...) moeten open zijn voor uitbreiding, maar gesloten voor wijziging.**

Met andere woorden: je moet nieuw gedrag aan een systeem kunnen toevoegen **zonder bestaande, al geteste code aan te passen**. In plaats van een bestaande klasse telkens te wijzigen wanneer er een nieuw geval bijkomt, ontwerp je de code zo dat nieuw gedrag toegevoegd kan worden door **nieuwe code te schrijven** (bijvoorbeeld een nieuwe klasse), niet door bestaande code open te breken.

- **"Open voor uitbreiding"** → het moet mogelijk zijn om het gedrag van de applicatie uit te breiden.
- **"Gesloten voor wijziging"** → de broncode van bestaande, werkende klassen moet daarvoor niet aangepast worden.

## De problemen die ontstaan als je OCP niet volgt

Wanneer nieuw gedrag telkens vereist dat je een bestaande klasse aanpast (vaak door een `if`/`else`- of `switch`-blok uit te breiden), ontstaan er een aantal concrete problemen:

1. **Risico op regressies**
   Elke keer dat je een bestaande, al geteste klasse aanpast, loop je het risico dat je iets breekt dat voorheen perfect werkte. Hoe meer een klasse gewijzigd wordt, hoe groter de kans op onbedoelde bugs in ongerelateerde functionaliteit.

2. **Groeiende, onoverzichtelijke `if`/`switch`-ketens**
   Een typisch symptoom van een OCP-schending is een methode met een steeds langer wordende `if`/`else if`- of `switch`-structuur, waarbij elk nieuw geval een extra tak toevoegt. Deze methode wordt na verloop van tijd moeilijk leesbaar en moeilijk te onderhouden.

3. **Schending van SRP als bijwerking**
   Zo'n klasse krijgt vaak ook een nieuwe reden om te veranderen telkens er een nieuw geval bijkomt — wat rechtstreeks ingaat tegen het Single Responsibility Principle. De klasse "weet" over steeds meer soorten gevallen, in plaats van verantwoordelijk te zijn voor één duidelijk afgebakende taak.

4. **Moeilijker te testen**
   Bij elke uitbreiding van de `if`/`switch`-keten moet je (in theorie) de volledige klasse opnieuw testen, ook al wilde je enkel nieuw gedrag toevoegen. Bestaande testen kunnen breken door een wijziging die eigenlijk niets met hen te maken had.

5. **Hogere kans op merge conflicts**
   Als meerdere ontwikkelaars tegelijk nieuwe gevallen moeten toevoegen aan dezelfde `if`/`switch`-structuur, werken ze noodgedwongen in hetzelfde stuk code, wat leidt tot conflicten bij het samenvoegen van hun wijzigingen.

## De oplossing: OCP toepassen

De oplossing bestaat erin om **abstractie te gebruiken** (een interface of abstracte klasse) in plaats van een concrete klasse die alle mogelijke gevallen zelf kent. Nieuw gedrag voeg je dan toe door een **nieuwe klasse te schrijven** die deze abstractie implementeert, zonder de bestaande code aan te raken.

Typisch patroon:

1. Definieer een interface (of abstracte klasse) die het gemeenschappelijke gedrag beschrijft.
2. Laat elke variant zijn eigen klasse zijn, die deze interface implementeert.
3. De code die met deze varianten werkt, doet dat via de interface — ze weet niets over de concrete implementaties.
4. Een nieuw geval toevoegen = een nieuwe klasse schrijven die de interface implementeert. Geen enkele bestaande klasse moet aangepast worden.

### Waarom dit de bovenstaande problemen oplost

- **Geen risico op regressies in bestaande code**: bestaande klassen worden niet aangeraakt wanneer je een nieuw geval toevoegt, dus bestaande, al geteste functionaliteit blijft gegarandeerd werken.
- **Geen groeiende `if`/`switch`-ketens**: er is geen centrale methode meer die alle gevallen kent — elk geval leeft in zijn eigen klasse.
- **SRP blijft gerespecteerd**: elke klasse blijft verantwoordelijk voor precies één geval, en heeft dus maar één reden om te veranderen.
- **Makkelijker te testen**: nieuwe functionaliteit zit in een nieuwe klasse, die apart en geïsoleerd getest kan worden, zonder bestaande testen te raken.
- **Minder merge conflicts**: verschillende ontwikkelaars kunnen tegelijk nieuwe klassen toevoegen zonder in hetzelfde bestand te moeten werken.

## Voorbeelden

### Voorbeeld 1 — OCP overtreden

```csharp
public class AreaCalculator
{
    public double CalculateArea(object shape)
    {
        if (shape is Circle circle)
        {
            return Math.PI * circle.Radius * circle.Radius;
        }
        else if (shape is Rectangle rectangle)
        {
            return rectangle.Width * rectangle.Height;
        }

        throw new ArgumentException("Onbekende vorm.");
    }
}

public class Circle
{
    public double Radius { get; set; }
}

public class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }
}
```

Als je nu een `Triangle` wil toevoegen, moet je verplicht de methode `CalculateArea` in `AreaCalculator` aanpassen en een extra `else if`-tak toevoegen. Deze klasse is dus **niet gesloten voor wijziging**: elke nieuwe vorm dwingt een aanpassing af aan bestaande, al geteste code.

### Voorbeeld 2 — OCP toepassen

```csharp
public interface IShape
{
    double CalculateArea();
}

public class Circle : IShape
{
    public double Radius { get; set; }

    public double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double CalculateArea()
    {
        return Width * Height;
    }
}

public class AreaCalculator
{
    public double CalculateArea(IShape shape)
    {
        return shape.CalculateArea();
    }
}
```

Wil je nu een driehoek toevoegen? Dan schrijf je gewoon een nieuwe klasse:

```csharp
public class Triangle : IShape
{
    public double Base { get; set; }
    public double Height { get; set; }

    public double CalculateArea()
    {
        return 0.5 * Base * Height;
    }
}
```

`AreaCalculator` moet hiervoor **niet** aangepast worden — hij werkt via de `IShape`-interface en weet niets over de concrete vormen. De klasse is dus **open voor uitbreiding** (nieuwe vormen toevoegen kan altijd) maar **gesloten voor wijziging** (bestaande code blijft ongewijzigd en dus veilig).

### Voorbeeld 3 — Een klein alledaags geval: kortingen

```csharp
// Overtreedt OCP: elke nieuwe klantcategorie vereist een aanpassing hier
public class DiscountCalculator
{
    public double CalculateDiscount(string customerType, double amount)
    {
        if (customerType == "Regular")
        {
            return amount * 0.0;
        }
        else if (customerType == "Silver")
        {
            return amount * 0.05;
        }
        else if (customerType == "Gold")
        {
            return amount * 0.10;
        }

        throw new ArgumentException("Onbekend klanttype.");
    }
}
```

```csharp
// Volgt OCP: een nieuwe klantcategorie = een nieuwe klasse
public interface IDiscountPolicy
{
    double CalculateDiscount(double amount);
}

public class RegularDiscountPolicy : IDiscountPolicy
{
    public double CalculateDiscount(double amount) => amount * 0.0;
}

public class SilverDiscountPolicy : IDiscountPolicy
{
    public double CalculateDiscount(double amount) => amount * 0.05;
}

public class GoldDiscountPolicy : IDiscountPolicy
{
    public double CalculateDiscount(double amount) => amount * 0.10;
}

public class DiscountCalculator
{
    public double CalculateDiscount(IDiscountPolicy policy, double amount)
    {
        return policy.CalculateDiscount(amount);
    }
}
```

Een nieuwe categorie zoals `"Platinum"` toevoegen betekent nu gewoon een nieuwe klasse `PlatinumDiscountPolicy` schrijven die `IDiscountPolicy` implementeert. `DiscountCalculator` blijft volledig ongewijzigd.

## Samenvatting

| Zonder OCP | Met OCP |
|---|---|
| Nieuw gedrag = bestaande klasse aanpassen | Nieuw gedrag = nieuwe klasse toevoegen |
| Groeiende `if`/`switch`-ketens | Elk geval in zijn eigen klasse |
| Risico op regressies bij elke uitbreiding | Bestaande, geteste code blijft ongewijzigd |
| Vaak ook een SRP-schending als bijwerking | Elke klasse blijft één duidelijke verantwoordelijkheid houden |

OCP betekent niet dat je nooit meer iets mag aanpassen aan je systeem. Het betekent dat je ontwerpt met **abstractie** zodat de meest voorkomende soort wijziging — een nieuw geval toevoegen — kan gebeuren via **nieuwe code**, zonder bestaande, werkende code te moeten openbreken.