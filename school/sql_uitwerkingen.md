# SQL uitwerkingen - Ruben Eekhof

- [SQL uitwerkingen - Ruben Eekhof](#sql-uitwerkingen---ruben-eekhof)
  * [Waarom staat er een ; achter elke query?](#waarom-staat-er-een---achter-elke-query-)
  * [Hoe werkt GROUP BY?](#hoe-werkt-group-by-)
    + [Aggregate Functions](#aggregate-functions)
  * [Wat is een SELF JOIN?](#wat-is-een-self-join-)
  * [Week 8](#week-8)
    + [Opdracht 6](#opdracht-6)
    + [Opdracht 7](#opdracht-7)
    + [Opdracht 8](#opdracht-8)
    + [Opdracht 9](#opdracht-9)

## Waarom staat er een ; achter elke query?

Een `;`in sql werkt in feite hetzelfde als in PHP het geeft het einde van de query aan. (Dit is niet verplicht)

## Hoe werkt GROUP BY?

In veel opgaven word gebruikt gemaakt van de `GROUP BY` functie in sql. Je kan makkelijk herkennen of je deze functie nodig hebt door goed te lezen, als het gaat over unieke groepen data dan is de kans groot dat je een group by gaat moeten gebruiken. Bijvoorbeeld het gemiddelde cijfer van alle mensen in een bepaalde klas. Neem als voorbeeld deze inschrijvings tabel:

| cursist | cursus | begindatum | evaluatie |
| ------- | ------ | ---------- | --------- |
| 7499    | FOR    | 1995-12-17 | 2         |
| 7499    | PLS    | 1996-09-11 | *NULL*    |
| 7566    | FOR    | 1996-02-05 | 3         |

Met een simpele select in combinatie met een group by kunnen we alle unieke cursussen ophalen:

```sql
SELECT * FROM inschrijving GROUP BY cursus;
```

Het handige is nu dus dat we deze group by kunnen gebruiken om bepaalde dingen te acherhalen, bijvoorbeeld het aantal cursisten in een bepaalde cursus:

```sql
SELECT COUNT(cursist), cursus, evaluatie
FROM inschrijving 
GROUP BY cursus;
```

Dit geeft het volgende resultaat:

| COUNT(cursist) | cursus | evaluatie |
| -------------- | ------ | --------- |
| 8              | FOR    | 2         |
| 4              | OAG    | 4         |
| 3              | PLS    | *NULL*    |
| 2              | REP    | 5         |
| 9              | S02    | 4         |

Nu voeren we dezelfde query uit maar zonder group by:

```sql
SELECT COUNT(cursist), cursus, evaluatie
FROM inschrijving;
```

Dit geeft het volgende resultaat:

| COUNT(cursist) | cursus | evaluatie |
| -------------- | ------ | --------- |
| 26             | FOR    | 2         |

### Aggregate Functions

Waarom geeft deze query maar 1 rij terug? Dit komt omdat functies zoals `COUNT()`, `SUM()`en `AVG()` aggregate functions zijn. Dit betekend dat ze een calculatie uitvoeren op meerdere waardes en maar 1 waarde teruggeven. Dat verklaard dus waarom de bovenste query maar 1 rij terug geeft.

Maar waarom geeft dezelfde query met een group by wel meerdere waardes terug?

Dit komt simpelweg omdat we nu deze `COUNT()` functie uitvoeren over elke groep die we hebben aangegeven in onze group by. 

## Wat is een SELF JOIN?

Een SELF JOIN komt voor wanneer je een tabel joint met zichzelf. Er bestaat geen uniek keywoord voor je gebruikt gewoon een normale JOIN. Maar waarom is dit nuttig? Laten we als voorbeeld deze medewerkers tabel nemen:

| mnr  | naam | voorl | functie | chef | gbdatum | maandsal | comm | afd  |
| ---- | ---- | ----- | ------- | ---- | ------- | -------- | ---- | ---- |

In deze tabel bestaat er een relatie tussen mnr en chef. We kunne hierop dus een simpele JOIN uitvoeren:

```sql
SELECT a.mnr, a.naam, a.chef
FROM medewerker AS a
JOIN medewerker AS b ON a.chef = b.mnr;
```

Wat doet dit precies? Bij een join voeg je de rijen samen van 2 of meer tabellen gebasseerd op een gerelateerde kolom in die tabellen. In dit geval hebben we maar 1 tabel dus makt hij een kopie van die tabel en voegt hij ze samen. Het is hier dus wel van belang om aliasen te gebruiken om duidelijk te maken welke kolommen je precies opvraagt.

## Week 8

### Opdracht 6

```
De directeur wil graag inzicht krijgen in de mate waarin de medewerkers meedoen aan de interne scholingsmogelijkheden. Hij vraagt daarom om een overzicht met per medewerker het aantal keren dat die medewerker zich voor een cursus heeft ingeschreven.
```

In deze query moeten we dus 2 kolommen ophalen, de mederwerker en het aantal keren dat hij zich voor een cursus heeft ingeschreven. Deze kolommen staan beide in de inschrijvingstabel:

| cursist | cursus | begindatum | evaluatie |
| ------- | ------ | ---------- | --------- |

Een makkelijke manier om een query te schrijven is om eerst alle rijen op te halen die de opdracht van je vraagt:

```sql
SELECT cursist 
FROM inschrijving;
```

Deze query geeft ons een lijst terug met alle cursisten, maar zoals je kan zien staan er dubbele cursisten in, dit kun je voorkomen met `DISTINCT` of door een group by uit te voeren op cursist. In dit geval weten we dat we ook nog moeten ophalen hoeveel elke mederwerker zich heeft ingeschreven voor een cursus dus kunnen we het beste een `GROUP BY`gebruiken.

```sql
SELECT cursist 
FROM inschrijving 
GROUP BY cursist;
```

Het enige wat we missen is het aantal keren dat een medewerker zich heeft ingeschreven voor een curses en dit kunnen we krijgen door te zien hoevaak een cursist voorkomt in de tabel. Als een cursist immers 2 keer voorkomt in de inschrijvings tabel dan heeft hij zich 2 keer ingeschreven. Hiervoor gebruik je de sql `COUNT()`functie. Dit ziet er als volgt uit:

```sql
SELECT cursist, COUNT(cursist) AS 'aantal inschrijvingen'
FROM inschrijving 
GROUP BY cursist;
```

### Opdracht 7

```sql
Geef een overzicht van de docenten die cursussen geven of hebben gegeven met per docent het medewerkersnummer, de naam en het aantal keren dat hij/zij voor een cursus staat of stond ingepland.
```

In deze query moeten we dus 3 kolommen ophalen: het mederwerkersnummer en de naam van de docent die een cursus gegeven of hebben gegeven en, het aantal keren dat zij voor het geven ervan staan ingeplant. Voor deze query hebben we 2 tabellen nodig:

De uitvoering tabel:

| cursus | begindatum | docent | locatie |
| ------ | ---------- | ------ | ------- |

De medewerker tabel:

| mnr  | naam | voorl | functie | chef | gbdatum | maandsal | comm | afd  |
| ---- | ---- | ----- | ------- | ---- | ------- | -------- | ---- | ---- |

Net zoals bij de vorige opdracht is het handig om te beginnen alle mogelijke kolommen op te halen, we hebben hiervoor een simpele join (of subquery) nodig omdat we 2 tabellen gebruiken:

```sql
SELECT m.naam, m.mnr 
FROM medewerker AS m JOIN uitvoering AS u ON u.docent = m.mnr;
```

Net als in de vorige query hebben we geen baat aan duplicate medewerkers dus gebruiken we een `GROUP BY`

```sql
SELECT m.naam, m.mnr 
FROM medewerker AS m JOIN uitvoering AS u ON u.docent = m.mnr 
GROUP BY m.mnr;
```

Nu moeten we er nog achterkomen hoevaak de docenten staan ingeplant. Net als bij inschrijving kunnen we tellen hoevaak de docent voorkomt in de uitvoering tabel. Dit doen we met de `COUNT()`functie:

```sql
SELECT m.naam, m.mnr, COUNT(u.docent) 
FROM medewerker AS m JOIN uitvoering AS u ON u.docent = m.mnr 
GROUP BY m.mnr;
```

### Opdracht 8

```
Geef van de cursussen die vanaf het jaar 1996 gegeven zijn per cursus het aantal cursisten dat die cursus in de loop van de tijd gevolgd heeft en het gemiddelde evaluatiecijfer (d.w.z. het gemiddelde van de evaluatiecijfers die aan de betreffende cursus gegeven zijn. Neem hierbij ook de records zonder ingevulde evaluatie mee).
```

In deze query moeten we dus 3 kolommen ophalen: per cursus het aantal cursisten die die cursus gevolgd heeft, het gemiddelde evaluatie cijfer en de naam van de cursus. Hiervoor hebben we alleen de inschrijvings tabel nodig:

| cursist | cursus | begindatum | evaluatie |
| ------- | ------ | ---------- | --------- |

Om te beginnen halen we de benodigde velden op:

```sql
SELECT cursus, cursist, evaluatie FROM inschrijving;
```

Aangezien we per cursus bepaalde data willen ophalen kunnen we het beste grouperen op cursus:

```sql
SELECT cursus, cursist, evaluatie FROM inschrijving GROUP BY cursus;
```

De volgende stap is het achterhalen van het aantal cursisten, dit kunnen we net zoals eerder doen door te tellen hoeveleel cursisten er in de inschrijvingstabel staan voor die bepaalde cursus met de `COUNT()`functie. De gemiddelde evaluatie kun je berekenen met de `AVG()`functie. We krijgen dan de volgende query:

```sql
SELECT cursus, COUNT(cursist) AS 'aantal cursisten', AVG(evaluatie) AS 'gem evaluatie' 
FROM inschrijving
GROUP BY cursus;
```

### Opdracht 9

```text
Geef een overzicht van de medewerkers die meer verdienen dan hun chef.
Geef op dit overzicht van deze medewerkers het medewerkersnummer, de naam, en het maandsalaris.
```

In deze query moeten we dus de volgende kolommen ophalen: het medewerkers nummer, de naam en hun maandsal van de werknemers die meer verdienen dan hun chef. We hebben hiervoor de medewerkers tabel nodig:

| mnr  | naam | voorl | functie | chef | gbdatum | maandsal | comm | afd  |
| ---- | ---- | ----- | ------- | ---- | ------- | -------- | ---- | ---- |

Er is iets bijzonders aan de hand met deze tabel want er bestaat een relatie tussen medewerker en chef. Dit betekent dat we voor deze query gebruik kunnen maken van een SELF JOIN (staat bovenin dit document uitgelegd):

```sql
SELECT a.mnr, a.naam, a.maandsal
FROM medewerker AS a
JOIN medewerker AS b ON a.chef = b.mnr;
```

Dit resultaat bevat veel dubbele waardes omdat als je `a.mnr` joint op `b.chef`je alle rijen pakt waarin deze waardes gelijk zijn van beide medewerkers tabellen. Alles wat je nu nog moet doen is alleen de salarissen laten zien met een hoger salaris dan hun chef. Dit kan je doen met een simpele where clausule:

``` sql
SELECT a.mnr, a.naam, a.maandsal
FROM medewerker AS a
JOIN medewerker AS b ON a.chef = b.mnr
WHERE a.maandsal > b.maandsal;
```

