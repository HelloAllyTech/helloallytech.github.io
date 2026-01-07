---
layout: default
title: Get Started
---

# Get Started

Ready to contribute? Here's how to get set up:

## Quick Start

1. **Clone the infrastructure repo** (easiest way to get everything):
   ```bash
   git clone git@github.com:HelloAllyTech/infra.git
   cd infra
   ./bootstrap.sh
   ```
   This will clone and set up all Ally repositories for you!

2. **Choose your area of interest** and navigate to that repository

3. **Check out our [Contributing Guide](./CONTRIBUTING.html)** for detailed setup instructions for each repo

## First Time Here?

Not sure where to start? Here are some suggestions:

- **Frontend developers**: Check out [ally-web](https://github.com/HelloAllyTech/ally-web) for React/Next.js work
- **Mobile developers**: Head to [ally-mobile](https://github.com/HelloAllyTech/ally-mobile) for React Native
- **Backend developers**: Explore [ally-be](https://github.com/HelloAllyTech/ally-be) for API development
- **AI/ML enthusiasts**: Dive into [ally-ai](https://github.com/HelloAllyTech/ally-ai) or [ally-ai-learn](https://github.com/HelloAllyTech/ally-ai-learn)
- **DevOps engineers**: Start with [infra](https://github.com/HelloAllyTech/infra)

## How We Track Issues

We use GitHub Issues across all our repositories to track bugs, feature requests, and improvements. Here's how it works:

- **Repository-specific issues**: Each repo has its own issue tracker for problems specific to that codebase
- **Labels**: We use labels like `bug`, `enhancement`, `good first issue`, `help wanted`, etc.
- **Good First Issues**: Look for the `good first issue` label to find beginner-friendly tasks
- **Milestones**: We organize work into milestones for better project planning

### Finding issues to work on:
1. Browse issues in the repository you're interested in
2. Look for `good first issue` or `help wanted` labels
3. Check if someone is already assigned
4. Comment on the issue to express interest before starting work

## Development Environment

Each repository has its own setup instructions, but here are the common requirements:

### General Requirements
- Git
- Docker and Docker Compose (recommended via Colima on macOS)
- Your preferred code editor (VS Code recommended)

### Repository-Specific Setup
Visit each repository's README for detailed setup instructions:
- [ally-web](https://github.com/HelloAllyTech/ally-web) - Node.js, npm/yarn
- [ally-mobile](https://github.com/HelloAllyTech/ally-mobile) - React Native CLI, Xcode/Android Studio
- [ally-be](https://github.com/HelloAllyTech/ally-be) - Node.js, PostgreSQL, Redis
- [ally-ai](https://github.com/HelloAllyTech/ally-ai) - Python 3.12, Poetry
- [ally-ai-learn](https://github.com/HelloAllyTech/ally-ai-learn) - Python 3.12, LiveKit
- [infra](https://github.com/HelloAllyTech/infra) - Terraform, Docker

## macOS Development Setup (Optional)

If you're developing on macOS, you can use **Colima** as a lightweight, free alternative to Docker Desktop for container virtualization.

### Why Colima?
- **Free and open-source** - No licensing costs
- **Lightweight** - Uses fewer resources than Docker Desktop
- **Fast** - Quick startup and container operations
- **Compatible** - Works seamlessly with all Docker commands

### Quick Setup

Our centralized Colima setup is available in the [infra repository](https://github.com/HelloAllyTech/infra/tree/main/colima). You can install it with a single command:

```bash
# One-time global installation
curl -fsSL https://raw.githubusercontent.com/HelloAllyTech/infra/main/colima/install.sh | bash
```

This will:
- Install Colima and required dependencies
- Set up the global `docker-switch` command
- Configure your Docker environment for Colima

### Usage

After installation, you can easily switch between Docker environments:

```bash
# Switch to Colima (lightweight)
docker-switch colima

# Switch to Docker Desktop (if you have it installed)
docker-switch desktop
```

### Documentation

For complete setup instructions, troubleshooting, and advanced configuration:
- [Colima Setup Guide](https://github.com/HelloAllyTech/infra/tree/main/colima)

## Need Help?

- **Documentation**: Each repository has its own README with detailed setup instructions
- **Issues**: Ask questions by creating an issue in the relevant repository
- **Contributing Guide**: Check our [Contributing Guide](./CONTRIBUTING.html) for detailed information

---

**Explore our [Tech Stack](/tech-stack.html)** to learn more about the technologies we use.
