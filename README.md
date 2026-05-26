# DAM Docs

Internal documentation for the IBM-hosted DAM service.

## Local dev

```
pip install -r requirements.txt
mkdocs serve
```

## Deploy

The site is built locally and deployed to the `gh-pages` branch. GitHub Pages serves from that branch.

```
mkdocs gh-deploy
```

This builds the site and pushes it to `gh-pages`. The live site at https://pages.github.ibm.com/dam-agents/docs/ updates within seconds.
