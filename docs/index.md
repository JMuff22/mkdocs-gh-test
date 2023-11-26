# Mkdocs Github deploy

Examples of deploying an Mkdocs site to Github pages.

## Way 1 - Using `mkdocs gh-deploy`

In your `.github/workflows/ci.yml`:

```yaml
name: ci 
on:
  push:
    branches:
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          git config user.email 41898282+github-actions[bot]@users.noreply.github.com
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV 


      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material 


      - run: mkdocs gh-deploy --force
```

On pushes to the main branch:

1. Checkout at the Github repo
2. Set Git credentials
3. Setup python
4. Cache some artifacts
5. Install dependencies
6. Use `mkdocs gh-deploy`

This follows the documentation found [here](https://www.mkdocs.org/user-guide/deploying-your-docs/). The `ci.yml` is copied from the [Material for Mkdocs documentation](https://squidfunk.github.io/mkdocs-material/publishing-your-site/).

## Way 2 - Using [`JamesIves/github-pages-deploy-action@v4`](https://github.com/JamesIves/github-pages-deploy-action)

In your `.github/workflows/ci.yml`:

```yaml
name: ci 
on:
  push:
    branches:
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV

      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Build docs
        run: |
          mkdocs build

      - name: Deploy to gh-pages 🚀
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          branch: gh-pages
          folder: site
          force: false
```

This has the added benefit of trying nice commit history in your `gh-pages` branch.