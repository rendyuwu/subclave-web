# subclave-web

Landing page for [Subclave](https://github.com/rendyuwu/subclave), a local-first desktop password manager.

Live at <https://subclave.rendy.dev/>.

Looking for the app, releases or issues? Go to [rendyuwu/subclave](https://github.com/rendyuwu/subclave).

## Local preview

Plain HTML, CSS and JS with no build step. Serve the repo root:

```bash
python3 -m http.server 4173
```

Then open <http://127.0.0.1:4173/>. Download links resolve to the latest [release](https://github.com/rendyuwu/subclave/releases/latest) through the GitHub API when the page loads.
