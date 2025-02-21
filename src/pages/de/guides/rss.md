---
layout: ~/layouts/MainLayout.astro
title: RSS
description: Eine Einführung in RSS in Astro.
---

Astro unterstützt die schnelle, automatische Generierung von RSS-Feeds für Blogs und andere Content-Websites. Weitere Informationen zu RSS-Feeds im Allgemeinen findest du unter [aboutfeeds.com](https://aboutfeeds.com/).

## Einrichten von `@astrojs/rss`

Das `@astrojs/rss`-Paket bietet Hilfsfunktionen zur Erzeugung von RSS-Feeds mithilfe von [API-Endpunkten](/de/core-concepts/astro-pages/#nicht-html-seiten). Dies ermöglicht _sowohl_ statische Feeds, die zum Erstellungszeitpunkt der Website generiert werden, als auch die On-Demand-Generierung bei Verwendung eines [SSR-Adapters](/de/guides/server-side-rendering/#enabling-ssr-in-your-project).

Installiere zu Beginn `@astrojs/rss` mit deinem bevorzugten Paketmanager:

```bash
# npm
npm i @astrojs/rss
# yarn
yarn add @astrojs/rss
# pnpm
pnpm i @astrojs/rss
```

Stelle dann sicher, dass du in deiner `astro.config` eine [`site` konfigurierst](/de/reference/configuration-reference/#site). Dies ermöglicht die Verwendung der [`SITE`-Umgebungsvariable](/de/guides/environment-variables/#default-environment-variables) zur Erzeugung von Links in deinem RSS-Feed.

:::note[Benötigt v1]
Die `SITE`-Umgebungsvariable existiert nur in der neusten Astro 1.0 Beta. Aktualisiere entweder auf die aktuelle Astro-Version (`astro@latest`) oder fülle den `site`-Parameter manuell, falls dies nicht möglich ist (siehe Beispiele unten).
:::

Lass uns jetzt unseren ersten RSS-Feed generieren! Erstelle die Datei `rss.xml.js` in deinem `src/pages/`-Ordner, um einen über die URL `rss.xml` erreichbaren Feed zu erzeugen. Du kannst den Dateinamen nach deinen Wünschen anpassen, wenn du eine andere Ausgabe-URL bevorzugst.

Importiere nun die `rss`-Hilfsfunktion aus dem `@astrojs/rss`-Paket und rufe sie mit folgenden Parametern auf:

```js
// src/pages/rss.xml.js
import rss from '@astrojs/rss';

export const get = () => rss({
    // `<title>`-Feld in der XML-Ausgabe
    title: 'Buzz’s Blog',
    // `<description>`-Feld in der XML-Ausgabe
    description: 'Ein bescheidener Astronaut und sein Weg zu den Sternen',
    // Basis-URL für RSS-<item>-Links
    // SITE verwendet "site" aus der astro.config deines Projekts.
    site: import.meta.env.SITE,
    // Liste von `<item>`-Elementen in der XML-Ausgabe
    // Einfaches Beispiel: Items für jede md-Datei in /src/pages erzeugen
    // Siehe Abschnitt "Generieren von `items`" für erforderliche Frontmatter und erweiterte Anwendungsfälle
    items: import.meta.glob('./**/*.md'),
    // (optional) Benutzerdefinierten XML-Code einfügen
    customData: `<language>en-us</language>`,
  });
```

## Generieren von `items`

Das `items`-Feld akzeptiert entweder:
1. [Ein `import.meta.glob(...)`-Ergebnis](#1-importmetaglob-ergebnis) **(nutze dies nur für `.md`-Dateien in deinem `src/pages/`-Verzeichnis!)**
2. [Eine Liste der RSS-Feed-Objekte](#2-liste-der-rss-feed-objekte), jeweils mit den Feldern `link`, `title` und `pubDate` sowie optional auch `description` und `customData`.

### 1. `import.meta.glob`-Ergebnis

Wir empfehlen diese Option als eine bequeme Abkürzung für `.md`-Dateien unter `src/pages/`. Jeder Eintrag sollte die Frontmatter-Eigenschaften `title` und `pubDate` sowie optional auch `description` und `customData` haben. Falls dies nicht möglich ist, oder du die Werte lieber selbst im Code generierst, [sieh dir Option 2 an](#2-liste-der-rss-feed-objekte).

Angenommen, deine Blog-Einträge sind im Verzeichnis `src/pages/blog/`. Du kannst dann einen RSS-Feed wie folgt erzeugen:

```js
// src/pages/rss.xml.js
import rss from '@astrojs/rss';

export const get = () => rss({
    title: 'Buzz’s Blog',
    description: 'Ein bescheidener Astronaut und sein Weg zu den Sternen',
    site: import.meta.env.SITE,
    items: import.meta.glob('./blog/**/*.md'),
  });
```

Sieh dir die [Vite-Dokumentation zu Glob Import](https://vitejs.dev/guide/features.html#glob-import) für weitere Informationen über diese Syntax an.

### 2. Liste der RSS-Feed-Objekte

Wir empfehlen diese Option für `.md`-Dateien außerhalb des `pages`-Verzeichnises. Dies ist üblich beim Generieren von Routen [via `getStaticPaths`](/de/reference/api-reference/#getstaticpaths).

Angenommen, deine `.md`-Einträge sind in einem `src/posts/`-Verzeichnis gespeichert. Jeder Eintrag hat einen `title`, `pubDate` und `slug` in seinem Frontmatter, wobei `slug` der Ausgabe-URL auf deiner Website entspricht. Wir können dann wie folgt einen RSS-Feed mithilfe von [Vites Hilfsfunktion `import.meta.glob`](https://vitejs.dev/guide/features.html#glob-import) generieren:

```js
// src/pages/rss.xml.js
import rss from '@astrojs/rss';

const postImportResult = import.meta.glob('../posts/**/*.md', { eager: true });
const posts = Object.values(postImportResult);

export const get = () => rss({
    title: 'Buzz’s Blog',
    description: 'Ein bescheidener Astronaut und sein Weg zu den Sternen',
    site: import.meta.env.SITE,
    items: posts.map((post) => ({
      link: post.url,
      title: post.frontmatter.title,
      pubDate: post.frontmatter.pubDate,
    }))
  });
```

## Ein Stylesheet hinzufügen

Du kannst deinen RSS-Feed ansprechend gestalten, um die Nutzererfahrung bei der Betrachtung im Browser zu verbessern.

Nutze die Option `stylesheet` der `rss`-Funktion, um einen absoluten Pfad zu deinem Stylesheet anzugeben.

```js
rss({
  // Beispiel: Nutze dein Stylesheet aus "public/rss/styles.xsl"
  stylesheet: '/rss/styles.xsl',
  // ...
});
```

Falls du kein RSS-Stylesheet zur Verfügung hast, empfehlen wir das [Pretty Feed v3 Standard-Stylesheet](https://github.com/genmon/aboutfeeds/blob/main/tools/pretty-feed-v3.xsl), das du dir von GitHub herunterladen und im `public/`-Verzeichnis deines Projekts speichern kannst.
