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
