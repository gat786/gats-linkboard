# AGENTS.md

Instructions for AI agents working in this repository.

**There is no test suite, no linter, no formatter, and no type checking** (`jsconfig.json` sets
`checkJs: false`). Nothing will catch a mistake for you. Every change must be verified by running the
app and looking at it. "It builds" is not verification.

## Project shape

| | |
|---|---|
| Framework | SvelteKit `2.70.1` on Svelte `5.56.7` |
| Build | Vite `8.1.5`, `@sveltejs/adapter-static` `3.0.10` |
| Mode | Static SPA — `prerender = true`, `ssr = false` ([src/routes/+layout.js](src/routes/+layout.js)) |
| Language | JavaScript only. No TypeScript. |
| Dev server | `npm run dev` → http://localhost:5173 |

Almost the entire app is one file: [src/routes/+page.svelte](src/routes/+page.svelte) — markup, data
fetching, styles, and the theme palette as `:root[data-theme='…']` blocks. Link data is
[static/links.json](static/links.json).

Things that look tidyable but are not:

- **Theme tokens must be declared at `:root`** (hence `:global(:root)`). Custom properties inherit
  downward only, so tokens set on a nested element leave `background: var(--bg)` on `body` resolving
  to nothing — silently, no error, page renders unpainted.
- **[src/app.html](src/app.html) has a blocking inline script** that applies the theme before first
  paint. Deferring it, or moving it into the component, brings back the flash it prevents. `:root`
  holds the dark palette, so `data-theme='dark'` — and any unrecognised value — needs no block of its
  own, which is why the script does not validate what it reads.
- **There is no `svelte.config.js`.** Kit config is passed inline to the `sveltekit()` plugin in
  [vite.config.js](vite.config.js).
- **Runes mode is forced on** ([vite.config.js](vite.config.js)). Svelte 4 syntax is a compile error:
  use `$state`/`$props`/`$derived`/`$effect` and `onclick={…}`, never `export let`, `$:`, `on:click`.

## Check the current docs before writing code

Do this before editing. Training data skews heavily toward Svelte 4 and SvelteKit 1; recalled syntax
is frequently wrong here. Read the installed versions first, never assumed ones:

```bash
node -e "const p=require('./package-lock.json').packages;['svelte','@sveltejs/kit','vite'].forEach(k=>console.log(k.padEnd(18),p['node_modules/'+k]?.version??'-'))"
```

Then fetch the matching docs. Svelte publishes LLM-oriented plain-text versions — prefer these over
blog posts or memory:

| URL | Use for |
|---|---|
| `https://svelte.dev/llms.txt` | Index of everything below |
| `https://svelte.dev/docs/svelte/llms.txt` | Svelte 5 — runes, snippets, event attributes |
| `https://svelte.dev/docs/kit/llms.txt` | SvelteKit — routing, `+layout.js`, prerendering, adapters |
| `https://svelte.dev/llms-small.txt` | Compressed full docs, when context is tight |
| `https://vite.dev/llms.txt` | Vite 8 config and build behaviour |

If the docs contradict this file, the docs win — and update this file to match.

## End-to-end verification

Run the app and confirm the change yourself. Don't report work as done off a passing build, and don't
ask the user to check for you.

**Read computed styles and DOM state, not just screenshots.** A screenshot under one OS setting hides
bugs that are glaring under another — an unpainted background looks fine on a dark-mode machine and is
unreadable on a light-mode one.

**Two measurement traps, both of which produced false results in this repo:**

- Transitioned properties read stale via `getComputedStyle` when the browser pane is hidden — nothing
  paints, so transitions never advance. Disable transitions before reading values
  (`* { transition: none !important }`), and confirm anything suspicious with an actual screenshot.
  A "bug" where every element is exactly one theme behind is this, not a real defect.
- If a check reports zero, empty, or "none", confirm it actually ran before trusting it. A grep that
  assumes quoted attribute selectors finds nothing after minification; a hex parser that assumes six
  digits misreads `#fff`. Both have caused false alarms here.

### Checklist

- [ ] **All five themes** — Light, Dark, Ocean, Forest, Sunset. Not just the default.
- [ ] **Both OS colour schemes** — `prefers-color-scheme: light` *and* `dark`, for each theme. Every
      theme must look right on a machine set to the opposite scheme.
- [ ] **Contrast** — real ratios for changed colours. AA is 4.5:1 normal text, 3:1 large.
- [ ] **Persistence** — pick a theme, reload, confirm it survives with no flash of the wrong one.
- [ ] **Storage denied** — `localStorage` *throws* in Safari private browsing and when site data is
      blocked. Confirm the theme still applies and links still render.
- [ ] **First visit** — no stored theme should follow the OS preference.
- [ ] **Mobile** — below the 600px breakpoint; the five-button switcher is the tight spot.
- [ ] **Empty state** — temporarily empty `static/links.json`. The spinner must never hang.
- [ ] **Console clean** — zero errors or warnings.
- [ ] **Production build** — `npm run build && npm run preview`. Dev and the prerendered static build
      differ; `ssr = false` means the shipped HTML has no markup, so first paint behaves differently.
- [ ] **Repo clean** — `git status --short`. Revert incidental churn (`package-lock.json` from an
      install, scratch files, temp configs). Write scratch files outside the repo.

## Fallback: no browser available

Use only when you genuinely cannot drive a browser — check for an automation tool, preview pane, MCP
browser server, or headless runner first. This path catches broken *code*; it cannot see broken
*rendering*. Say so explicitly in your report.

Build must succeed:

```bash
npm run build
```

Then check contrast, which is the one accessibility check that loses nothing by being headless — the
maths is the maths. This parses the palette out of the source and checks every theme against AA:

```bash
node -e '
const fs=require("fs");
const src=["src/app.html","src/routes/+page.svelte"].map(f=>fs.readFileSync(f,"utf8")).join("\n");const themes={};
for(const m of src.matchAll(/:root(?:\[data-theme=.(\w+).\])?\)?\s*\{([^}]*)\}/g)){const t={};
  for(const p of m[2].matchAll(/(--[a-z-]+)\s*:\s*(#[0-9a-f]{6})/gi)) t[p[1].toLowerCase()]=p[2];
  if(Object.keys(t).length) themes[m[1]||"dark(:root)"]=t;}
const L=h=>{const c=[1,3,5].map(i=>parseInt(h.slice(i,i+2),16)/255).map(v=>v<=.03928?v/12.92:((v+.055)/1.055)**2.4);return .2126*c[0]+.7152*c[1]+.0722*c[2]};
const R=(a,b)=>{const x=L(a),y=L(b);return (Math.max(x,y)+.05)/(Math.min(x,y)+.05)};
const PAIRS=[["--text","--bg","page heading"],["--text-secondary","--bg","subtitle/footer"],["--text","--surface","card title"],["--text-secondary","--surface","card domain"],["--pill-text","--pill","inactive tab"],["--accent-text","--accent","ACTIVE tab label"]];
let fails=0;
for(const[name,t]of Object.entries(themes)){console.log("\n"+name.toUpperCase());
  for(const[a,b,label]of PAIRS){if(!t[a]||!t[b])continue;const r=R(t[a],t[b]),ok=r>=4.5;if(!ok)fails++;
    console.log(`  ${ok?"PASS":"FAIL"} ${r.toFixed(2).padStart(6)}:1  ${label}  (${t[a]} on ${t[b]})`);}}
console.log(`\nthemes: ${Object.keys(themes).length}  AA failures: ${fails}`);process.exit(fails?1:0)
'
```

If it reports fewer than 5 themes, it failed to parse — fix the check before trusting it. Note it
compares each theme against its own `--bg`; it cannot tell you whether that `--bg` is ever painted.

You can also inspect the shipped build without a browser: `build/index.html` should contain the
pre-paint script, and `build/_app/immutable/assets/*.css` the theme blocks (minified, so attribute
selectors lose their quotes).

**What this cannot catch** — report these as unverified: rendered colours and how a theme looks
against the OS scheme, any visual regression, the load flash, hover/focus states, runtime console
errors, responsive behaviour, and whether storage failure leaves a hanging spinner.

Close with a short manual checklist the user can run in under two minutes — exact URL, which themes,
which OS scheme, and what correct looks like.

## Reporting

State what you verified and how. Name the themes and colour schemes you actually exercised, and give
real numbers for contrast claims. If you skipped something, say which and why.

If you used the no-browser fallback, lead with that and list what it could not cover. Never let static
checks read as end-to-end verification — "the build passes and contrast is clean" is honest; "the
themes work" is not.

Commits follow Conventional Commits (`feat:`, `fix:`), matching the existing history.
