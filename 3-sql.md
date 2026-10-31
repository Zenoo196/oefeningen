# 🗄️ 3. SQL: MySQL Workbench

[← terug naar overzicht](README.md) · [← 1. Tekening](1-tekening.md) · [← 2. PowerShell-script](2-powershell-script.md)

**Waar let de beoordelaar op (12 punten, 3 per criterium)**
| Criterium | Wat de beoordelaar wil zien | Zie |
|---|---|---|
| T1 Autorisatie configureren | Gebruiker aanmaken en de **juiste rechten** geven (niet meer dan nodig) | [§4](#4-autorisatie-gebruikers-en-rechten) |
| T2 Eenvoudige selectie-query's | `SELECT`, `WHERE`, `ORDER BY` | [§3](#3-uitgewerkte-queries) |
| T2 Complexe selectie-query's | subquery (`AVG`, `NOT IN`), `JOIN`, `GROUP BY`, `COUNT` | [§5](#5-complexe-querys) |
| T3 Gegevens aanpassen | `INSERT`, `UPDATE`, `DELETE` | [§6](#6-gegevens-aanpassen-insert-update-delete) |

> **Autorisatie** kwam in de oefening niet voor, maar staat wel op het beoordelingsformulier. Oefen §4 dus zeker!

Bij **elke** query in je document: SQL-code + screenshot van het resultaat + 1–2 zinnen uitleg ("20 rows returned", "1 row affected").

---

## 1. MySQL Workbench basis
| Wat | Hoe |
|---|---|
| Verbinden | Startscherm → klik op **Local instance MySQL80** → wachtwoord root |
| Database kiezen | Dubbelklik op de database links bij **SCHEMAS** (wordt vet) of `USE ritmania;` |
| Query uitvoeren (alles/selectie) | `Ctrl + Shift + Enter` of het **bliksem-icoon** |
| Alleen regel bij cursor | `Ctrl + Enter` of bliksem met **I** |
| SQL-bestand openen | File → **Open SQL Script...** (`Ctrl + Shift + O`) |
| Schemas verversen | rechtsklik bij SCHEMAS → **Refresh All** |
| Foutmelding lezen | onderaan bij **Action Output**, rood kruisje |
| Kolomnamen zien | klik op pijltje bij de tabel → Columns, of `DESCRIBE memoryusage;` |

## 2. Database klaarzetten en Results.txt importeren

1. Open het aangeleverde script (Bijlagen Kandidaat) dat de database **ritmania** maakt (tabellen `artist`, `album`, `memoryusage`) en voer het uit.
2. Refresh → `ritmania` staat links.
3. File → Open SQL Script → **Results.txt** (zet bestandstype op *All files*).
4. Voer uit. Je krijgt waarschijnlijk:

**Error Code: 1064** *You have an error in your SQL syntax*
- **Oorzaak:** de laatste regel van Results.txt eindigt op een **komma**:
  `('SRV01',5023660,20261012153824),`
- **Oplossing:** vervang de laatste `,` door `;`:
  `('SRV01',5023660,20261012153824);`
- Opnieuw uitvoeren → `20 row(s) affected`.

**Error Code: 1146** *Table 'ritmania.country' doesn't exist*
- **Oorzaak:** verkeerde database actief.
- **Oplossing:** `USE world;` (of dubbelklik op `world`).

**Error Code: 1175** *You are using safe update mode...*
- **Oorzaak:** `UPDATE`/`DELETE` zonder `WHERE` op de primary key.
- **Oplossing:** gebruik `WHERE ID > 0` / `WHERE ID = 5`, of zet het even uit:
  `SET SQL_SAFE_UPDATES = 0;`

**Error Code: 1046** *No database selected* → `USE ritmania;`

**Error Code: 1054** *Unknown column* → kolomnaam verkeerd gespeld, check met `DESCRIBE tabel;`

> Schrijf deze fouten + oplossing **in je document**. Dat laat probleemoplossend vermogen zien (dat miste bij mijn oefening).

### Hoe Results.txt eruitziet
```sql
DELETE FROM MemoryUsage WHERE ID > 0;
INSERT INTO MemoryUsage (Hostname,FreeMemory,Date) VALUES
('SRV01',5006236,20261012153725),
('SRV01',5007000,20261012153728),
...
('SRV01',5023660,20261012153824);   -- laatste regel met ; !
```

---

## 3. Uitgewerkte queries

Database **ritmania** voor 1 t/m 5, database **world** voor 6.

**1. Laat alle velden van de tabel MemoryUsage zien**
```sql
SELECT *
FROM memoryusage;
```
*Uitleg:* `*` = alle kolommen. Resultaat: alle 20 metingen.

**2. Laat hostname en freememory zien, gesorteerd op freememory van laag naar hoog**
```sql
SELECT hostname, freememory
FROM memoryusage
ORDER BY freememory;
```
*Uitleg:* `ORDER BY` sorteert standaard oplopend (`ASC`). Hoog naar laag = `ORDER BY freememory DESC`.

**3. Laat date en freememory zien waarvoor freememory hoger is dan 5000000**
```sql
SELECT date, freememory
FROM memoryusage
WHERE freememory > 5000000;
```
*Uitleg:* `WHERE` filtert de rijen; alleen metingen boven 5.000.000 KB.

**4. Voeg een nieuw record toe (zelf waarden verzinnen)**
```sql
INSERT INTO memoryusage (hostname, freememory, date)
VALUES ('MyNewPC', 4567098, 20221013);
```
*Uitleg:* `ID` hoef je niet in te vullen (auto_increment). Resultaat: `1 row(s) affected`. Controleer met query 1.

**5. Laat date en freememory zien waarvoor freememory kleiner is dan het gemiddelde**
```sql
SELECT date, freememory
FROM memoryusage
WHERE freememory < (SELECT AVG(freememory) FROM memoryusage);
```
*Uitleg:* eerst wordt de **subquery** tussen haakjes uitgevoerd: die berekent het gemiddelde vrije geheugen van alle records. Daarna vergelijkt de buitenste query elke rij met dat getal en toont alleen de rijen die eronder zitten.

**6. (database world) Laat code, name, region en localname zien van landen die niet in de tabel City voorkomen**
```sql
USE world;
SELECT code, name, region, localname
FROM country
WHERE code NOT IN (SELECT countrycode FROM city);
```
*Uitleg:* de subquery geeft een lijst van alle landcodes die in `city` staan. `NOT IN` houdt alleen de landen over waarvan de `code` **niet** in die lijst staat, dus landen zonder steden (resultaat: 7 landen).

---

## 4. Autorisatie: gebruikers en rechten

### Met SQL
```sql
-- gebruiker aanmaken
CREATE USER 'monitor'@'localhost' IDENTIFIED BY 'Welkom123!';

-- rechten geven: alleen wat nodig is
GRANT SELECT, INSERT ON ritmania.memoryusage TO 'monitor'@'localhost';

-- alle rechten op één database
GRANT ALL PRIVILEGES ON ritmania.* TO 'beheerder'@'localhost';

-- alleen lezen op hele database
GRANT SELECT ON ritmania.* TO 'lezer'@'localhost';

-- rechten actief maken
FLUSH PRIVILEGES;

-- controleren
SHOW GRANTS FOR 'monitor'@'localhost';
SELECT user, host FROM mysql.user;

-- recht afpakken
REVOKE INSERT ON ritmania.memoryusage FROM 'monitor'@'localhost';

-- wachtwoord wijzigen
ALTER USER 'monitor'@'localhost' IDENTIFIED BY 'NieuwWw456!';

-- gebruiker verwijderen
DROP USER 'monitor'@'localhost';
```

| Onderdeel | Betekenis |
|---|---|
| `'monitor'@'localhost'` | gebruiker mag alleen vanaf de server zelf inloggen |
| `'monitor'@'%'` | gebruiker mag vanaf **elke** computer inloggen |
| `'monitor'@'192.168.1.%'` | alleen vanaf dat netwerk |
| `ritmania.*` | alle tabellen in ritmania |
| `ritmania.memoryusage` | alleen die tabel |
| `*.*` | alles op de server (niet doen tenzij gevraagd) |
| Rechten | `SELECT` lezen · `INSERT` toevoegen · `UPDATE` wijzigen · `DELETE` verwijderen · `CREATE`, `DROP`, `ALTER` structuur · `ALL PRIVILEGES` alles |

### Met de Workbench (klikken)
1. Links onderaan tab **Administration** → **Users and Privileges**.
2. **Add Account** → Login Name, *Limit to Hosts Matching* = `localhost`, wachtwoord 2x.
3. Tab **Administrative Roles**: niks aanvinken (tenzij gevraagd).
4. Tab **Schema Privileges** → **Add Entry...** → *Selected schema* → kies `ritmania` → OK.
5. Vink de rechten aan (bijv. `SELECT`, `INSERT`) → **Apply**.

### Testen (screenshot voor je document!)
1. Startscherm → **+** naast MySQL Connections → nieuwe verbinding met username `monitor`.
2. Inloggen → probeer een toegestane query (`SELECT`) → werkt.
3. Probeer een niet-toegestane query (bijv. `DELETE`) → **Error 1142** *command denied to user*. Zo laat je zien dat de rechten kloppen.

> Leg in je document uit **waarom** je deze rechten geeft: "Het monitorscript hoeft alleen metingen toe te voegen en te lezen, daarom krijgt de gebruiker alleen SELECT en INSERT (principe van minimale rechten)."

---

## 5. Complexe query's

### Database ritmania (tabellen artist, album, memoryusage)
`artist (artist_id, artist_name, artist_country)` · `album (album_id, album_name, album_year, artist_id)`

```sql
-- albums met de naam van de artiest (JOIN)
SELECT album.album_name, album.album_year, artist.artist_name
FROM album
JOIN artist ON album.artist_id = artist.artist_id;

-- aantal albums per artiest
SELECT artist.artist_name, COUNT(album.album_id) AS aantal_albums
FROM artist
JOIN album ON album.artist_id = artist.artist_id
GROUP BY artist.artist_name;

-- artiesten zonder album (NOT IN)
SELECT artist_name
FROM artist
WHERE artist_id NOT IN (SELECT artist_id FROM album);

-- artiesten zonder album (LEFT JOIN, zelfde resultaat)
SELECT artist.artist_name
FROM artist
LEFT JOIN album ON album.artist_id = artist.artist_id
WHERE album.album_id IS NULL;

-- artiesten uit Nederland, alfabetisch
SELECT artist_name FROM artist
WHERE artist_country = 'Netherlands'
ORDER BY artist_name;

-- hoogste, laagste en gemiddelde vrije geheugen
SELECT MAX(freememory) AS hoogste, MIN(freememory) AS laagste, AVG(freememory) AS gemiddeld
FROM memoryusage;

-- meting met het meeste vrije geheugen
SELECT * FROM memoryusage ORDER BY freememory DESC LIMIT 1;

-- metingen tussen twee waarden
SELECT * FROM memoryusage WHERE freememory BETWEEN 5000000 AND 5010000;

-- aantal metingen per hostname
SELECT hostname, COUNT(*) AS aantal FROM memoryusage GROUP BY hostname;
```

### Database world (tabellen country, city, countrylanguage)
```sql
-- landen in Europa, meeste inwoners eerst
SELECT name, population FROM country
WHERE continent = 'Europe'
ORDER BY population DESC;

-- landen met meer dan 100 steden
SELECT countrycode, COUNT(*) AS aantal_steden
FROM city
GROUP BY countrycode
HAVING COUNT(*) > 100;

-- landen met meer inwoners dan gemiddeld
SELECT name, population FROM country
WHERE population > (SELECT AVG(population) FROM country);

-- stad + landnaam (JOIN)
SELECT city.name AS stad, country.name AS land
FROM city
JOIN country ON city.countrycode = country.code;

-- steden in Nederland
SELECT name, district, population FROM city WHERE countrycode = 'NLD';

-- landen die met 'N' beginnen
SELECT name FROM country WHERE name LIKE 'N%';
```

| Wat | Syntax |
|---|---|
| Sorteren | `ORDER BY kolom` (oplopend), `ORDER BY kolom DESC` (aflopend) |
| Meerdere voorwaarden | `WHERE a > 5 AND b = 'x'`, `OR`, `NOT` |
| Tekst met wildcard | `LIKE 'A%'` (begint met A), `'%a'` (eindigt), `'%a%'` (bevat) |
| Tussen | `BETWEEN 10 AND 20` |
| In lijst | `IN ('NLD','BEL')`, `NOT IN (subquery)` |
| Leeg | `IS NULL`, `IS NOT NULL` |
| Functies | `COUNT(*)`, `SUM()`, `AVG()`, `MIN()`, `MAX()` |
| Groeperen | `GROUP BY kolom` + filter op groep met `HAVING` |
| Aantal rijen | `LIMIT 5` |
| Kolom hernoemen | `AS nieuwe_naam` |
| Koppelen | `JOIN tabel2 ON tabel1.kolom = tabel2.kolom` |
| Unieke waarden | `SELECT DISTINCT kolom` |

**Volgorde van een query (altijd zo):**
`SELECT` → `FROM` → `JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `ORDER BY` → `LIMIT`

---

## 6. Gegevens aanpassen (INSERT, UPDATE, DELETE)

```sql
-- toevoegen
INSERT INTO memoryusage (hostname, freememory, date)
VALUES ('MyNewPC', 4567098, 20261102);

-- meerdere tegelijk
INSERT INTO artist (artist_name, artist_country) VALUES
('Racoon', 'Netherlands'),
('Coldplay', 'United Kingdom');

-- wijzigen (voorbeeld: in de data staat 'Get Your Wings' met jaar 1874, moet 1974 zijn)
UPDATE album
SET album_year = 1974
WHERE album_name = 'Get Your Wings';

-- wijzigen op ID (veilig)
UPDATE memoryusage SET hostname = 'SRV02' WHERE ID = 21;

-- verwijderen
DELETE FROM memoryusage WHERE hostname = 'MyNewPC';

-- alles verwijderen (zoals in Results.txt)
DELETE FROM memoryusage WHERE ID > 0;
```

> **Nooit** `UPDATE` of `DELETE` zonder `WHERE`, dan pas je **alle** rijen aan. Check eerst met een `SELECT` met dezelfde `WHERE` welke rijen je raakt. Na afloop nog een `SELECT` als bewijs (screenshot).

---

## 7. Tabel zelf maken (als dat gevraagd wordt)
```sql
CREATE DATABASE IF NOT EXISTS ritmania;
USE ritmania;

CREATE TABLE IF NOT EXISTS DiskUsage (
  ID        INT NOT NULL AUTO_INCREMENT,
  Hostname  VARCHAR(50),
  FreeSpace BIGINT,
  Date      DATETIME,
  PRIMARY KEY (ID)
);

DESCRIBE DiskUsage;          -- structuur bekijken
SHOW TABLES;                 -- alle tabellen
ALTER TABLE DiskUsage ADD COLUMN Drive VARCHAR(2);   -- kolom toevoegen
DROP TABLE DiskUsage;        -- tabel weg (pas op!)
```
| Datatype | Voor |
|---|---|
| `INT` | getal (tot ~2 miljard) |
| `BIGINT` | groot getal (bytes schijfruimte!) |
| `VARCHAR(50)` | tekst tot 50 tekens |
| `DATETIME` | datum + tijd (`20261102093000` werkt ook) |
| `DATE` | alleen datum |
| `DECIMAL(5,2)` | kommagetal |

---

## 8. Checklist vóór inleveren
- [ ] Juiste database actief bij elke query (`USE ...`)
- [ ] Results.txt geïmporteerd, fout 1064 beschreven + opgelost
- [ ] Elke query: opdracht + SQL + screenshot + uitleg
- [ ] Subquery's (`AVG`, `NOT IN`) uitgelegd in eigen woorden
- [ ] `INSERT`/`UPDATE`/`DELETE` gecontroleerd met een `SELECT` erna
- [ ] Gebruiker aangemaakt met **alleen de nodige rechten** + getest + uitgelegd waarom
- [ ] Titel en korte inleiding in je document
