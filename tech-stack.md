---
layout: default
title: Tech Stack
---

# Tech Stack

Ally runs on a modern, polyglot stack focused on reliability, security, and speed of iteration. Explore the key components below.

## Frontend {#frontend-stack}
- React 18, Next.js 14, and Vite 5 power our web surfaces
- TypeScript everywhere with Tailwind CSS and Redux Toolkit for stateful experiences
- Shared component systems tuned to match the helloally.ai design language

## Mobile {#mobile-stack}
- React Native 0.79 with TypeScript for iOS and Android parity
- LiveKit-native modules for secure audio streaming and transcription
- Secure storage, push notifications, and accessibility tooling built into the app shell

## Backend & APIs {#backend-stack}
- NestJS (TypeScript) for REST, WebSocket, and event-driven services
- PostgreSQL, Redis, and Socket.IO delivering transactional and realtime workloads
- AWS SQS, S3, and SES integrations orchestrated through domain-driven service modules

## AI Services {#ai-stack}
- Python 3.12 services orchestrated with FastAPI
- LangChain pipelines across OpenAI and proprietary models for counseling insights
- Weaviate vector search and Deepgram speech intelligence underpinning semantic analysis

## Data & Realtime {#data-stack}
- LiveKit for low-latency voice and presence channels
- Redis streams and PostgreSQL analytics layers for longitudinal session data
- Event pipelines translating transcripts into searchable, privacy-aware knowledge

## Infrastructure & DevOps {#infra-stack}
- Terraform-managed cloud environments and shared modules across regions
- Public GitHub Actions workflows covering lint, test, security, build, and image publish steps
- GitOps deployment flows coordinating multi-service releases with automated approvals

## Progressive Docker Compose Rollout {#docker-compose}
We are migrating each service family onto curated Docker Compose bundles so local and production environments stay in lockstep. New repositories launch with Compose files that mirror production images and networking, while legacy stacks are being refactored in phases—starting with infrastructure utilities and backend APIs, followed by AI workloads and frontends. This progressive rollout keeps onboarding friction low (`docker compose up` per bundle), lets teams adopt Compose incrementally, and gives CI/CD pipelines the same topology developers run on their laptops.

---

**Ready to contribute?** Check out our [Get Started](/get-started.html) guide to set up your development environment.
