---
name: clarity-site
description: Add Microsoft Clarity site monitoring to a Hugo documentation site
---

# Skill: Adding Microsoft Clarity to a Hugo Site

## Overview

This skill adds Microsoft Clarity visitor analytics to a Hugo site built with the **Lotus Docs** theme (or any Hugo theme that uses a separate `docs/head.html` partial for documentation pages).

---

## Step 1 — Prompt for the Clarity Script

Before making any changes, ask the user to provide their Clarity script tag. The script looks like this:

```html
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "<YOUR_CLARITY_ID>");
</script>
```

Use `ask_user` to collect this. Extract the **Clarity project ID** (the alphanumeric string at the end of `https://www.clarity.ms/tag/<ID>`).

---

## Step 2 — Locate the Hugo site root

Identify the Hugo site directory (e.g. `docs/learn`). Look for a `hugo.yaml` or `hugo.toml` at the root. Confirm the `docs.pathName` param — it defaults to `"docs"` if not set.

---

## Step 3 — Inject into the landing page head partial

The landing page uses `layouts/partials/head.html`. If it exists, add the Clarity snippet **before `</head>`**, wrapped in `{{- if hugo.IsProduction }}`:

```html
    <!-- Microsoft Clarity -->
    {{- if hugo.IsProduction }}
    <script type="text/javascript">
        (function(c,l,a,r,i,t,y){
            c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
            t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
            y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
        })(window, document, "clarity", "script", "<CLARITY_ID>");
    </script>
    {{- end }}
</head>
```

If `layouts/partials/head.html` does not exist, create it by copying the theme's equivalent partial from `_vendor/` or the theme module cache.

---

## Step 4 — Inject into the docs head partial (critical — Lotus Docs)

Lotus Docs uses a **separate** partial for all documentation pages: `partials/<pathName>/head.html` (e.g. `partials/docs/head.html`). The docs `baseof.html` resolves it dynamically:

```
{{ partial (printf "%s/%s" ($.Scratch.Get "pathName") "head.html") . }}
```

Because of this, `layouts/partials/head.html` is **not** used on `/docs/*` pages. You must create a local override:

1. Check if `layouts/partials/docs/head.html` already exists in the site's layouts.
2. If **not**, copy the theme's version as a baseline:
   ```
   _vendor/github.com/colinwilson/lotusdocs/layouts/partials/docs/head.html
   ```
3. Add the same Clarity snippet block (from Step 3) immediately before `</head>`.

---

## Step 5 — Verify

The correct guard is `{{- if hugo.IsProduction }}`, **not** `{{- if not hugo.IsServer }}`.

- `hugo.IsServer` is `true` whenever `hugo serve` is used — even with `--environment production` — so it would always suppress the script during local testing.
- `hugo.IsProduction` is `true` when `--environment production` is passed (or the environment is set to `production`), regardless of whether the server flag is used.

To verify locally, run:

```bash
hugo serve -s docs/learn --environment production
```

Then curl the page and confirm the snippet is present:

```bash
curl -s http://localhost:1313 | grep -i clarity
curl -s http://localhost:1313/docs/ | grep -i clarity
```

Both should return a line containing `https://www.clarity.ms/tag/`.

---

## Key Rules

- **Always use `hugo.IsProduction`** — never `not hugo.IsServer` — as the guard condition.
- **Always patch both partials**: `layouts/partials/head.html` (landing page) and `layouts/partials/docs/head.html` (docs pages). Patching only one will leave the other without tracking.
- When creating `layouts/partials/docs/head.html` as an override, copy the full theme file first so no existing functionality is lost.
- Do not add the Clarity script inside `{{- if not hugo.IsServer }}` — this silently drops the script even with `--environment production`.
