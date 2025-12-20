# Mobile System Design Interview Guide - Project Context

## Project Overview

This repository hosts "A Simple Framework For Mobile System Design Interviews," a comprehensive guide designed to help iOS and Android engineers navigate system design interviews. Unlike code repositories, this project is a knowledge base comprised of instructional Markdown documents and diagrams.

The core philosophy is that interviewers look for "Signal"—evidence of a candidate's thought process, communication skills, and ability to handle ambiguity—rather than a perfect, production-ready solution.

## Structure & Key Files

### Core Documentation
*   **`README.md`**: The primary entry point. It outlines the complete interview framework:
    *   **Process:** Introductions -> Task Definition -> High-level Discussion -> Detailed Discussion.
    *   **Strategy:** "Breadth-First Search" (BFS) approach to cover ground before diving deep.
    *   **Technical Topics:** API Design (REST vs GraphQL vs WebSocket), Data Storage (Database vs Key-Value), Caching, QoS, and Offline Support.
*   **`common-interview-mistakes.md`**: A critical guide for both candidates and interviewers, detailing behavioral and technical pitfalls (e.g., "Rushing to solution," "Being silent," "Grilling the candidate").
*   **`BLOGPOSTS.MD`**: A curated list of engineering blog posts from major tech companies (Uber, Facebook, Airbnb, etc.), categorized by topic (Caching, Networking, Architecture).

### Exercises (`/exercises/`)
This directory contains mock interview scenarios. Each file follows the framework defined in the README.

**Solved Examples:**
*   **`exercises/chat-app.md`**: Design a WhatsApp-like application. Covers real-time communication (WebSockets vs Polling), local storage, and media handling.
*   **`exercises/caching-library.md`**: Design a general-purpose LRU caching library. Covers memory vs disk cache, eviction policies, and thread safety.
*   **`exercises/image-library.md`**: Design an image loading library. Covers request management, memory caching, and view lifecycle integration.
*   **`exercises/file-downloader-library.md`**: Design a file downloader. Covers concurrency, background vs foreground execution, and job dispatching.

**Unsolved/Prompted Topics:**
The `exercises/README.md` lists many other common prompts (e.g., "Design Instagram Feed", "Design Location Sharing") that serve as practice prompts without provided solutions.

### Specialized Topics (`/topics/`)
Contains in-depth articles on specific architectural or design topics:
*   **`topics/api-design-principles.md`**: Best practices for robust RESTful API design tailored for mobile constraints.
*   **`topics/mobile-pagination-strategies.md`**: Detailed guide on pagination strategies (Cursor vs Offset), mistakes to avoid, and security considerations.
*   **`topics/image-loading-strategies.md`**: Best practices for asynchronous image loading, memory caching, and view recycling.

### Assets (`/images/`)
Contains SVG diagrams used throughout the documentation to illustrate high-level architectures (e.g., `twitter-feed-high-level-diagram.svg`, `resumable-uploads.svg`). These are crucial for understanding the "High-Level Diagram" step of the framework.

## Key Concepts for Context

When interacting with this repository, be aware of these specific definitions:
*   **"The Signal"**: The positive indicators a candidate provides to an interviewer (e.g., clear assumptions, trade-off analysis).
*   **Functional vs. Non-Functional Requirements**: The framework explicitly separates "what it does" from "how it performs" (latency, offline support, battery usage).
*   **Deep Dive**: A specific phase in the interview where the candidate focuses on one component (e.g., "Tweet Feed Flow") to demonstrate specialized knowledge.
*   **Client-Side Focus**: Unlike general system design, this repo prioritizes mobile-specific concerns: Battery life, Network bandwidth, App lifecycle, and Local persistence.

## Usage Guide

1.  **Start with `README.md`**: Internalize the 45-60 minute interview structure and the "BFS" approach.
2.  **Review `common-interview-mistakes.md`**: Identify behaviors to avoid.
3.  **Study Solved Exercises**: Read `exercises/chat-app.md` or `exercises/caching-library.md` to see the framework applied in practice. Note how the "candidate" drives the conversation.
4.  **Practice**: Pick a topic from `exercises/README.md` and attempt to produce a design document following the standard structure (Requirements -> High Level Diagram -> Deep Dive).