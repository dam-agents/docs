# DAM Docs

User documentation for [DAM](https://github.com/dam-agents/dam), an open platform for long-lived AI agents.

These docs describe DAM as it runs on the IBM hosted deployment. If you deploy DAM yourself, the
concepts and features are the same, but sign-in, model providers, and the server URL are whatever
your deployment configures. See the [deployment guide](https://github.com/dam-agents/dam#run-your-own)
in the main repository.

## Local dev

```
pip install -r requirements.txt
mkdocs serve
```

## Deploy

Every push to `main` builds the site with `mkdocs build --strict` and publishes it to GitHub Pages
(`.github/workflows/pages.yml`). There is nothing to deploy by hand.

## Contributing

Corrections and improvements are welcome. Open an issue or a pull request on this repository.

## License

[Apache-2.0](LICENSE), the same as the main DAM repository.
