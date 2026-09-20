# Moonito sensor

A 2.9 KB script that helps [Moonito](https://moonito.net) tell a real visitor
from an automated one.

```html
<script defer
  src="https://cdn.jsdelivr.net/gh/moonito-net/sensor@1.0.0/t.js"
  data-key="YOUR_PUBLIC_KEY"></script>
```

Paste it before `</head>`. That is the whole installation. Your public key is on
the [API page](https://moonito.net/api).

---

## What it does, and what it deliberately does not

It reports what the browser looks like. That is all it does.

It does **not** block anyone, redirect anyone, or decide anything. Every
decision happens on Moonito's servers, where an attacker cannot read the logic.
Your site works exactly the same whether this file loads, fails to load, or is
blocked by an extension.

It is optional. Moonito's server-side protection works without it. This makes
the answers better.

## The problem it helps with

Most bot protection asks one question per request: is this address on a list?

That question cannot see a rotating residential proxy, because the whole point
of one is that every request arrives from a different, genuinely residential
address that is on no list. Nothing about any single request looks wrong.

What does look wrong is the shape over time. One browser appearing through
twelve different networks in ninety seconds. A clock set to Jakarta arriving
from a German home connection. A page that loads and converts with no scrolling
and no pointer movement at all.

Some of that is visible from the server. The rest needs the browser, which is
what this file reports. A rotating proxy changes the network path; it does not
change the timezone on the operator's own machine.

## What it collects

| | |
|---|---|
| Environment | timezone, languages, screen size, CPU cores, memory, platform, whether cookies and storage work |
| Rendering | a hash of how the browser draws, to recognise the same browser again |
| Activity | **counts** of pointer, scroll, key and click events, plus how long the page was open |

## What it never collects

- What anyone types. Keystroke **counts** only, never contents
- Anything from a form field
- Page text, screenshots, or the DOM
- Your visitors' other cookies, or anything from other sites
- Location. It never triggers a permission prompt

Nothing here is shared between Moonito customers as a profile of a person.
Network-level statistics are pooled; individual visitors are not.

## Performance

| | |
|---|---|
| Size | 2.9 KB gzipped |
| Effect on LCP | none. It runs after parse, in an idle callback, never before paint |
| Effect on CLS | none. It never touches the DOM |
| Requests added | one, plus at most one retry |

The build refuses to produce a file over 6 KB gzipped. That limit is why
expensive fingerprinting is left out of the default: it loads on every page
view of every protected site, and a security script that costs you your Core
Web Vitals has taken more than it gave.

## Consent

If you need consent before collecting anything identifying:

```html
<script defer
  src="https://cdn.jsdelivr.net/gh/moonito-net/sensor@1.0.0/t.js"
  data-key="YOUR_PUBLIC_KEY" data-consent="required"></script>

<!-- once the visitor has agreed -->
<script>window.moonito.consent(true);</script>
```

Until then it sends only screen size, timezone and activity counts: no
fingerprint, and it does not read the Moonito cookie. Detection is weaker in
that mode, and Moonito treats it as weaker rather than pretending otherwise.

`navigator.globalPrivacyControl` is honoured automatically. A visitor can opt
out permanently with `moonito.optOut()`.

## Options

| Attribute | Default | |
|---|---|---|
| `data-key` | required | Your public key. Never your secret key |
| `data-consent` | `granted` | `required` holds back identifying data until you allow it |
| `data-collect` | `standard` | `minimal`, `standard`, or `full` |
| `data-spa` | `off` | `on` reports again on route changes |

## API

```js
moonito.set('page_type', 'checkout')  // label a page in your own logs
moonito.consent(true)                 // allow full collection
moonito.optOut()                      // stop, permanently, for this browser
```

Nothing returns a score or a verdict. The browser is never told what the server
concluded, because anything sent to the browser is readable by whoever is
trying to get past it.

## Pinning with Subresource Integrity

```html
<script defer
  src="https://cdn.jsdelivr.net/gh/moonito-net/sensor@1.0.0/t.js"
  integrity="sha384-wLcLPqdSEPFqtF0BDB2bu5CnpuQ4+/NYhvC2y4PavDTo3e7rVEp5eTE+Dgtr1Udu"
  crossorigin="anonymous"
  data-key="YOUR_PUBLIC_KEY"></script>
```

The hash for each release is in `INTEGRITY`.

## Why the version is in the URL

An unversioned jsDelivr path caches for seven days in your visitors' browsers,
and nothing can clear that early. If a bad build ever shipped, it would sit on
their machines for a week.

A pinned version cannot do that to you. The cost is that updates are not
automatic: when a new version ships we will say so, and updating means changing
one number.

## Honestly, what this cannot do

A determined operator running a real browser on a real residential connection,
at human speed, clearing state between sessions, is not detectable by this or by
any product that claims otherwise. The goal is to make automation expensive
enough that it stops being worth doing, and to catch the large majority who
automate rather than sit there.

Everything this file collects runs in an environment the attacker controls, so
it can be faked. Moonito never lets it lower a risk score on its own, and always
checks it against what the server independently observed. Disagreement between
the two is itself a signal. The value is not that this cannot be faked; it is
that faking it convincingly, on every request, costs real work.

---

Built by [Moonito](https://moonito.net). Minified, never obfuscated: there is
no logic in here worth hiding, and you should be able to read what you are
putting on your pages.

[Documentation](https://moonito.net/docs) ·
[Support](https://moonito.net/support) ·
MIT licensed
