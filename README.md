# Sonar Coding Loops: interactive presentation

An alternative to slide decks for Sonar Solutions Engineers: a single HTML page to introduce Sonar to a customer, then go into detail on each product only when the customer wants to.

- **Introduction**: an overview of the three verification-powered coding loops (Agentic, CI verification, Code maintenance), presented step by step.
- **Product detail**: each product opens a panel with what it does, where it runs (SonarQube Server / Cloud), and a key stat.

## Presenting

Serve the folder over HTTP (opening the file directly won't work) and open it in a browser:

```bash
python3 -m http.server
# http://localhost:8000/
```

An internet connection is required (Three.js and fonts are loaded from CDNs).

| Action | Control |
|---|---|
| Next / previous step | `→` / `←` (or Space, PageDown / PageUp) |
| Zoom out one level | `Esc`, or double-click |
| Overview / jump to a loop | `0` / `1` `2` `3` |
| Fullscreen | `F` |
| Free navigation | Scroll to zoom, drag to pan, click a loop or product |

Deep links open a specific view directly: `#2` (loop 2), `#2.1` (loop 2, first product). Add `?instant` to skip the intro animation.

## Editing the content

All text lives in the `LOOPS` array at the top of the script in `index.html` (the "Content — edit here" block). All of it is shown to customers, and every product claim must come from an official Sonar source.
