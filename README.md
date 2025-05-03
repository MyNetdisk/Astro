## Github Action
```yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: withastro/action@v3

      - name: Upload dist/ as artifact
        uses: actions/upload-artifact@v4
        with:
          name: astro-dist
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout (dummy, required by gh-pages action)
        uses: actions/checkout@v4

      - name: Download dist artifact
        uses: actions/download-artifact@v4
        with:
          name: astro-dist
          path: dist

      - name: Show downloaded files
        run: ls -R dist

      - name: Deploy to GitHub Pages repo
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.REPO_TOKEN }}
          publish_dir: dist
          external_repository: MyNetdisk/MyNetdisk.github.io
          publish_branch: master
          force_orphan: true
```
## CNAME配置
```
myselfdisk.filegear-sg.me
```