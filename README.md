# Zenn CLI Content Management

This repository contains articles and books published on [Zenn](https://zenn.dev/).

## 🚀 Getting Started

```bash
# Install dependencies
npm install zenn-cli

# Create new article
npx zenn new:article

# Create new book
npx zenn new:book

# Preview content at localhost:8000
npx zenn preview
```

## 📝 Directory Structure

```
.
├── articles/
│   └── *.md
├── books/
│   └── */
└── README.md
```

## 📘 Article Management

- Articles are managed as markdown files in the `articles` directory
- Each article must include frontmatter configuration:
  ```yaml
  ---
  title: ""      # Article title
  emoji: "😸"    # One emoji character for thumbnail
  type: "tech"   # tech/idea
  topics: []     # Up to 5 topics (e.g., ["react", "typescript"])
  published: true # true/false for publishing
  ---
  ```

## 📚 Book Management

- Books are managed in the `books` directory
- Each book requires:
  - `config.yaml` - Book configuration
  - `cover.png` - Book cover image (500x700px recommended)
  - Chapter files (*.md)

## 🔗 Useful Links

- [Zenn CLI Guide](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Markdown Guide](https://zenn.dev/zenn/articles/markdown-guide)

## 📖 License

The content of this repository is licensed under [license name].
