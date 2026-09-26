# 🔐 Password Generator

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![License](https://img.shields.io/badge/license-MIT-4c1?style=for-the-badge)

### 🔗 [Live Demo](https://xaviercop.github.io/password-generator/)

## Description

A simple, dependency-free password generator that runs entirely in the browser. Choose your length and character sets, generate a random password, and copy it to the clipboard with one click — no build step, no libraries, no backend.

## Screenshot

![Password Generator screenshot](./screenshots/app.png)

## Features

- 🎚️ **Customizable length** — set any password length (defaults to 16)
- ✅ **Character set toggles** — independently include or exclude:
  - Uppercase letters (`A-Z`)
  - Lowercase letters (`a-z`)
  - Numbers (`0-9`)
  - Symbols (`!@#$%^&*()_+{}[]:;,.<>?`)
- 📋 **Copy to clipboard** — one-click copy with confirmation
- 🎨 **Clean, responsive UI** — centered card layout with gradient accents
- ⚡ **Zero dependencies** — just open the HTML file, nothing to install

## How It Works

Generating a password reads the selected length and character-set checkboxes, builds a combined character pool from whichever sets are ticked, then randomly assembles a password of that length. The copy button selects the output field and writes it straight to the clipboard, with a brief confirmation so you know it worked.

## Tech Stack

| Technology | Purpose |
|---|---|
| HTML5 | Markup and structure |
| CSS3 | Styling, layout, gradient/hover effects |
| JavaScript (Vanilla) | Password generation logic, clipboard handling |
| Google Material Icons | Copy icon (loaded via CDN) |

## Project Structure

```
.
├── index.html   # UI markup and entry point
├── index.js     # Password generation + copy-to-clipboard logic
└── main.css     # Styling
```

## Running Locally

No build tools or install required:

1. Clone or download this repository.
2. Open `index.html` in any modern web browser.

That's it.

## Note on Security

Passwords are generated using JavaScript's `Math.random()`, which is **not cryptographically secure**. This project is intended as a learning/demo tool — for passwords you actually rely on, use a dedicated password manager.

## License

Released under the [MIT License](./LICENSE).
