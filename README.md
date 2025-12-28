# Ally Developer Hub

This repository contains the source code for the Ally Developer Hub website, hosted on GitHub Pages at [helloallytech.github.io](https://helloallytech.github.io).

## About

The Ally Developer Hub is the central documentation and contribution portal for the Ally open-source mental health AI platform. It provides:

- **Welcome page** - Introduction to the Ally ecosystem
- **Contributing guide** - Detailed setup instructions for all Ally repositories
- **Issue tracking information** - How to find and work on issues
- **Code standards** - Contribution guidelines and best practices

## What is Ally?

Ally is an open-source mental health AI platform designed to empower counselors and make quality mental health support more accessible. The platform includes:

- **Web applications** (Next.js, React, Vite)
- **Mobile app** (React Native)
- **Backend API** (NestJS)
- **AI services** (Python, FastAPI, LangChain, LiveKit)
- **Infrastructure** (Terraform, Docker)

## Local Development

This site is built with Jekyll and uses GitHub Pages for hosting.

### Prerequisites

- Ruby (2.5.0 or higher)
- Bundler
- Jekyll

### Setup

```bash
# Install dependencies
bundle install

# Run locally
bundle exec jekyll serve

# Visit http://localhost:4000
```

### Making Changes

1. Edit content in `index.md` or `CONTRIBUTING.md`
2. Update styles in `assets/css/style.scss`
3. Modify layout in `_layouts/default.html`
4. Test locally before pushing

## Repository Structure

```
.
├── _config.yml              # Jekyll configuration
├── _layouts/
│   └── default.html         # Site layout template
├── assets/
│   └── css/
│       └── style.scss       # Custom CSS (helloally.ai theme)
├── index.md                 # Welcome page
├── CONTRIBUTING.md          # Contribution guide
├── CNAME                    # Custom domain configuration
└── README.md               # This file
```

## Theme

The site uses a custom theme inspired by [helloally.ai](https://helloally.ai) with:

- **Color palette:** Cream/off-white backgrounds, warm accents (blue, gold, orange)
- **Typography:** Clean, readable fonts with clear hierarchy
- **Tone:** Friendly, conversational, and approachable
- **Design:** Generous whitespace, card-like sections, subtle shadows

## Contributing

To contribute to this documentation site:

1. Fork this repository
2. Create a feature branch (`feat/improve-docs`)
3. Make your changes
4. Test locally
5. Submit a pull request

For contributing to other Ally projects, see the [Contributing Guide](https://helloallytech.github.io/CONTRIBUTING.html).

## License

This documentation site is open source under the MIT License, consistent with all Ally projects.

## Links

- **Main Site:** [helloally.ai](https://helloally.ai)
- **GitHub Organization:** [HelloAllyTech](https://github.com/HelloAllyTech)
- **Developer Hub:** [helloallytech.github.io](https://helloallytech.github.io)

---

*Built with Jekyll and hosted on GitHub Pages*
