# somonox blog

A portfolio blog built with Jekyll. Features encrypted posts, dark mode, and a clean design.

## Features
- **Portfolio Design** — Hero section, category filters, animated post cards
- **Dark / Light Mode** — Toggle or auto-detect system preference
- **Encrypted Posts** — AES-256 client-side encryption
- **Responsive** — Mobile-first with glassmorphism header

## Local Development

```bash
bundle install
bundle exec jekyll serve
```

## Creating Encrypted Posts

1. Open `scripts/encrypt.html` in your browser
2. Write content in Markdown and enter a password
3. Click **Encrypt** and copy the output
4. Create a new file in `_posts/` with the encrypted content

## Structure
- `_layouts/` — HTML templates (default, home, post, encrypted)
- `_posts/` — Blog posts
- `assets/css/` — Stylesheet
- `scripts/` — Encryption tool
