# chandanv89.github.io

Personal blog and portfolio site for **Chandan Veerabhadrappa** — built with [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/).

## 🔗 Live Site

**[chandanv89.github.io](https://chandanv89.github.io/)**

## Tech Stack

| Layer     | Technology                                                                                                                              |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Generator | [Jekyll 3.10+](https://jekyllrb.com/) via [GitHub Pages](https://pages.github.com/)                                                     |
| Theme     | [Minima 2.5](https://github.com/jekyll/minima/tree/v2.5.1) (auto light/dark)                                                            |
| Math      | [MathJax 3](https://www.mathjax.org/) — conditional per-post via `math: true`                                                           |
| Diagrams  | [Mermaid 11](https://mermaid.js.org/) — fenced code blocks                                                                              |
| Comments  | [Giscus](https://giscus.app/) — powered by GitHub Discussions                                                                           |
| Fonts     | [Inter](https://rsms.me/inter/) · [Lora](https://fonts.google.com/specimen/Lora) · [Monaspace Xenon](https://monaspace.githubnext.com/) |
| Icons     | [Font Awesome 7](https://fontawesome.com/)                                                                                              |
| Markdown  | [Kramdown](https://kramdown.gettalong.org/) with GFM input                                                                              |
| Syntax    | [Rouge](https://github.com/rouge-ruby/rouge)                                                                                            |

## Blog Posts

- **SciCalc** — A full scientific calculator in C (recursive-descent parser, 158 tests, 13 modules)
- **Payment Gateways** — A system design perspective (ISO 8583, PCI DSS, settlement flows)
- **Dockerize Spring Boot** — Step-by-step Docker containerization guide
- **Docker Hub Automated Builds** — CI/CD pipeline for Docker images

## Local Development

### Prerequisites

- Ruby 2.7+ with Bundler
- GCC / Make (for native gem extensions)

### Setup & Run

```bash
# Clone the repo
git clone https://github.com/chandanv89/chandanv89.github.io.git
cd chandanv89.github.io

# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve --livereload
```

The site will be available at **http://localhost:4000**.

> **Note:** Changes to `_config.yml` require restarting the server.

### Build Only

```bash
bundle exec jekyll build
```

Output goes to `_site/`.

## Project Structure

```
├── _config.yml          # Site configuration
├── _includes/           # Reusable HTML partials (head, footer, comments, etc.)
├── _layouts/            # Page templates (base, home, post, page)
├── _posts/              # Blog posts (Markdown)
├── _sass/minima/        # Custom SCSS overrides and variables
├── assets/              # Static assets (CSS, images, fonts)
├── about.markdown       # About page
├── work-experience.markdown
├── tags.html            # Tags index page (filterable by hash)
├── index.markdown       # Home page
└── Gemfile              # Ruby dependencies
```

## Customizations Over Base Minima

- Auto light/dark theme with `prefers-color-scheme` sync
- SVG favicon with dark mode support (CV monogram); dynamic header logo (icon + name on homepage, icon-only elsewhere)
- Reading progress bar on posts (brand color → green gradient)
- Collapsible Table of Contents (auto-generated from headings)
- Estimated read time in post metadata
- Heading anchor links (click-to-copy URL)
- Code block line numbers with one-click copy button
- MathJax 3 for LaTeX rendering (per-post opt-in via `math: true`)
- Mermaid diagram support
- Giscus comments with real-time theme switching
- Custom fonts: Inter (headings/UI), Lora (body), Monaspace Xenon (code)
- **Tag pills on posts** — clickable, styled pill badges rendered from frontmatter `tags`; link to the tags index page filtered by tag
- **Tags index page** (`/tags/`) — displays all tags as a cloud with post counts; each tag section lists its posts with dates; URL hash filtering shows only the selected tag's posts

## License

Content &copy; Chandan Veerabhadrappa. The site theme is based on [Minima](https://github.com/jekyll/minima), licensed under the [MIT License](https://opensource.org/licenses/MIT).
