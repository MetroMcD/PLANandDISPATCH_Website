# dispatcher-ai.de

Statische Website für PLAN and DISPATCH, gehostet auf GitHub Pages (Domain via `CNAME`).

## Struktur

Jede Seite gibt es auf Deutsch und Englisch. Die deutschen liegen im Wurzelverzeichnis, die englischen unter `en/`:

| Deutsch | Englisch | |
|---|---|---|
| `index.html` | `en/index.html` | Startseite |
| `ueber-mich.html` | `en/about.html` | Über mich |
| `impressum.html` | `en/imprint.html` | Impressum (noindex) |
| `datenschutz.html` | `en/privacy.html` | Datenschutz (noindex) |

```
assets/             Bilder (WebP) + og-image.jpg für Social-Previews
robots.txt          erlaubt alles, verweist auf die Sitemap
sitemap.xml         die indexierbaren Seiten
.nojekyll           schaltet Jekyll-Verarbeitung auf GitHub Pages ab
```

Impressum und Datenschutz stehen bewusst auf `noindex` und nicht in der Sitemap — Rechtstexte sollen nicht ranken. Die englischen Fassungen tragen den Hinweis, dass nur die deutsche Version rechtlich verbindlich ist.

## Pflege

Die HTML-Dateien sind die Quelle — sie werden **direkt bearbeitet**. Änderung committen und pushen, GitHub Pages deployt automatisch.

Vorher war die Seite ein Claude-Artifact-Bundle: der Inhalt lag Base64- und gzip-kodiert in einem 4,3-MB-`index.html` und wurde erst per JavaScript ausgepackt. Google konnte davon nichts lesen — die Seite war nicht indexierbar. Deshalb gilt:

> **Einen Claude-Export nie wieder direkt über `index.html` schreiben.** Damit wäre die Seite sofort wieder unsichtbar für Suchmaschinen.

### Worauf beim Bearbeiten zu achten ist

- **DE und EN sind getrennte Dateien.** Jede inhaltliche Änderung muss in beiden Fassungen gepflegt werden — z. B. in `index.html` *und* `en/index.html`.
- **`hreflang` muss wechselseitig bleiben.** Jede Seite verweist im `<head>` auf sich selbst *und* auf ihre Übersetzung. Zeigt eine Seite auf eine Übersetzung, die nicht zurückzeigt, ignoriert Google die Angabe komplett. Beim Umbenennen einer Datei also immer beide Seiten anfassen.
- **Neue Seite?** `canonical` setzen, das `hreflang`-Paar auf beiden Seiten ergänzen und einen `<url>`-Eintrag in `sitemap.xml` hinzufügen.
- **Neues Bild?** Als WebP nach `assets/` legen und am `<img>` immer `width`, `height` und `loading="lazy"` setzen (`fetchpriority="high"` statt lazy, wenn das Bild above the fold steht). Ohne `width`/`height` springt das Layout beim Laden.
- **Hover-Effekte** laufen über die CSS-Klassen `hv-link`, `hv-cta`, `hv-pill`, `hv-box`, `hv-soft` im `<style>`-Block.
- **Menü und Sprachumschalter**: Das Burger-Menü hängt an dem kleinen Skript am Seitenende. Der Sprachumschalter ist ein normaler Link zwischen `/` und `/en/` — das muss ein echter `<a href>` bleiben, sonst findet Google die englische Version nicht.

### Prüfen, ob der Inhalt für Google sichtbar ist

Der entscheidende Test — misst den Text, den ein Crawler ohne JavaScript sieht:

```bash
curl -s https://dispatcher-ai.de/ | python3 -c "
import sys,re,html
t=re.sub(r'(?s)<(script|style).*?</\1>','',sys.stdin.read())
t=html.unescape(re.sub(r'(?s)<[^>]+>',' ',t))
print(len(re.sub(r'\s+',' ',t).strip()))"
```

Erwartung: über 5.000 Zeichen. Sinkt der Wert auf ein paar hundert, ist der Inhalt wieder hinter JavaScript verschwunden.

Lokal testen:

```bash
python3 -m http.server 8899   # dann http://localhost:8899/
```

## Nach Änderungen an Seitenstruktur oder URLs

In der [Google Search Console](https://search.google.com/search-console) die Sitemap erneut einreichen und für geänderte Seiten „Indexierung beantragen" auslösen.
