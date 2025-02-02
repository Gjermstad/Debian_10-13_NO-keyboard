# Debian 10.13 ARM64 for mac med M-chip (med norske taster på tastaturet)

---

Denne guiden forutsetter at du bruker en Debian‑basert VM med XFCE (eller lignende) der du har tilgang til en terminal og kan redigere filer i din hjemmekatalog. Guiden lager et oppstartsskript som laster opp en modifisert XKB-konfigurasjon slik at tasten til venstre for **`1`** (nøkkel-ID `<TLDE>`) får følgende oppførsel:

- **Nivå 1 (vanlig trykk):** `'` (apostrof)
- **Nivå 2 (med Shift):** `§` (seksjonstegn)
- **Nivå 3 (med Option):** `~` (tilde)
- **Nivå 4 (med Shift+Option):** `~` (tilde)

Samt at venstre `Option`-tast fungerer riktig.

> *Merk: I eksempelet under bruker vi standard norsk tastaturoppsett med varianten mac_nodeadkeys. Du kan justere oppsettet etter dine behov.*

> Denne oppskriften er satt opp for VM’en av Debian 10.13 vi fikk utlevert i faget **PG3401 Programmering i C for Linux** på Høyskolen Kristiania våren 2025 fra foreleseren vår Bengt Østby for å kunne kjøre Debian 10 på en mac med Apple Silicon chip (M-prosessor).

> *Jeg har brukt rundt 9-10 timer denne helgen sammen med **ChatGPT o3-mini** på å endelig få til et system som fungerer. For VM’en vi fikk fra Bengt er lagt opp så venstre Option-knapp ikke fungerer og venstre`Option+8/9` gir ikke `[]`, venstre `Shift+Option+8/9` gir ikke `{}`, knappen over `@`-knappen gir ikke `~` når man holder inne venstre Option og knappen til venstre for `1` gav meg `|` istedenfor `'` (og apostrof er viktig i C siden `'a'` og `"a"` er to forskjellige ting). Jeg har testet dette på en Macbook Pro M1 Max og en Macbook Air M3. *- Kenneth 02.02.2025* 

** Obs! Siden dette en en oppskrift for å fikse at knappene ikke gir riktig tegn, så vær klar over at for å kunne skrive inn terminalkommandoene i hvert steg må du enten:**

- Åpne nettleseren, søk på “ASCII table w3school” og kopiere og lime inn tegnene når du trenger dem:

---

## Steg 0: Sett opp menyen til Keyboard, for å være sikker på at du starter på samme sted som jeg gjorde (usikker på om viktig)

Gå til Application > Settings > Keyboard > Layout inni VM'en.

Sett "Keyboard model: Macbook/Macbook Pro (Intl.)" og "Keyboard layout: Norwegian (Macintosh, no dead keys)".

---

## Steg 1: Sett opp tastaturet i Linux

Åpne terminalen og kjør: 

```bash
sudo nano /etc/default/keyboard
```

i filen som åpner seg endre (om nødvendig) slik at filen viser: 

```
XKBMODEL="pc105"
XKBLAYOUT="no"
XKBVARIANT="mac_nodeadkeys"
XKBOPTIONS="lv3:lalt_switch"

BACKSPACE="guess"
```

Deretter kjører du: 

```bash
sudo dpkg-reconfigure keyboard-configuration
```

Nå kommer det opp en blå skjerm og du velger følgende valg (bruk pil opp/ned og `Enter`):

1. Generic 105 key PC (Intl.)
2. Det er ikke sikkert denne menyen med bare “English (US)” kommer opp, men om den gjør det så går du nederst, velger “Other”, så kommer neste valg opp og du kan fortsette og velge “Norwegian”.
3. Norwegian
4. Norwegian - Norwegian (Macintosh, no dead keys)
5. Left Alt
6. No Compose Key
7. No

Så slår du av og starter opp igjen VM’en ved å bruke ***prog > Shut Down*** (øverst til høyre på skjermen i VM'en).

---

## Steg 2: Lag din personlige XKB-katalog (hvis den ikke allerede finnes)

Åpne en terminal og kjør:

```bash
mkdir -p ~/.xkb/symbols
```

Dette oppretter katalogen `~/.xkb/symbols` der du kan legge dine egne XKB-filer.

---

## Steg 3: Lag en fil med din tilpassede variant

Vi lager en fil for den norske layouten med din overstyring. Her omdøper vi layouten til `no` (dette overstyrer den eksisterende `no`-filen). Hvis du ønsker å beholde den originale, kan du bruke et annet navn, for eksempel `no_custom`, men da må du bruke det navnet i setxkbmap‑kommandoen.

I dette eksempelet endrer vi filen `~/.xkb/symbols/no`:

1. Åpne filen med en editor (her bruker vi `nano`):
    
    ```bash
    nano ~/.xkb/symbols/no
    ```
    
2. Lim inn følgende innhold (sørg for at du fjerner eventuell tidligere innhold hvis du vil overstyre hele filen):
    
    ```
    partial alphanumeric_keys
    xkb_symbols "mac_nodeadkeys_custom" {
        include "pc+macintosh_vndr/no+inet(evdev)+level3(alt_switch)"
        key <TLDE> { [ apostrophe, section, asciitilde, asciitilde ] };
    };
    ```
    
    **Forklaring:**
    
    - Vi bruker `partial alphanumeric_keys` (merk understreken) for å angi at vi definerer noen alfanumeriske taster.
    - Vi definerer en variant med navnet **mac_nodeadkeys_custom**.
    - Vi inkluderer den standard definisjonen for norsk Mac (bruker inkluderingsstrengen `"pc+macintosh_vndr/no+inet(evdev)+level3(alt_switch)"` – denne skal matche det som finnes i systemets oppsett).
    - Vi overstyrer nøkkelen `<TLDE>` slik at den gir:
        - Nivå 1: `apostrophe`
        - Nivå 2: `section`
        - Nivå 3 og 4: `asciitilde`
3. Lagre filen:
    - I nano: Trykk **`Ctrl+O`**, deretter **`Enter`** for å lagre filen, og så **`Ctrl+X`** for å avslutte.

---

## Steg 4: Opprett oppstartsskriptet for å laste inn den modifiserte XKB-konfigurasjonen

Vi lager et skript som kjører ved oppstart av den grafiske sesjonen. Den enkleste måten er å bruke filen **~/.xsessionrc**.

1. Åpne (eller opprett) filen:
    
    ```bash
    nano ~/.xsessionrc
    ```
    
2. Lim inn følgende innhold:
    
    ```bash
    #!/bin/bash
    # Sett standard tastaturoppsett for norsk Mac med mac_nodeadkeys-variant og Option som Level3-switch
    setxkbmap -layout no -variant mac_nodeadkeys -option lv3:alt_switch
    
    # Dump den nåværende tastaturoppsettet til en fil med full XKB-konfigurasjon
    xkbcomp -xkb $DISPLAY ~/current.xkb
    
    # Erstatt hele blokken for <TLDE> med ønsket mapping:
    # Området fra linjen som begynner med "key <TLDE>" til linjen med "};"
    sed -i '/^ *key <TLDE>/, /};/c\        key <TLDE> { type= "FOUR_LEVEL", symbols[Group1]= [ apostrophe, section, asciitilde, asciitilde ] };' ~/current.xkb
    
    # Last inn den modifiserte tastaturoppsettet
    xkbcomp -I$HOME/.xkb ~/current.xkb $DISPLAY
    ```
    
    **Merk:**
    
    - Sed-kommandoen ovenfor bruker et område (fra linjen som begynner med `key <TLDE>` til den linjen som inneholder `};`) og erstatter hele blokken med den nye definisjonen.
    - Fjern eventuelle ekstra linjer som setter andre `setxkbmap -option`kommandoer senere i skriptet, slik at overstyringen du nettopp la inn ikke blir overskrevet.
3. Lagre filen og avslutt:
    - I nano: **`Ctrl+O`** (lagre), **`Enter`**, deretter **`Ctrl+X`**.
4. Gjør skriptet kjørbart:
    
    ```bash
    chmod +x ~/.xsessionrc
    ```
    

---

## Steg 5: Test oppsettet

1. Start VM-en på nytt (***prog > Shut down*** øverst til høyre).
2. Når du er logget inn, åpne et terminalvindu og prøv:
    
    Trykk på tasten til venstre for **`1`** (nøkkel-ID `<TLDE>`). Forvent følgende:
    
    - Ved vanlig trykk: Du skal se at tastetrykket registrerer **aspostrof** (`'`).
    - Med venstre**`Shift`**: Du skal få **paragraf** (`§`).
    - Med venstre **`Shift+Option`**: Du skal få **tilde** (`~`).
    
    Når du holder inne venstre `Option` og trykker på 8 skal du få `[` og på 9 `]`.
    
    Når du holder inne venstre `Shift+Option` og trykker på 8 skal du få `{` og på 9 `}`.
    
3. Du kan også åpne en teksteditor og teste inndata for å bekrefte at oppsettet fungerer slik du ønsker.

---

## Ferdig 🎉

Om alt nå fungerer skal du ha følgende:

- *venstre`Option+8/9` gir `[]`*
- *venstre `Shift+Option+8/9` gir `{}`*
- *knappen til venstre for `1` (med `'`-tegnet) gir  `'` og venstre `Option+'` gir `~`*
- Ellers skal knappene fungere som normalt 🤞

---

## Ekstra tips

- Om du noen gang trenger å se den modifiserte XKB-konfigurasjonen, kan du åpne filen `~/current.xkb` med en teksteditor:
    
    ```bash
    nano ~/current.xkb
    ```
    
- Hvis du ønsker å gjøre endringer i fremtiden, rediger først filen `~/.xkb/symbols/no` med dine tilpasninger og deretter start oppstartsskriptet på nytt (eller logg ut og inn).
- Om du vil se hvilken input en tast gir, åpne et terminalvindu og prøv:
    
    ```bash
    xev
    ```
    

---

## Oppsummering

1. Åpne terminalen og kjør: 
    
    ```bash
    sudo nano /etc/default/keyboard
    ```
    
    - i filen som åpner seg endre (om nødvendig) slik at filen viser:
        
        ```
        XKBMODEL="pc105"
        XKBLAYOUT="no"
        XKBVARIANT="mac_nodeadkeys"
        XKBOPTIONS="lv3:lalt_switch"
        
        BACKSPACE="guess"
        ```
        
    - Deretter kjører du (se skjermbilder i Steg 1 lenger opp om nødvendig):
        
        ```bash
        sudo dpkg-reconfigure keyboard-configuration
        ```
        
2. **Lag XKB-katalogen og filen:**
    - Kjør:
        
        ```bash
        mkdir -p ~/.xkb/symbols
        nano ~/.xkb/symbols/no
        ```
        
    - Lim inn:
        
        ```
        partial alphanumeric_keys
        xkb_symbols "mac_nodeadkeys_custom" {
            include "pc+macintosh_vndr/no+inet(evdev)+level3(alt_switch)"
            key <TLDE> { [ apostrophe, section, asciitilde, asciitilde ] };
        };
        ```
        
    - Lagre og avslutt.
3. **Lag oppstartsskriptet i `~/.xsessionrc`:**
    - Kjør:
        
        ```bash
        nano ~/.xsessionrc
        ```
        
    - Lim inn skriptet:
        
        ```bash
        #!/bin/bash
        setxkbmap -layout no -variant mac_nodeadkeys -option lv3:alt_switch
        
        xkbcomp -xkb $DISPLAY ~/current.xkb
        
        sed -i '/^ *key <TLDE>/, /};/c\        key <TLDE> { type= "FOUR_LEVEL", symbols[Group1]= [ apostrophe, section, asciitilde, asciitilde ] };' ~/current.xkb
        
        xkbcomp -I$HOME/.xkb ~/current.xkb $DISPLAY
        ```
        
    - Lagre, avslutt, og kjør:
        
        ```bash
        chmod +x ~/.xsessionrc
        ```
        
4. **Start VM'en på nytt og test med `xev`.**

Dette er hele oppsettet fra start til slutt, slik at du kan gjenopprette konfigurasjonen om du må reinnstallere VM-en.
