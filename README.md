# Billett → Wallet

Statische Web-App, die den Barcode eines Tickets (PDF/PNG/JPG) ausliest und daraus eine
signierte `.pkpass`-Datei für Apple Wallet baut. Alles passiert im Browser – Ticket,
Zertifikat und privater Schlüssel verlassen das Gerät nicht.

## Hosten

1. Neues Repository anlegen, `index.html` ins Root legen, pushen.
2. Settings → Pages → Source: `Deploy from a branch`, Branch `main`, Ordner `/ (root)`.
3. Nach ein bis zwei Minuten läuft die Seite unter `https://<user>.github.io/<repo>/`.

HTTPS ist Pflicht, weil die App `crypto.subtle` für die SHA-1-Hashes des Manifests nutzt.
GitHub Pages liefert das mit. Lokal testen entsprechend über `localhost`, nicht per `file://`.

## Voraussetzung: Zertifikat

Apple Wallet installiert nur signierte Pässe. Nötig sind:

- **Pass Type ID Zertifikat als `.p12`** – Apple Developer Program (99 $/Jahr),
  unter *Identifiers → Pass Type IDs* eine ID anlegen, Zertifikat erzeugen,
  in der Schlüsselbundverwaltung samt privatem Schlüssel als `.p12` exportieren.
- **Apple WWDR Zwischenzertifikat** (`.cer`) aus dem Bereich *Certificates* bei Apple.

`passTypeIdentifier` und `teamIdentifier` liest die App automatisch aus dem Zertifikat.

## Was die App erzeugt

```
pass.json        Passdaten inkl. barcodes[]
icon.png         29/58/87 px, aus Anbietername und Farben generiert
logo.png         160/320 px, optional aus hochgeladenem Logo
manifest.json    SHA-1 je Datei
signature        PKCS#7 detached, SHA-256, Kette Pass-Zertifikat + WWDR
```

Alles zusammen als ZIP mit MIME-Typ `application/vnd.apple.pkpass`.

## Grenzen

- Barcodes werden **nicht umkodiert**. Ein Aztec-Code bleibt Aztec, sonst scheitert die Kontrolle.
- Binäre Nutzlasten (häufig bei Bahntickets) überstehen den Umweg über Text nicht immer.
  Wenn die App auf `utf-8` umschaltet, ist Vorsicht angebracht.
- Ob ein selbst gebauter Pass am Kontrollgerät akzeptiert wird, entscheidet der Anbieter.
- Kein Push-Update der Pässe: dafür bräuchte es `webServiceURL`, `authenticationToken`
  und einen echten Server.

## Abhängigkeiten

Per CDN geladen: JSZip, node-forge, pdf.js, qrcode, `zxing-wasm` (Barcode-Erkennung
als ES-Modul, WebAssembly-Kern von ZXing-C++). `zxing-wasm` ersetzt die vorher
genutzte reine JS-Portierung `@zxing/library`, die bestimmte echte Aztec-Codes
(z. B. Eurowings-Bordkarten) auch auf sauberen Bildern nicht zuverlässig las.
Für Offline-Betrieb die Dateien ins Repo legen und die `<script src>`/`import`
anpassen.
