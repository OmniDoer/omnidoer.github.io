# OmniDoer GitHub Page

This repository hosts the OmniDoer project landing page at:

- https://omnidoer.github.io/

It is a single static page (`index.html`) with multi-language support (English, 中文, Español, Français, Deutsch, 日本語, 한국어).

## One-command install

```bash
curl -fsSL https://raw.githubusercontent.com/omnidoer/omnidoer/main/omnidoer/scripts/install-cloud-direct.sh | sh
```

Cloud Direct deployment:

```bash
curl -fsSL https://raw.githubusercontent.com/omnidoer/omnidoer/main/omnidoer/scripts/install-cloud-direct.sh | \
  OMNIDOER_CLOUD_DIRECT=1 OMNIDOER_PUBLIC_URL=https://agent.example.com sh
```

## One-command page preview

```bash
cd omnidoer.github.io
python3 -m http.server 4173
```

Then open `http://localhost:4173` in a browser.

## Push and publish pages

```bash
git add index.html assets
git commit -m "Update landing page"
git push
```

If this repo is `omnidoer.github.io` and GitHub Pages source is set to branch `main` root, updates are published automatically.

## Development story line

- [第一夜：OmniDoer 开始把自己搬家](artical/omnidoer-moves-itself.md)
