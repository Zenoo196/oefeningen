# 🧰 Oefeningen: scripting en databases

Mijn eigen naslag voor drie onderdelen: een **tekening** (flowchart) maken, een **PowerShell-script** afmaken en **SQL** in MySQL.
Alles is uitgewerkt aan de hand van het voorbeeld **MonitorFreeMemory**, met de fouten die ik eerder maakte erbij. Een nieuwe opdracht heeft meestal **dezelfde opbouw**, alleen een ander onderwerp.

## 📚 Onderdelen

| | Onderdeel | Wat doe je | Pagina |
|---|---|---|---|
| ✏️ | **1. Tekening** | Flowchart in **Visio** + ontwerpsjabloon invullen | 👉 [1-tekening.md](1-tekening.md) |
| ⚡ | **2. PowerShell-script** | Script aanvullen bij de `OPDRACHT:`-blokken, testen en documenteren | 👉 [2-powershell-script.md](2-powershell-script.md) |
| 🗄️ | **3. SQL** | Results.txt importeren, query's, gebruikers en rechten | 👉 [3-sql.md](3-sql.md) |

## 🔎 Snel naar...
- **Tekening:** [Symbolen](1-tekening.md#3-symbolen) · [Signaalwoorden](1-tekening.md#4-signaalwoorden-van-tekst-naar-symbool) · [Lussen en keuzes](1-tekening.md#5-de-vier-patronen-die-altijd-terugkomen) · [Voorbeeldflowchart](1-tekening.md#6-uitgewerkt-voorbeeld-monitorfreememory) · [Sjabloon ingevuld](1-tekening.md#7-ontwerpsjabloon-ingevuld-voorbeeld-monitorfreememory)
- **PowerShell:** [Templates per OPDRACHT](2-powershell-script.md#2-templates-per-soort-opdracht) · [Volledig script](2-powershell-script.md#3-uitgewerkt-voorbeeld-monitorfreememory) · [Varianten (schijf/CPU/ping)](2-powershell-script.md#4-varianten-ander-onderwerp-zelfde-opbouw) · [Spiekbrief](2-powershell-script.md#5-powershell-spiekbrief) · [Testtabel](2-powershell-script.md#6-testen-en-documenteren)
- **SQL:** [Foutmeldingen](3-sql.md#2-database-klaarzetten-en-resultstxt-importeren) · [Uitgewerkte query's](3-sql.md#3-uitgewerkte-queries) · [Gebruikers en rechten](3-sql.md#4-autorisatie-gebruikers-en-rechten) · [JOIN/GROUP BY](3-sql.md#5-complexe-querys) · [UPDATE/DELETE](3-sql.md#6-gegevens-aanpassen-insert-update-delete)

## 🧭 Zo pak ik het aan
1. **Lees de hele opdracht** en kijk in de map met bijlagen wat je krijgt (script, SQL-script, sjabloon).
2. **Tekening eerst**: nummer de stappen in de opdrachttekst, zoek de signaalwoorden, teken in Visio, vul het sjabloon in.
3. **Script**: zoek alle `OPDRACHT:` met `Ctrl + F`, vul ze één voor één in, test elk pad, maak screenshots.
4. **SQL**: database aanmaken, je eigen Results.txt importeren (laatste komma → `;`), query's + gebruiker/rechten.
5. **Overal**: screenshot + uitleg in eigen woorden. Laat zien dat je het **snapt**.

## ⚠️ Mijn top-5 fouten (niet nog een keer!)
1. Ontwerpsjabloon **leeg** gelaten, terwijl het net zo zwaar telt als de tekening.
2. Invoerlussen niet als **ruit met terugpijl** getekend.
3. `param()` gebruikt in plaats van `$args[0]` / `$args[1]` → OPDRACHT leek leeg.
4. Header (auteur/datum) niet aangepast.
5. Foutmeldingen die ik oploste (1064, 1146) niet **beschreven** in mijn SQL-document.

## ✅ Waar wordt op gelet
| ✏️ Tekening (9 pt) | ⚡ Script (15 pt) | 🗄️ SQL (12 pt) |
|---|---|---|
| Eisen en wensen beschreven | Juiste processen gekozen | **Gebruikers en rechten** instellen |
| Ontwerp doet wat gevraagd is | Onderdelen en relaties vastgelegd | Eenvoudige selectie-query's |
| Keuzes duidelijk uitgelegd | Script werkt | Complexe selectie-query's |
| | Lussen, variabelen, keuzes goed gebruikt en begrepen | Toevoegen/wijzigen/verwijderen |
| | Documentatie volledig en correct | |

Per punt: 0 = niet, 1 = enigszins, 2 = grotendeels, 3 = volledig.
