# restlos.com – Ausfallseite (GitHub Pages)

Generische, statische Ausfallseite für Störungen der Unternehmensseite restlos.com und der Auktionsplattform auktionen.restlos.com. Der Text nennt kein Datum und keinen konkreten Anlass und kann ohne Anpassung live gehen. Keine Cookies, kein JS, keine externen Requests.

## Inhalt
- `index.html` – die Seite
- `404.html` – Kopie der Startseite, fängt alle anderen Deep-Links auf (z. B. restlos.com/faq). Einziger Unterschied: Die Footer-Links sind wurzelrelativ (`/impressum/`), weil die Seite unter beliebigen Pfaden ausgeliefert wird. Funktioniert deshalb nur mit Custom Domain, nicht unter `<org>.github.io/homepage/`.
- `impressum/index.html`, `datenschutzerklaerung/index.html` – unter den Original-Pfaden, damit die Footer-Links der Auktionsplattform (restlos.com/impressum/, restlos.com/datenschutzerklaerung/) weiter funktionieren
- `CNAME` – Custom Domain `restlos.com`
- `.nojekyll` – Jekyll-Verarbeitung aus

## Deploy (ca. 5 Minuten)
1. Repo existiert bereits: `RESTLOS-COM/homepage` (öffentlich, Branch `main`). Inhalte aktualisieren, committen, pushen.
2. Repo → Settings → Pages → Source: *Deploy from a branch*, Branch `main`, Ordner `/ (root)`.
3. Settings → Pages → Custom domain: `restlos.com` eintragen, speichern. Das Feld muss mit der CNAME-Datei übereinstimmen.
4. Warten, bis GitHub "DNS check successful" meldet, dann **Enforce HTTPS** aktivieren (Zertifikat kommt automatisch, dauert nach DNS-Umstellung einige Minuten bis ca. 1 Stunde).

## DNS (Route 53) – Achtung, kein CNAME am Apex!
`restlos.com` ist die Zone-Apex. Dort ist ein CNAME per DNS-Standard nicht erlaubt (und würde MX/TXT der Zone brechen → E-Mail). Route 53 lehnt das ab.

Stattdessen:

**Apex `restlos.com` – A-Record (TTL 300), bestehende A-Records ersetzen:**
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
**Apex `restlos.com` – AAAA-Record (TTL 300):**
```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```
**`www.restlos.com` – CNAME (TTL 300):** `<github-org-oder-user>.github.io.`

(Route-53-Alias auf GitHub geht nicht, GitHub ist kein AWS-Ziel. Die A/AAAA-Adressen vor dem Setzen noch einmal gegen https://docs.github.com/pages → "custom domain" prüfen.)

MX, TXT (SPF/DKIM/DMARC), `auktionen`-Einträge **nicht anfassen**.

## Ausfall der Auktionsplattform (auktionen.restlos.com) – noch nicht eingerichtet
Eine GitHub-Pages-Site bedient nur eine Custom Domain (plus `www`). Dieses Repo hängt an `restlos.com`, für `auktionen.restlos.com` braucht es einen eigenen Weg, z. B.:
- zweites Pages-Repo mit denselben Dateien und `CNAME` = `auktionen.restlos.com`; im Störfall in Route 53 `auktionen` per CNAME auf `<github-org>.github.io.` umstellen (Subdomain, also CNAME erlaubt), oder
- dieselben Dateien als Fehlerseite in CloudFront/S3 hinterlegen, sodass AWS sie bei 5xx selbst ausliefert.

## Vor dem Umschalten notieren
- Aktuelle A-Records und TTL von `restlos.com` (für den Rückweg)
- TTL des alten Eintrags bestimmt, wie lange Clients noch die alte IP sehen

## Rückweg
A/AAAA am Apex wieder auf die Hoster-Werte setzen, `www`-CNAME zurück, in GitHub die Custom Domain entfernen.

Empfehlung: `restlos.com` in den Org-Einstellungen unter *Settings → Pages → Verified domains* verifizieren (GitHub nennt den nötigen TXT-Record). Dann kann kein fremdes Repo die Domain als Custom Domain beanspruchen, auch nicht, solange sie hier noch eingetragen ist.

## Pflege
- Texte nur in `index.html` ändern und `404.html` danach daraus erzeugen (macht die Footer-Links wurzelrelativ):
  ```
  sed 's|href="impressum/"|href="/impressum/"|; s|href="datenschutzerklaerung/"|href="/datenschutzerklaerung/"|' index.html > 404.html
  ```
- Das RESTLOS-Logo (Quelle: restlos.com/wp-content/uploads/2025/01/restlos_logo.svg) ist als Inline-SVG im Header jeder Seite eingebettet, damit es ohne zusätzliche Requests und unter jedem Pfad funktioniert.
