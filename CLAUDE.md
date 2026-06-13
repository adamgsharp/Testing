# Family Tree Visualizer

A single-file web app (`index.html`) that lets users build and print a family tree. No build step, no dependencies — just open the file in a browser.

## What it does

- Add people (name, birth/death year, gender, notes)
- Add relationships: spouse/partner or parent→child
- Auto-layouts the tree top-to-bottom by generation
- Decorative vine/branch corners, printable on 8.5×11" paper
- Data persists in browser `localStorage` under the key `ft2`

## File structure

```
index.html          — the entire app (HTML + CSS + JS, ~900 lines)
.github/workflows/
  deploy-pages.yml  — GitHub Actions workflow to publish to GitHub Pages
```

## How the code is organized (inside index.html)

| Section | What it does |
|---|---|
| `<style>` | Screen layout, modals, print CSS (`@media print`) |
| `#controls` bar | Sticky top bar: tree name input, Add Person, Add Relationship, Print |
| `#tree-svg` | SVG canvas — redrawn from scratch on every state change |
| Modal HTML | Two modals: add/edit person, add relationship |
| **STATE** | `state = { treeName, people[], relationships[] }` — plain JS object |
| **PERSISTENCE** | `persist()` / `loadSaved()` — JSON in localStorage |
| **LAYOUT** | `computeLayout()` — recursive subtree-width algorithm, returns a `Map<id, {x,y,w,h}>` |
| **RENDER** | `render()` → `drawShell()` (border + vines + title) → `drawConnections()` → `drawCard()` per person |

## Key data shapes

```js
// Person
{ id: 'p<timestamp>', name, birth, death, gender: ''|'male'|'female', notes }

// Relationship
{ id: 'r<timestamp>', type: 'spouse'|'parent', from: personId, to: personId }
// 'parent' means: from=parent, to=child
```

## Layout algorithm

`computeLayout()` in ~60 lines:
1. Builds `spouseOf`, `childrenOf`, `parentsOf` maps from `state.relationships`
2. Groups people into "units" (a person + their spouse, if any)
3. Recursively calculates `unitWidth()` — each unit is as wide as the wider of (self, sum of children)
4. Places root units centered at SVG x=408, then places children below each unit centered within its allocated width
5. Any unconnected people are placed in a row at the top as a fallback

## Deployed URL

`https://adamgsharp.github.io/Testing/`

Source branch for Pages: `gh-pages` (configured in repo Settings → Pages)

## Things to know

- The SVG redraws completely on every change — no diffing. Fine for typical family sizes.
- Card width (`CW = 128px`) and vertical gap (`VGAP = 88px`) are constants at the top of the JS — easy to tune.
- Names longer than 16 chars are split across two lines at the word midpoint.
- Right-clicking a card opens a delete option; left-clicking opens edit.
- The `gh-pages` branch contains only `index.html` (pushed via GitHub MCP). The feature branch `claude/family-tree-viz-v7tob6` has the full history including the Actions workflow.
