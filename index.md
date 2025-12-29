---
layout: default
title: Welcome to Ally Developer Hub
---

# Welcome to Ally's Developer Hub 👋

Hello and welcome! We're excited that you're here. Ally is an open-source mental health AI platform designed to empower counselors and make quality mental health support more accessible to everyone.

## What is Ally?

Ally is a comprehensive platform that combines AI technology with human expertise to support mental health professionals in their daily practice. Think of it as a copilot for counselors - providing intelligent insights, real-time assistance, and training tools to help mental health professionals deliver better care.

### Our Mission

We believe that mental health support should be accessible, effective, and empowering for both counselors and clients. By combining cutting-edge AI with thoughtful design, we're building tools that:

- **Support counselors** with real-time insights and AI-powered assistance
- **Enable training** through realistic AI-simulated client conversations
- **Ensure privacy** with HIPAA-compliant, secure communication
- **Provide accessibility** through web, mobile, and voice interfaces

## The Ally Ecosystem

Our platform is built across several repositories, each serving a specific purpose:

### 🌐 [Ally Web](https://github.com/HelloAllyTech/ally-web)
A modern monorepo containing our web applications built with React, Next.js, and Vite.

**What's inside:**
- **Landing Page** - Showcase the Ally platform and its mission
- **Helpline Dashboard** - For mental health professionals to manage sessions
- **Admin Dashboard** - Super admin tools for user and simulation management

**Tech:** React 18, Next.js 14, Vite 5, TypeScript, Tailwind CSS, Redux Toolkit

---

### 📱 [Ally Mobile](https://github.com/HelloAllyTech/ally-mobile)
Native mobile app for iOS and Android that enables secure counselor-client communication.

**Features:**
- Secure, privacy-first counseling sessions
- Live audio streaming with transcription
- Detailed call summaries and analytics
- Real-time communication via LiveKit

**Tech:** React Native 0.79, TypeScript, Redux Toolkit, LiveKit, Socket.IO

---

### 🚀 [Ally Backend](https://github.com/HelloAllyTech/ally-be)
The core API and real-time communication server powering the Ally platform.

**Capabilities:**
- RESTful API for all platform features
- WebSocket support for real-time sessions
- AI-powered transcription and analysis
- HIPAA-compliant audit logging
- Multi-tenant organization support

**Tech:** NestJS, TypeScript, PostgreSQL, Redis, Socket.io, AWS (SQS, S3, SES)

---

### 🤖 [Ally AI](https://github.com/HelloAllyTech/ally-ai)
AI-powered mental health support service that acts as a copilot for counselors.

**What it does:**
- Intelligent conversation analysis
- Real-time insights and suggestions
- Vector-based semantic search
- Sentiment analysis and pattern detection

**Tech:** Python 3.12, FastAPI, LangChain, OpenAI, Weaviate, Deepgram

---

### 🎓 [Ally AI Learn](https://github.com/HelloAllyTech/ally-ai-learn)
LiveKit-based AI agent for counselor training through simulated client conversations.

**Training features:**
- Realistic AI client simulations
- Real-time event detection (identifying counseling skills)
- Immediate scoring and feedback
- Multiple client personas and scenarios

**Tech:** Python 3.12, FastAPI, LiveKit, LangGraph, OpenAI, Deepgram

---

### ⚙️ [Infrastructure](https://github.com/HelloAllyTech/infra)
Central hub for infrastructure management and development environment setup.

**Includes:**
- Automated repository setup scripts
- Infrastructure as Code (Terraform)
- Docker and Colima configurations
- Development environment bootstrapping

**Tech:** Terraform, Docker, Bash scripts

---

## Tech Stack

Ally runs on a modern, polyglot stack focused on reliability, security, and speed of iteration. We use TypeScript-first frontends (React, Next.js, Vite), Python AI services (FastAPI, LangChain), and a NestJS backend, all orchestrated through Docker Compose for consistent dev-to-prod environments.

**[Explore our full tech stack →](/tech-stack.html)**

---

## Getting Started

Ready to contribute? Start by cloning our infrastructure repo which sets up all Ally repositories with a single command. We have opportunities for frontend, mobile, backend, AI/ML, and DevOps contributors.

**[Get started contributing →](/get-started.html)**

## How We Track Issues

We use GitHub Issues across all our repositories to track bugs, feature requests, and improvements. Here's how it works:

- **Repository-specific issues**: Each repo has its own issue tracker for problems specific to that codebase
- **Labels**: We use labels like `bug`, `enhancement`, `good first issue`, `help wanted`, etc.
- **Good First Issues**: Look for the `good first issue` label to find beginner-friendly tasks
- **Milestones**: We organize work into milestones for better project planning

**Finding issues to work on:**
1. Browse issues in the repository you're interested in
2. Look for `good first issue` or `help wanted` labels
3. Check if someone is already assigned
4. Comment on the issue to express interest before starting work

## Code of Conduct

We're committed to providing a welcoming and inclusive environment for everyone. We expect all contributors to:

- Be respectful and considerate
- Welcome newcomers and help them get started
- Focus on what's best for the community
- Show empathy toward other community members

## Get Help

Need assistance or have questions?

- **Documentation**: Each repository has its own README with detailed setup instructions
- **Issues**: Ask questions by creating an issue in the relevant repository
- **Contributing Guide**: Check our [Contributing Guide](./CONTRIBUTING.md) for detailed information

## License

All Ally projects are open source and released under the **MIT License**. This means you're free to use, modify, and distribute the code, as long as you include the original copyright notice.

---

## Thank You!

We truly appreciate your interest in contributing to Ally. Whether you're fixing a typo, adding a feature, or just exploring the code, your involvement helps us build better tools for mental health professionals and the people they serve.

Together, we're making mental health support more accessible and effective. Thank you for being part of this mission.

**Let's build something amazing together! 🌟**

---

*This site is maintained by the Ally development team. Last updated: December 2025*
