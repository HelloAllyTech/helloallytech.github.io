---
layout: default
title: Contributing to Ally
---

# Contributing to Ally

Thank you for your interest in contributing to Ally! This guide will help you get started with setting up each repository, understanding our development workflow, and making your first contribution.

## Table of Contents

- [General Guidelines](#general-guidelines)
- [Repository Setup Guides](#repository-setup-guides)
  - [Ally Web](#ally-web)
  - [Ally Mobile](#ally-mobile)
  - [Ally Backend](#ally-backend)
  - [Ally AI](#ally-ai)
  - [Ally AI Learn](#ally-ai-learn)
  - [Infrastructure](#infrastructure)
- [Development Workflow](#development-workflow)
- [Code Standards](#code-standards)
- [Pull Request Process](#pull-request-process)

---

## General Guidelines

All Ally repositories follow the same contribution standards:

### Branch Naming Convention

```
<type>/<short-description>
```

**Branch Types:**
- `feat` - New feature implementation
- `fix` - Bug fixes
- `chore` - Maintenance tasks, dependency updates
- `refactor` - Code restructuring without changing functionality
- `docs` - Documentation updates
- `test` - Adding or updating tests
- `style` - Code formatting (not CSS)
- `perf` - Performance improvements
- `build` - Build system changes
- `ci` - CI/CD pipeline changes
- `revert` - Reverting previous commits
- `hotfix` - Critical production fixes

**Examples:**
```
feat/add-user-authentication
fix/login-error-handling
docs/update-setup-guide
```

### Commit Message Format

Follow the Conventional Commits specification:

```
<type>: short summary
```

**Rules:**
- Use imperative mood ("add" not "added" or "adds")
- First letter lowercase after the type
- Keep summary short and descriptive (50 chars or less)
- No period at the end
- Be specific about what changed

**Examples:**
```
feat: add user profile page
fix: handle null pointer in login flow
refactor: optimize database queries
docs: update contributing guidelines
test: add unit tests for authentication
```

---

## Repository Setup Guides

### Ally Web

**Repository:** [https://github.com/HelloAllyTech/ally-web](https://github.com/HelloAllyTech/ally-web)

**What it is:** Monorepo containing the Ally landing page (Next.js), helpline dashboard, and admin dashboard (both Vite).

#### Prerequisites
- Node.js (LTS version recommended)
- npm or yarn
- Git

#### Setup Steps

For a full-stack development environment that includes the backend, you can use the unified development script.

**Quick Setup (Web & Backend):**

If you have all the repositories cloned in the same parent directory, you can start both `ally-web` and `ally-be` with a single command from the project root:

```bash
./infra/dev_env.sh
```

This will start all necessary Docker containers for both projects.

**Manual Setup (Frontend Only):**

If you want to run only the frontend services:

```bash
# 1. Clone the repository
git clone git@github.com:HelloAllyTech/ally-web.git
cd ally-web

# 2. Install dependencies
npm install

# 3. Start development server
npm run start:web          # Ally Web (Next.js on port 3000)
npm run start:helpline     # Helpline Dashboard (Vite)
npm run start:admin        # Admin Dashboard (Vite)
```

#### Useful Commands

```bash
# Building
npm run build:web          # Build Ally Web
npm run build:helpline     # Build Helpline Dashboard
npm run build:admin        # Build Admin Dashboard

# Testing
npm run test:web           # Run tests for Ally Web
npm run test:helpline      # Run tests for Helpline Dashboard
npm run test:admin         # Run tests for Admin Dashboard
npm run test:coverage      # Generate coverage report

# Code Quality
npm run lint               # Check all code
npm run lint:fix           # Fix linting issues
npm run format             # Format with Prettier
npm run format:check       # Check formatting
```

**Dockerized Testing:**

For running tests within Docker containers, use the `./test-docker.sh` script:

```bash
./test-docker.sh all         # Run all tests in parallel
./test-docker.sh web         # Run ally-web tests
./test-docker.sh helpline    # Run ally-helpline-dashboard tests
./test-docker.sh admin       # Run ally-admin-dashboard tests
./test-docker.sh coverage    # Generate coverage report
./test-docker.sh watch       # Run tests in watch mode
```

#### Tech Stack
- React 18, Next.js 14, Vite 5
- TypeScript
- Tailwind CSS, Material-UI
- Redux Toolkit, React Router
- Nx (monorepo management)

#### Code Style
- Semicolons: Required
- Quotes: Double quotes
- Tab width: 2 spaces
- Print width: 100 characters
- Import order: External → Internal → Relative → Type imports

---

### Ally Mobile

**Repository:** [https://github.com/HelloAllyTech/ally-mobile](https://github.com/HelloAllyTech/ally-mobile)

**What it is:** React Native mobile app for iOS and Android enabling secure counselor-client communication.

#### Prerequisites
- Node.js >= 18
- Ruby (for iOS)
- Xcode (for iOS development)
- Android Studio (for Android development)
- CocoaPods

#### Setup Steps

```bash
# 1. Clone the repository
git clone git@github.com:HelloAllyTech/ally-mobile.git
cd ally-mobile

# 2. Install dependencies
npm install

# 3. iOS-specific setup
bundle install
bundle exec pod install

# 4. Configure environment variables
cp env/.env.example .env
# Edit .env with your configuration

# 5. Add Firebase configuration
# iOS: Add GoogleService-Info.plist to ios/
# Android: Add google-services.json to android/app/
```

#### Running the App

```bash
# Start Metro bundler
npm start

# iOS
npm run ios

# Android
npm run android
npm run android:dev      # Dev variant
npm run android:prod     # Production variant
```

#### Environment Variables

Required in `.env`:
- `ENVIRONMENT` - development or production
- `API_BASE_URL` - Backend API base URL
- `API_VERSION` - API version (default: v1)
- `ALLY_WEB_URL` - Web app URL for deep linking
- `MICROPHONE_SOCKET_PATH` - Socket path (default: /microphone-chat)
- `CLOUD_TELEPHONY_PATH` - Telephony socket path (default: /cloud-telephony-chat)
- `TERMS_AND_CONDITION_FLAG` - Enable/disable T&C flow

#### Useful Commands

```bash
npm run lint               # Run ESLint
npm test                   # Run Jest tests
npm run test:coverage      # Generate coverage
npm run format             # Format with Prettier
npm run build:dev          # Build Android dev release
npm run build:prod         # Build Android prod release
```

#### Tech Stack
- React Native 0.79, TypeScript
- Redux Toolkit, React Navigation
- LiveKit (WebRTC), Socket.IO
- Firebase Crashlytics

#### Code Standards
- Import organization strictly enforced (see CONTRIBUTING.md in repo)
- Husky + lint-staged for pre-commit checks
- Conventional commits required

---

### Ally Backend

**Repository:** [https://github.com/HelloAllyTech/ally-be](https://github.com/HelloAllyTech/ally-be)

**What it is:** NestJS backend API powering the Ally platform with real-time communication.

#### Prerequisites
- Node.js v18
- npm
- Docker v20.10+
- Docker Compose v2.0+
- PostgreSQL v14+ (optional with Docker)
- Redis (optional with Docker)

#### Setup Steps

For a full-stack development environment that includes the frontend, you can use the unified development script.

**Quick Setup (Backend & Web):**

If you have all the repositories cloned in the same parent directory, you can start both `ally-be` and `ally-web` with a single command from the project root:

```bash
./infra/dev_env.sh
```

This will start all necessary Docker containers for both projects.

**Manual Setup (Backend Only):**
```bash
# 1. Clone the repository
git clone git@github.com:HelloAllyTech/ally-be.git
cd ally-be

# 2. Setup environment files
cp docker.env.example docker.env
cp .env.example .env
# Edit both files with your configuration

# 3. Start Docker services (PostgreSQL, Redis, LocalStack)
docker-compose up

# 4. Run database migrations
npm run migration:run

# 5. (Optional) Seed the database
npm run seed -- src/database/seeds/admin_user.ts
```

#### Running the App

**Option A: With Docker (Recommended)**
```bash
docker-compose up
# App runs at http://localhost:8001
```

**Option B: Direct Node.js**
```bash
npm install
npm run migration:run
npm run start:dev          # Development mode
npm run start:prod         # Production mode
```

#### API Access Points
- Swagger Docs: http://localhost:8001/api-docs
- Health Check: http://localhost:8001/api/health
- API Base: http://localhost:8001/api/v1

#### Useful Commands

```bash
npm run start:dev          # Development with watch mode
npm run start:debug        # Debug mode
npm run lint               # Check linting
npm run lint:fix           # Auto-fix linting issues
npm run format             # Format with Prettier
npm run test               # Run unit tests
npm run test:cov           # Test coverage
npm run migration:run      # Run migrations
npm run migration:show     # View migration status
```

**Dockerized Testing:**

For running tests within Docker containers, use the `./test-docker.sh` script:

```bash
./test-docker.sh all         # Run all unit tests
./test-docker.sh e2e         # Run end-to-end tests
./test-docker.sh coverage    # Generate coverage report
./test-docker.sh watch       # Run tests in watch mode
```

#### Tech Stack
- NestJS, TypeScript
- PostgreSQL, Redis
- Socket.io (WebSocket)
- AWS (SQS, S3, SES)
- LiveKit, Deepgram
- JWT + OTP authentication

---

### Ally AI

**Repository:** [https://github.com/HelloAllyTech/ally-ai](https://github.com/HelloAllyTech/ally-ai)

**What it is:** AI-powered copilot service for mental health counselors.

#### Prerequisites
- Python 3.12+
- Poetry
- Docker and Docker Compose

#### Setup Steps

```bash
# 1. Clone the repository
git clone git@github.com:HelloAllyTech/ally-ai.git
cd ally-ai

# 2. Install dependencies
poetry install

# 3. Configure environment
cp env_sample .env
# Edit .env with your API keys

# 4. Install pre-commit hooks
pre-commit install

# 5. Set up database
poetry run python scripts/migrate.py all
```

#### Required Environment Variables
- `OPENAI_API_KEY` - OpenAI API key
- `DEEPGRAM_API_KEY` - Deepgram API key
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` - AWS credentials
- Weaviate configuration
- Slack alerts (optional)

#### Running the App

**Local Development:**
```bash
# Start Weaviate
docker-compose up -d

# Start the application
poetry run python app/main.py

# Access at:
# - API: http://localhost:8000
# - Docs: http://localhost:8000/docs
```

**Full Docker Stack:**
```bash
docker compose -f docker-compose.full.yml up --build
# App at http://localhost:8001
```

#### Useful Commands

```bash
# Testing
poetry run pytest
poetry run pytest --cov=app --cov-report=html

# Code Quality
poetry run black app/
poetry run isort app/
poetry run flake8 app/
pre-commit run --all-files

# Database
poetry run python scripts/migrate.py status
poetry run python scripts/migrate.py up
poetry run python scripts/migrate.py down
poetry run python scripts/migrate.py generate "migration-name"
```

#### Tech Stack
- Python 3.12, FastAPI
- LangChain, OpenAI
- Weaviate (vector database)
- Deepgram (speech-to-text)
- AWS (S3, SQS)

#### Code Standards
- Black (88 char line length)
- isort (Black profile)
- flake8
- Type hints required
- Comprehensive docstrings

---

### Ally AI Learn

**Repository:** [https://github.com/HelloAllyTech/ally-ai-learn](https://github.com/HelloAllyTech/ally-ai-learn)

**What it is:** LiveKit-based AI agent for counselor training with simulated client conversations.

#### Prerequisites
- Python 3.12+
- Poetry
- Docker and Docker Compose (optional)

#### Setup Steps

```bash
# 1. Clone the repository
git clone git@github.com:HelloAllyTech/ally-ai-learn.git
cd ally-ai-learn

# 2. Install dependencies
poetry install

# 3. Configure environment
cp env_sample .env
# Edit .env with your configuration
```

#### Required Environment Variables
- `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, `LIVEKIT_URL` - LiveKit credentials
- `OPENAI_API_KEY` - OpenAI API key
- `DEEPGRAM_API_KEY` - Deepgram API key
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` - AWS credentials
- `SQS_MESSAGE_EVENT_QUEUE_NAME` - SQS queue name
- TTS providers (ElevenLabs, Sarvam, Google, Hume) - optional

#### Running the App

**Option 1: Manual (Two Terminals)**
```bash
# Terminal 1: Start LiveKit worker
poetry run python -m app.worker dev

# Terminal 2: Start FastAPI server
poetry run uvicorn app.main:app --reload --port 8000
```

**Option 2: Startup Script**
```bash
./start.sh
```

**Option 3: Docker**
```bash
docker build -t ally-learn-core .
docker run -p 8000:8000 --env-file .env ally-learn-core
```

**Option 4: With LocalStack**
```bash
docker-compose up -d
```

#### Useful Commands

```bash
# Testing
poetry run pytest
./run_tests.sh

# Code Quality
poetry run black app/
poetry run isort app/
poetry run flake8 app/
pre-commit install
pre-commit run --all-files
```

#### Tech Stack
- Python 3.12, FastAPI
- LiveKit (voice agents)
- LangGraph, OpenAI
- Deepgram (STT), Multiple TTS providers
- AWS SQS

#### Code Standards
- Black (88 char line length)
- isort (Black profile)
- flake8
- Pre-commit hooks enforced
- All tests must pass

---

### Infrastructure

**Repository:** [https://github.com/HelloAllyTech/infra](https://github.com/HelloAllyTech/infra)

**What it is:** Central hub for infrastructure management and development environment setup.

#### Prerequisites
- Git
- Bash shell (macOS/Linux)
- SSH access to HelloAllyTech repositories

#### Setup

```bash
# Clone and run bootstrap script
git clone git@github.com:HelloAllyTech/infra.git
cd infra
./bootstrap.sh
```

The bootstrap script will:
- Clone or update all Ally repositories
- Check for uncommitted changes
- Fetch latest changes from remotes
- Provide colored output for status

#### Colima Setup (macOS)

For lightweight Docker environment:
```bash
./colima.sh
```

Configuration:
- 10 CPUs
- 24GB memory
- 48GB disk
- ARM64 architecture

#### Tech Stack
- Terraform (Infrastructure as Code)
- Docker
- Bash scripts

---

## Development Workflow

### Making Your First Contribution

1. **Fork the repository** you want to contribute to (or clone if you have access)

2. **Create a feature branch** following our naming convention:
   ```bash
   git checkout -b feat/your-feature-name
   ```

3. **Make your changes** following the code standards for that repository

4. **Test your changes** thoroughly:
   ```bash
   # Run tests
   npm test        # For JavaScript/TypeScript
   poetry run pytest   # For Python

   # Run linters
   npm run lint
   poetry run flake8 app/
   ```

5. **Commit your changes** using conventional commits:
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

6. **Push to your fork**:
   ```bash
   git push origin feat/your-feature-name
   ```

7. **Create a Pull Request** on GitHub

### Working on Issues

1. Browse issues in the repository you're interested in
2. Look for `good first issue` or `help wanted` labels
3. Comment on the issue to express interest
4. Wait for maintainer approval before starting work
5. Reference the issue in your PR description (e.g., "Closes #123")

---

## Code Standards

### All Repositories

**General Principles:**
- Write clean, readable, self-documenting code
- Add comments only where logic isn't self-evident
- Follow the existing code style in each repository
- Write tests for new functionality
- Update documentation when adding features

**Pre-commit Checks:**
All repositories use pre-commit hooks that automatically check:
- Code formatting
- Linting
- Tests (in some repos)
- Import ordering
- Trailing whitespace

### JavaScript/TypeScript Repositories

**Import Order:**
1. Built-in Node.js modules
2. External dependencies (React first)
3. Internal modules (using @ alias)
4. Parent/sibling imports (relative)
5. Type imports

Separate groups with newlines, alphabetize within groups.

### Python Repositories

**Import Order (isort):**
1. Standard library imports
2. Third-party imports
3. Local application imports

**Code Style:**
- Black formatting (88 char line length)
- Type hints for all functions
- Comprehensive docstrings (Google style)

---

## Pull Request Process

### PR Title Format

Use the same format as commit messages:
```
<type>: short summary
```

Examples:
```
feat: add user authentication
fix: resolve login timeout issue
docs: update contributing guide
```

### PR Description Template

Use this template for your PR description:

```markdown
## Summary
Brief description of what this PR does.

## Changes
- Changed X to improve Y
- Added Z feature
- Fixed bug in A

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## Related Issues
Closes #123
Related to #456

## Screenshots (if applicable)
[Add screenshots for UI changes]

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added where needed
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests pass locally
```

### Review Process

1. **Submit PR** with clear title and description
2. **Automated checks** will run (tests, linting, etc.)
3. **Wait for review** from maintainers
4. **Address feedback** by pushing new commits
5. **Get approval** (at least one required)
6. **Merge** will be handled by maintainers

### PR Best Practices

- Keep PRs focused on a single feature or fix
- Keep PRs reasonably sized (< 500 lines when possible)
- Write clear, descriptive commit messages
- Update tests and documentation
- Respond to review feedback promptly
- Be respectful and professional in discussions

---

## Getting Help

### Documentation
- Each repository has detailed README files
- Check the repo's CONTRIBUTING.md for specific guidelines
- Review existing issues and PRs

### Ask Questions
- Create an issue in the relevant repository
- Tag it with `question` label
- Be specific about your problem or confusion

### Community
- Be respectful and welcoming to all contributors
- Help newcomers get started
- Share knowledge and best practices

---

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. All contributors are expected to:

- **Be respectful** - Treat everyone with respect and consideration
- **Be inclusive** - Welcome diverse perspectives and experiences
- **Be collaborative** - Work together and help each other
- **Be professional** - Keep discussions focused and constructive
- **Be patient** - Everyone is learning and growing

Unacceptable behavior includes:
- Harassment or discrimination of any kind
- Trolling, insulting, or derogatory comments
- Personal or political attacks
- Publishing others' private information
- Any conduct that would be inappropriate in a professional setting

---

## License

All Ally projects are licensed under the **MIT License**. By contributing, you agree that your contributions will be licensed under the same license.

---

## Thank You!

Your contributions make Ally better for everyone. Whether you're fixing a bug, adding a feature, improving documentation, or helping other contributors, you're making a real difference in building tools that empower mental health professionals.

**Happy coding, and thank you for being part of the Ally community!** 🎉

---

*Last updated: December 2025*
