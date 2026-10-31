# ⚡ 2. PowerShell-script: aanvullen en testen

[← terug naar overzicht](README.md) · [← 1. Tekening](1-tekening.md) · [3. SQL →](3-sql.md)

**Waar let de beoordelaar op (15 punten, 3 per criterium)**
| Criterium | Wat de beoordelaar wil zien |
|---|---|
| Te automatiseren processen kiezen | Je script doet wat de opdracht vraagt. |
| Onderdelen en relaties vastleggen | Script en flowchart passen bij elkaar; header + commentaar kloppen. |
| Script/code werkend | Het script draait **zonder fouten**, met en zonder argumenten. |
| Basisconcepten | Variabelen, `If`, `While`, `For`, argumenten zijn goed gebruikt en je **laat zien dat je het snapt** (commentaar!). |
| Documenteren | Testdocument: technisch, volledig, correct. Testtabel + screenshots. |

> Onder **elke** `OPDRACHT:` moet zichtbare code of uitleg staan. Een leeg OPDRACHT-blok = punten kwijt.

---

## 1. Werkwijze
1. Open het script in **Visual Studio Code** of **PowerShell ISE** (staat op de server).
2. Zoek met `Ctrl + F` naar `OPDRACHT` en loop ze één voor één af.
3. Pas de **header** aan (auteur, datum, versie).
4. Opslaan → in PowerShell naar de map gaan → script draaien:
   ```powershell
   cd C:\Users\Administrator\Desktop      # of waar het script staat
   .\MonitorFreeMemory.ps1                # zonder argumenten
   .\MonitorFreeMemory.ps1 10 test2.txt   # met argumenten
   ```
5. Foutmelding "running scripts is disabled"? Eenmalig:
   ```powershell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
   ```
6. Elk pad testen en screenshot maken (`Win + Shift + S`), zie [§6 Testen](#6-testen-en-documenteren).

---

## 2. Templates per soort OPDRACHT

### OPDRACHT: header invullen
```powershell
<#
  Scriptnaam:  MonitorFreeMemory.ps1
  Functie   :  Dit script monitort het vrije geheugen van de machine waarop het script uitgevoerd wordt.
               De resultaten worden weggeschreven naar een resultatenfile als uit te voeren SQL query.
  Auteur    :  Jafeth Worm
  Versie    :  1.1
  Datum     :  2 november 2026

  Verwachte argumenten: aantal seconden (numeriek) en naam van de output file (tekst)
#>
```

### OPDRACHT: "zorg dat default waarden voor de variabelen worden ingesteld"
```powershell
$seconds    = 5               # standaard aantal seconden
$ResultFile = 'Results.txt'   # standaard naam resultatenbestand
$MachineName = hostname       # naam van deze computer
```

### OPDRACHT: "controleer of als (eerste/tweede) argument ... wordt ingevoerd, zo ja vervang de default"
```powershell
# Is het eerste argument een getal? Dan vervangt het de standaard seconden
If ($args[0] -Is [int]) {$seconds = $args[0]}

# Is het tweede argument tekst? Dan vervangt het de standaard bestandsnaam
If ($args[1] -Is [string]) {$ResultFile = $args[1]}
```
- `$args[0]` = eerste woord na de scriptnaam, `$args[1]` = tweede, enz.
- **Gebruik geen `param()` als de opdracht over argumenten/`$args` gaat.** Met `param()` blijft `$args` leeg. (Dat deed ik bij de oefening, daardoor stond er onder de OPDRACHT niks.)
- Alternatief als je het simpel wilt:
  ```powershell
  If ($args.Count -ge 1) {$seconds = $args[0]}
  If ($args.Count -ge 2) {$ResultFile = $args[1]}
  ```

### OPDRACHT: "vraag net zolang tot de gebruiker 1, 2 of 9 invoert" (max 3 regels)
```powershell
While (($answer -ne "1") -And ($answer -ne "2") -And ($answer -ne "9")) {
  $answer = Read-Host -Prompt "Please make your choice"
}
```
Korter (mag ook):
```powershell
While ($answer -notin "1","2","9") {
  $answer = Read-Host -Prompt "Please make your choice"
}
```
Met `Do...Until` (vraagt minstens één keer, dan heb je de `Read-Host` erboven niet nodig):
```powershell
Do {
  $answer = Read-Host -Prompt "Please make your choice"
} Until ($answer -in "1","2","9")
```
> Let op `-And`: de lus loopt door **zolang** het antwoord niet 1 **en** niet 2 **en** niet 9 is. Met `-Or` krijg je een oneindige lus!

### OPDRACHT: "leg in 1 commentaarblok uit wat de volgende regels doen"
Commentaarblok = `<# ... #>`. Eén regel = `#`.
```powershell
<#  Deze 4 regels tekenen een streep onder de melding:
    - $LengthLineOut krijgt het aantal tekens van de melding ($LineOut).
    - De For-lus loopt van 1 tot dat aantal en schrijft elke keer één "=" zonder nieuwe regel.
    - De laatste Write-Host sluit de regel af.
    Zo is de streep van "="-tekens precies even lang als de melding erboven.
#>
$LengthLineOut = $LineOut.Length
For($i=1;$i -le $LengthLineOut;$i++){
  Write-Host -NoNewline "="
}
Write-Host "`r"
```
Tip: schrijf in je commentaar **wat** het doet én **waarom**. Noem de variabelen bij naam.

### OPDRACHT: "zorg dat de gebruiker iets bewegends te zien krijgt"
Aftellen met een `For`-lus (netter dan drie gekopieerde regels):
```powershell
For ($i = 3; $i -ge 1; $i--) {
  Write-Host -NoNewline "> Next check in $i `r"   # `r = terug naar begin regel
  Start-Sleep -Seconds 1
}
```
Draaiend streepje:
```powershell
ForEach ($c in '|','/','-','\','|','/','-','\') {
  Write-Host -NoNewline "`rBezig... $c"
  Start-Sleep -Milliseconds 250
}
```
Puntjes erbij:
```powershell
For ($i = 1; $i -le 5; $i++) {
  Write-Host -NoNewline "."
  Start-Sleep -Milliseconds 400
}
Write-Host ""
```
Voortgangsbalk:
```powershell
For ($i = 0; $i -le 100; $i += 20) {
  Write-Progress -Activity "Wachten op volgende meting" -PercentComplete $i
  Start-Sleep -Milliseconds 400
}
```

### OPDRACHT: "keuze verwerken" (1 aanvullen / 2 overschrijven / 9 stoppen)
```powershell
If ($answer -Eq '9') {
  Exit                                             # stop het script
}
If ($answer -Eq '2') {
  Set-Content -Path $ResultFile -Value "DELETE FROM MemoryUsage WHERE ID > 0;"   # overschrijft
  Add-Content -Path $ResultFile -Value "INSERT INTO MemoryUsage (Hostname,FreeMemory,Date) VALUES"
}
# bij '1' gebeurt niks: de resultaten worden later toegevoegd
```
Met `Switch` (mag ook):
```powershell
Switch ($answer) {
  '1' { Write-Host "Resultaten worden toegevoegd." }
  '2' { Set-Content -Path $ResultFile -Value "DELETE FROM MemoryUsage WHERE ID > 0;"
        Add-Content -Path $ResultFile -Value "INSERT INTO MemoryUsage (Hostname,FreeMemory,Date) VALUES" }
  '9' { Exit }
}
```

### OPDRACHT: "vraag of het bestand geopend moet worden (y/n)"
```powershell
$answer = Read-Host -Prompt "`nDo you want to open it (y/n)?"
While (($answer -ne "y") -And ($answer -ne "n")) {
  $answer = Read-Host -Prompt "Please make your choice (y/n)"
}
If ($answer -Eq 'y') {Invoke-Item $ResultFile}
Else {Write-Host "Het bestand wordt niet geopend."}      # nette melding, geen "Dan niet joh..."
```

### OPDRACHT: tijdlus (zolang de tijd niet om is)
```powershell
$stopAt = (Get-Date).AddSeconds($seconds)      # of: (Get-Date) + (New-TimeSpan -Seconds $seconds)
While ((Get-Date) -Lt $stopAt) {
  # meten, tonen, wegschrijven, iets bewegends
}
```

### OPDRACHT: tellerlus (x keer herhalen)
```powershell
$teller = 0
While ($teller -lt $aantal) {
  # doe iets
  $teller++                    # vergeet dit niet, anders oneindige lus!
}
```

---

## 3. Uitgewerkt voorbeeld: MonitorFreeMemory

Gebaseerd op de docent-uitwerking (Bijlage 2), met de verbeteringen uit mijn feedback.

```powershell
<#
  Scriptnaam:  MonitorFreeMemory.ps1
  Functie   :  Dit script monitort het vrije geheugen van de machine waarop het script uitgevoerd wordt.
               De resultaten worden weggeschreven naar een resultatenfile als uit te voeren SQL query.
  Auteur    :  Jafeth Worm
  Versie    :  1.1
  Datum     :  2 november 2026

  Verwachte argumenten: aantal seconden (numeriek) en naam van de output file (tekst)
#>
clear

# OPDRACHT: default waarden
$seconds     = 5
$ResultFile  = 'Results.txt'
$MachineName = hostname

# Eerste argument (getal) vervangt de standaard seconden
If ($args[0] -Is [int]) {$seconds = $args[0]}

# OPDRACHT: tweede argument (tekst) vervangt de standaard bestandsnaam
If ($args[1] -Is [string]) {$ResultFile = $args[1]}

# Bestaat het resultatenbestand al?
If (Test-Path -LiteralPath $ResultFile) {
  Write-Host -ForegroundColor Red "WARNING: " -NoNewline
  Write-Host "$ResultFile already exists. What do you want to do?"
  Write-Host "[1] Add the results to this file"
  Write-Host "[2] Delete it and start with an empty one"
  Write-Host "[9] Quit"
  $answer = Read-Host -Prompt "Please make your choice"

  # OPDRACHT: invoerlus, net zolang vragen tot 1, 2 of 9
  While (($answer -ne "1") -And ($answer -ne "2") -And ($answer -ne "9")) {
    $answer = Read-Host -Prompt "Please make your choice"
  }

  If ($answer -Eq '2') {
    Set-Content -path $ResultFile -value "DELETE FROM MemoryUsage WHERE ID > 0;"
    Add-Content -path $ResultFile -value "INSERT INTO MemoryUsage (Hostname,FreeMemory,Date) VALUES"
  }
  If ($answer -Eq '9') {
    Exit
  }
}
Else {
  # Bestand bestaat nog niet: nieuw bestand met de eerste SQL-regels
  Set-Content -path $ResultFile -value "DELETE FROM MemoryUsage WHERE ID > 0;"
  Add-Content -path $ResultFile -value "INSERT INTO MemoryUsage (Hostname,FreeMemory,Date) VALUES"
}

# Stoptijd bepalen
$aantalSeconden = New-TimeSpan -Seconds $seconds
$stopAt = (Get-Date) + $aantalSeconden

clear
$LineOut = "Checking free virtual memory on $MachineName for $seconds seconds"
Write-Host $LineOut
<#  OPDRACHT: uitleg
    Deze regels bepalen de lengte van de melding ($LengthLineOut) en tekenen daaronder
    met een For-lus precies zoveel "="-tekens, als onderstreping van de melding.
#>
$LengthLineOut = $LineOut.Length
For($i=1;$i -le $LengthLineOut;$i++){
  Write-Host -NoNewline "="
}
Write-Host "`r"

# Tijdlus: meten zolang de stoptijd niet bereikt is
While ((Get-Date) -Lt $stopAt) {
  $FreeMemory = Get-CimInstance Win32_OperatingSystem | Select-Object FreePhysicalMemory
  ForEach ($entry in $FreeMemory){
    $TimeStamp = Get-Date -Format "yyyyMMddHHmmss"
    $MemValue  = $entry.FreePhysicalMemory
    $NewLine   = "('$MachineName',$MemValue,$TimeStamp),"
    Add-Content -Path $ResultFile -Value $NewLine
    $TimeStamp = Get-Date
    Write-Host "Free memory at ${TimeStamp}:" $entry.FreePhysicalMemory
  }

  # OPDRACHT: iets bewegends
  For ($i = 3; $i -ge 1; $i--) {
    Write-Host -NoNewline "> Next check in $i `r"
    Start-Sleep -Seconds 1
  }
}

clear
Write-Host -NoNewline "Monitoring process is done. Results can be found in "
Write-Host -ForegroundColor Red $pwd\$ResultFile

# Invoerlus y/n
$answer = Read-Host -Prompt "`nDo you want to open it (y/n)?"
While (($answer -ne "y") -And ($answer -ne "n")) {
  $answer = Read-Host -Prompt "Please make your choice (y/n)"
}
If ($answer -Eq 'y') {Invoke-Item $ResultFile}
Else {Write-Host "Het bestand wordt niet geopend."}

Write-Host "====================================`nScript finished at", (Get-Date), "`n"
```

**Wat ik fout deed op de oefening (niet nog een keer doen!)**
- `param()` gebruikt i.p.v. `$args[0]`/`$args[1]` → OPDRACHT "tweede argument" leek leeg.
- Header niet aangepast (auteur Olaf Ritman, 2022).
- "Dan niet joh..." → **nette** melding gebruiken.
- `$startAt` werd nergens gebruikt (mag weg).

---

## 4. Varianten: ander onderwerp, zelfde opbouw

Een nieuw script is vaak een ander onderwerp met **dezelfde opbouw**. Je hoeft dan alleen het **meet-commando** en de **tabelnaam** te veranderen.

| Wat meten | Commando | Waarde |
|---|---|---|
| Vrij geheugen (KB) | `(Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory` | getal |
| Totaal geheugen | `(Get-CimInstance Win32_OperatingSystem).TotalVisibleMemorySize` | getal |
| Vrije schijfruimte C: | `(Get-CimInstance Win32_LogicalDisk -Filter "DeviceID='C:'").FreeSpace` | bytes |
| CPU-belasting % | `(Get-CimInstance Win32_Processor).LoadPercentage` | getal |
| Aantal processen | `(Get-Process).Count` | getal |
| Draait een proces? | `Get-Process -Name notepad -ErrorAction SilentlyContinue` | leeg = niet |
| Status service | `(Get-Service -Name Spooler).Status` | Running/Stopped |
| Ping | `Test-Connection 8.8.8.8 -Count 1 -Quiet` | True/False |
| Ping-tijd (ms) | `(Test-Connection 8.8.8.8 -Count 1).ResponseTime` (PS 5) of `.Latency` (PS 7) | getal |
| Ingelogde gebruiker | `$env:USERNAME` | tekst |
| Datum/tijd voor SQL | `Get-Date -Format "yyyyMMddHHmmss"` | `20261102093000` |

**Voorbeeld: meetblok voor schijfruimte**
```powershell
$Free      = (Get-CimInstance Win32_LogicalDisk -Filter "DeviceID='C:'").FreeSpace
$TimeStamp = Get-Date -Format "yyyyMMddHHmmss"
Add-Content -Path $ResultFile -Value "('$MachineName',$Free,$TimeStamp),"
Write-Host "Free disk space at $(Get-Date): $Free bytes"
```
Eerste regels van het bestand dan:
```powershell
Set-Content -Path $ResultFile -Value "DELETE FROM DiskUsage WHERE ID > 0;"
Add-Content -Path $ResultFile -Value "INSERT INTO DiskUsage (Hostname,FreeSpace,Date) VALUES"
```
> Tekst in SQL tussen **enkele quotes** (`'SRV01'`), getallen zonder quotes.

---

## 5. PowerShell-spiekbrief

### Basis
| Wat | Code |
|---|---|
| Variabele | `$naam = 'waarde'` / `$getal = 5` |
| Scherm leeg | `clear` of `Clear-Host` |
| Tonen | `Write-Host "tekst $variabele"` |
| Tonen in kleur | `Write-Host -ForegroundColor Red "WARNING"` |
| Zonder nieuwe regel | `Write-Host -NoNewline "..."` |
| Vragen | `$antw = Read-Host -Prompt "Vraag"` |
| Wachten | `Start-Sleep -Seconds 1` / `-Milliseconds 500` |
| Stoppen | `Exit` |
| Huidige map | `$pwd` |
| Hostname | `hostname` of `$env:COMPUTERNAME` |
| Datum/tijd | `Get-Date`, `Get-Date -Format "dd-MM-yyyy HH:mm"` |
| Tijd optellen | `(Get-Date).AddSeconds(10)` |
| Lengte tekst | `$tekst.Length` |
| Type check | `$x -Is [int]`, `$x -Is [string]` |
| Commentaar | `# één regel`, `<# blok #>` |

### Speciale tekens (backtick `` ` ``)
| Teken | Betekenis |
|---|---|
| `` `n `` | nieuwe regel |
| `` `r `` | terug naar begin van de regel (overschrijven) |
| `` `t `` | tab |
| `${var}:` | variabele direct gevolgd door `:` |
| `$($obj.Eigenschap)` | eigenschap in een tekst zetten |

### Vergelijken en combineren
| Operator | Betekenis |
|---|---|
| `-eq` / `-ne` | gelijk / niet gelijk |
| `-lt` / `-le` | kleiner / kleiner of gelijk |
| `-gt` / `-ge` | groter / groter of gelijk |
| `-And` / `-Or` / `-Not` | en / of / niet |
| `-in` / `-notin` | zit (niet) in lijst: `$a -in "1","2","9"` |
| `-like` | met wildcard: `$naam -like "SRV*"` |

### Bestanden
| Wat | Code |
|---|---|
| Bestaat bestand? | `Test-Path -LiteralPath $ResultFile` |
| Nieuw / overschrijven | `Set-Content -Path $f -Value "tekst"` |
| Regel toevoegen | `Add-Content -Path $f -Value "tekst"` |
| Lezen | `Get-Content $f` |
| Openen | `Invoke-Item $f` (of `notepad $f`) |
| Verwijderen | `Remove-Item $f` |

### Structuren
```powershell
If ($x -eq 1) { ... } ElseIf ($x -eq 2) { ... } Else { ... }

While (voorwaarde) { ... }                 # eerst controleren
Do { ... } Until (voorwaarde)              # eerst doen
For ($i = 1; $i -le 10; $i++) { ... }      # tellen
ForEach ($item in $lijst) { ... }          # elk item
Switch ($keuze) { '1' {...} '2' {...} }    # meerdere keuzes
```

### Veelvoorkomende fouten
| Fout | Oorzaak / oplossing |
|---|---|
| `running scripts is disabled` | `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` |
| `The term '.\script.ps1' is not recognized` | Je zit in de verkeerde map → `cd` naar de map, of tik `.\` + Tab |
| Lus stopt nooit | `-Or` i.p.v. `-And`, of teller wordt niet verhoogd |
| `$args[0]` is leeg | Je gebruikt `param()` bovenin, of je gaf geen argument mee |
| `=` in een If | Vergelijken is `-eq`, niet `=` |

---

## 6. Testen en documenteren

### Opbouw testdocument
1. **Titel** + korte **inleiding**: welk script, welke versie, testomgeving (bijv. *SRV01, Windows Server 2022, PowerShell 5.1*).
2. **Testtabel** (hieronder).
3. Per testgeval een **screenshot met bijschrift** en het commando dat je typte.
4. **Conclusie**: alle tests geslaagd / wat je hebt verbeterd na een fout.

### Testtabel (kopieer en vul in)
| # | Testgeval | Invoer / commando | Verwacht resultaat | Werkelijk resultaat | OK? |
|---|---|---|---|---|---|
| 1 | Zonder argumenten, bestand bestaat niet | `.\MonitorFreeMemory.ps1` | 5 sec meten, Results.txt aangemaakt | | |
| 2 | Met argumenten | `.\MonitorFreeMemory.ps1 10 test2.txt` | 10 sec meten, test2.txt aangemaakt | | |
| 3 | Bestand bestaat, foute invoer | `x` | vraag wordt opnieuw gesteld | | |
| 4 | Bestand bestaat, optie 1 | `1` | metingen toegevoegd onder bestaande | | |
| 5 | Bestand bestaat, optie 2 | `2` | bestand opnieuw begonnen | | |
| 6 | Bestand bestaat, optie 9 | `9` | script stopt direct | | |
| 7 | Iets bewegends | – | aftelling 3-2-1 zichtbaar | | |
| 8 | Openen, foute invoer | `q` | vraag (y/n) wordt herhaald | | |
| 9 | Openen: y | `y` | bestand opent in Kladblok | | |
| 10 | Openen: n | `n` | nette melding, script klaar met tijd | | |
| 11 | Inhoud bestand | Kladblok | DELETE-regel, INSERT-regel, SQL-regels met hostname/waarde/datum | | |

> Fout gevonden tijdens het testen? **Schrijf het op** (wat ging mis, wat heb je aangepast, hertest). Dat levert punten op bij "documenteren".

---

## 7. Een onbekend script lezen (zo raak je niet in paniek)

1. **Lees de header** (`<# ... #>` bovenaan): wat doet het script, welke argumenten verwacht het?
2. **Zoek alle `OPDRACHT:`** met `Ctrl + F`. Schrijf ze op een kladje: 1 regel per opdracht.
3. **Kijk naar de variabelen** die al gebruikt worden (alles met `$`). Gebruik **dezelfde namen** in jouw code! Staat er verderop `$ResultFile`, noem het dan niet `$bestand`.
4. **Kijk wat er boven en onder de OPDRACHT staat.** Vaak staat er een voorbeeld vlak in de buurt (bijv. de y/n-lus onderaan is een voorbeeld voor de 1/2/9-lus bovenaan).
5. **Draai het script al vóór je iets aanpast.** Dan zie je wat wel en niet werkt.
6. **Debuggen:** zet tijdelijk `Write-Host "seconds = $seconds"` neer om te zien wat een variabele bevat.
7. **Hulp bij een commando:** `Get-Help Test-Path -Examples` of zoek op "PowerShell Test-Path".

---

## 8. Het `param()`-blok: zo bouw je het op

`param()` is een andere manier om argumenten binnen te halen. Het moet **helemaal bovenaan** het script staan (alleen commentaar mag erboven).

```powershell
param(
  [int]$seconds = 5,                  # type, naam, standaardwaarde
  [string]$ResultFile = 'Results.txt' # komma tussen elke parameter, NIET na de laatste
)
```

| Onderdeel | Betekenis |
|---|---|
| `[int]` / `[string]` / `[bool]` | het type: getal / tekst / waar-onwaar |
| `$seconds` | naam van de variabele |
| `= 5` | standaardwaarde als de gebruiker niks opgeeft |
| `,` | tussen de parameters |

**Aanroepen:**
```powershell
.\script.ps1                               # alles standaard: 5 en Results.txt
.\script.ps1 10 test.txt                   # op volgorde (positie)
.\script.ps1 -seconds 10 -ResultFile a.txt # met naam
.\script.ps1 -ResultFile a.txt             # alleen de tweede
```

**Verplichte parameter** (script vraagt erom als je hem vergeet):
```powershell
param(
  [Parameter(Mandatory=$true)]
  [string]$ComputerName,
  [int]$Count = 4
)
```

**Alleen bepaalde waarden toestaan:**
```powershell
param(
  [ValidateRange(1,60)][int]$seconds = 5,
  [ValidateSet('C:','D:')][string]$Drive = 'C:'
)
```

### `param()` of `$args`? Kies er één!
| | `$args` | `param()` |
|---|---|---|
| Waar | overal in het script | **moet bovenaan** |
| Standaardwaarde | aparte regels: `$seconds = 5` + `If ($args[0] ...)` | in het blok zelf |
| Met naam aanroepen | nee | ja (`-seconds 10`) |
| Samen gebruiken | **nee**: met `param()` is `$args` leeg | |
| Wanneer | als de opdracht `$args` noemt of de docent-uitwerking het gebruikt | als de opdracht "parameters" noemt of er al een `param()` staat |

> In de oefening noemde de opdracht "eerste/tweede argument" en gebruikte de docent `$args`. **Twijfel? Gebruik `$args[0]`/`$args[1]`.**

---

## 9. Bouwplan: elk script heeft deze volgorde

Krijg je een leeg of half script? Bouw het in deze volgorde op. Elk blok uit je tekening wordt een paar regels code.

```powershell
<# 1. HEADER
  Scriptnaam:  Naam.ps1
  Functie   :  Wat doet het script
  Auteur    :  Jouw naam
  Versie    :  1.0
  Datum     :  vandaag
#>

# 2. (optioneel) param-blok, MOET bovenaan
# param([int]$seconds = 5)

clear

# 3. DEFAULTS
$seconds    = 5
$ResultFile = 'Results.txt'
$MachineName = hostname

# 4. ARGUMENTEN
If ($args[0] -Is [int])    { $seconds = $args[0] }
If ($args[1] -Is [string]) { $ResultFile = $args[1] }

# 5. CONTROLES (bestaat bestand? menu? invoerlus?)
If (Test-Path $ResultFile) {
  # menu + invoerlus + keuze
}
Else {
  # nieuw bestand
}

# 6. VOORBEREIDEN LUS
$stopAt = (Get-Date).AddSeconds($seconds)
Write-Host "Script start..."

# 7. DE LUS (meten, tonen, wegschrijven, iets bewegends)
While ((Get-Date) -Lt $stopAt) {
  # ...
}

# 8. AFRONDEN (melding, vraag openen, eindtijd)
Write-Host "Klaar om $(Get-Date)"
```

---

## 10. Situaties: "de opdracht zegt..." → code

### Bestanden en mappen
**"Controleer of de map C:\Logs bestaat, zo niet maak hem aan"**
```powershell
$LogFolder = 'C:\Logs'
If (-Not (Test-Path $LogFolder)) {
  New-Item -ItemType Directory -Path $LogFolder | Out-Null
  Write-Host "Map $LogFolder aangemaakt."
}
```

**"Schrijf elke regel met datum en tijd in een logbestand"**
```powershell
$LogFile = 'C:\Logs\script.log'
Add-Content -Path $LogFile -Value "$(Get-Date -Format 'dd-MM-yyyy HH:mm:ss') - Script gestart"
```

**"Lees een lijst servers/processen uit een tekstbestand"**
```powershell
$Lijst = Get-Content -Path 'servers.txt'
ForEach ($Server in $Lijst) {
  Write-Host "Bezig met $Server"
}
```

**"Tel het aantal regels in het resultatenbestand"**
```powershell
$Aantal = (Get-Content $ResultFile).Count
Write-Host "Het bestand bevat $Aantal regels."
```

**"Vervang de laatste komma door een puntkomma"** (handig voor de SQL!)
```powershell
$Inhoud = Get-Content $ResultFile
$Inhoud[-1] = $Inhoud[-1].TrimEnd(',') + ';'
Set-Content -Path $ResultFile -Value $Inhoud
```

**"Exporteer naar CSV"**
```powershell
Get-Process | Select-Object Name, CPU, WorkingSet | Export-Csv -Path 'processen.csv' -NoTypeInformation
```

### Invoer van de gebruiker
**"Vraag een getal tussen 1 en 60, net zolang tot het klopt"**
```powershell
Do {
  $invoer = Read-Host "Geef het aantal seconden (1-60)"
  $getal  = $invoer -as [int]               # wordt $null als het geen getal is
} Until (($getal -ge 1) -And ($getal -le 60))
```

**"Vraag de naam van een server, niet leeg"**
```powershell
Do {
  $Server = Read-Host "Naam van de server"
} Until ($Server -ne "")
```

**"Vraag bevestiging (j/n) en stop bij n"**
```powershell
Do { $antw = Read-Host "Doorgaan? (j/n)" } Until ($antw -in 'j','n')
If ($antw -eq 'n') { Write-Host "Script gestopt."; Exit }
```

**"Toon een menu dat terugkomt tot de gebruiker 9 kiest"**
```powershell
Do {
  Write-Host "`n[1] Vrije schijfruimte"
  Write-Host "[2] Draaiende processen"
  Write-Host "[9] Afsluiten"
  $keuze = Read-Host "Maak een keuze"
  Switch ($keuze) {
    '1' { Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, FreeSpace }
    '2' { Get-Process | Select-Object -First 10 Name, CPU }
    '9' { Write-Host "Tot ziens!" }
    Default { Write-Host -ForegroundColor Red "Ongeldige keuze" }
  }
} Until ($keuze -eq '9')
```

### Meten en controleren
**"Controleer of een proces draait"**
```powershell
$Proces = Get-Process -Name 'notepad' -ErrorAction SilentlyContinue
If ($Proces) { Write-Host "Notepad draait." }
Else         { Write-Host -ForegroundColor Red "Notepad draait niet." }
```

**"Controleer of een service draait, zo niet start hem"**
```powershell
$Service = Get-Service -Name 'Spooler'
If ($Service.Status -ne 'Running') {
  Start-Service -Name 'Spooler'
  Write-Host "Spooler is gestart."
}
```

**"Ping een adres en toon of het gelukt is"**
```powershell
If (Test-Connection -ComputerName '8.8.8.8' -Count 1 -Quiet) {
  Write-Host -ForegroundColor Green "Ping gelukt"
} Else {
  Write-Host -ForegroundColor Red "Ping mislukt"
}
```

**"Waarschuw als het vrije geheugen onder 1 GB komt"**
```powershell
$FreeKB = (Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory   # in KB
If ($FreeKB -lt 1048576) {   # 1 GB = 1024 * 1024 KB
  Write-Host -ForegroundColor Red "WARNING: minder dan 1 GB vrij geheugen!"
}
```

**"Toon vrije ruimte in GB met 2 decimalen"**
```powershell
$Free = (Get-CimInstance Win32_LogicalDisk -Filter "DeviceID='C:'").FreeSpace
$FreeGB = [math]::Round($Free / 1GB, 2)
Write-Host "Vrije ruimte op C: $FreeGB GB"
```

**"Toon de 5 processen die het meeste geheugen gebruiken"**
```powershell
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 5 Name, WorkingSet
```

**"Toon alleen processen met meer dan 100 MB geheugen"**
```powershell
Get-Process | Where-Object { $_.WorkingSet -gt 100MB } | Select-Object Name, WorkingSet
```

### Lussen
**"Herhaal 10 keer"**
```powershell
For ($i = 1; $i -le 10; $i++) {
  Write-Host "Meting $i"
}
```

**"Meet elke 2 seconden, totdat de tijd om is"**
```powershell
$stopAt = (Get-Date).AddSeconds($seconds)
While ((Get-Date) -Lt $stopAt) {
  # meting
  Start-Sleep -Seconds 2
}
```

**"Doe iets voor elke regel / elk item"**
```powershell
ForEach ($item in $lijst) {
  Write-Host $item
}
```

### Overig
**"Maak een functie die een melding met tijd toont"**
```powershell
function Write-Log {
  param([string]$Tekst)
  Write-Host "$(Get-Date -Format 'HH:mm:ss') $Tekst"
}
Write-Log "Script gestart"     # aanroepen
```
Een functie moet **boven** de plek staan waar je hem gebruikt.

**"Vang een fout netjes af"**
```powershell
Try {
  Get-Content 'bestaatniet.txt' -ErrorAction Stop
}
Catch {
  Write-Host -ForegroundColor Red "Fout: $($_.Exception.Message)"
}
```

**"Schrijf een meting weg als SQL-regel"** (tekst in `' '`, getallen zonder)
```powershell
"('$MachineName',$Waarde,$TimeStamp),"            # ('SRV01',5006236,20261012153725),
"('$MachineName','$ProcesNaam',$Waarde,$TimeStamp),"  # extra tekstkolom: ook tussen ' '
```

**"Kleuren"**: `-ForegroundColor` met `Red`, `Green`, `Yellow`, `Cyan`, `White`; achtergrond met `-BackgroundColor`.

---

## 11. Commentaar schrijven dat punten oplevert

De beoordelaar wil zien dat je het **begrijpt**. Slecht vs. goed:

| Slecht | Goed |
|---|---|
| `# lus` | `# Herhaal de meting zolang de stoptijd ($stopAt) nog niet bereikt is` |
| `# if` | `# Als het eerste argument een getal is, gebruik het als aantal seconden` |
| `# schrijf` | `# Voeg de meting als SQL-regel toe aan het resultatenbestand, zodat het later in MySQL geladen kan worden` |
| `# vraag` | `# Vraag opnieuw zolang de invoer niet 1, 2 of 9 is (invoerlus)` |

Formule: **wat** het doet + **waarom** / **wanneer**, met de variabelenamen.

