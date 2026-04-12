# URL-Shortner

---

## Overview

URL Shortener is a backend service designed to transform long URLs into short, shareable links. The system focuses on performance, scalability, and simplicity, providing a reliable way to redirect users efficiently.

The API is built using Go and follows a clean architecture approach to ensure maintainability and extensibility.

---

## Problem

Long URLs are often inconvenient to share, store, and manage. Additionally, systems that frequently resolve the same URLs can suffer from unnecessary database load, impacting performance and scalability.

The challenge was to design a system capable of:

- Generating compact and unique short URLs
- Handling high-frequency redirections efficiently
- Minimizing database access latency

---

## Solution

To address these issues, I implemented a URL shortening service with a caching layer.

The system generates short identifiers using a Base62 encoding strategy, ensuring compact and URL-friendly results. To improve performance, a Redis cache layer is introduced between the API and the database, significantly reducing the number of direct database queries for frequently accessed URLs.

When a request is made:

- The system checks if the URL mapping exists in cache
- If found, it immediately returns the original URL
- Otherwise, it queries the database and updates the cache

This approach ensures fast response times, especially for frequently accessed links.

---

## Architecture

The project follows a Controller–Service–Repository architecture, where each layer has a single responsibility:

- Controller: Handles HTTP requests and responses
- Service: Contains business logic and orchestration
- Repository: Manages data persistence and retrieval

This separation improves maintainability and allows independent evolution of each layer.

Additionally, the system integrates:

- A MySQL database for persistent storage
- A Redis instance for caching and performance optimization

The project tree is showed bellow:

```
.
├── api
│   ├── config
│   │   └── config.go
│   ├── controllers
│   │   └── urlController.go
│   ├── database
│   │   └── database.go
│   ├── docker-compose.yaml
│   ├── go.mod
│   ├── go.sum
│   ├── helpers
│   │   └── responses.go
│   ├── init-db.sql
│   ├── main.go
│   ├── models
│   │   └── urlModel.go
│   ├── README.md
│   ├── repository
│   │   ├── cacheRepository.go
│   │   └── urlRepository.go
│   └── service
│       └── urlService.go
└── README.md
```
---

## Technologies Used

The project uses a focused and efficient technology stack:

- Go (Golang) – core backend language
- Gin Gonic – HTTP web framework
- MySQL – persistent data storage
- Redis – in-memory cache layer
- Docker – containerized environment setup

---

## Technical Decisions

Several decisions were made to ensure performance and maintainability:

- Adopt a layered architecture (Controller–Service–Repository) to enforce separation of concerns
- Use Base62 encoding to generate short, URL-safe identifiers
- Introduce Redis as a caching layer to reduce database load
- Use Docker Compose to simplify local development and environment setup

These choices result in a system that is both efficient and easy to extend.

---

## Technical Challenges

One of the main challenges was designing an efficient caching strategy.

It was necessary to ensure consistency between Redis and MySQL while avoiding stale data and unnecessary cache misses. Balancing cache usage and database synchronization required careful handling of read/write flows.

Another challenge was generating unique short URLs without collisions while maintaining performance.

---

## Results

The final result is a production-ready API capable of handling URL shortening and redirection efficiently.

Due to the caching layer, frequently accessed URLs are resolved with very low latency, significantly improving overall performance and scalability.

The system is modular, easy to maintain, and ready to be integrated with a frontend application.

---

## Key learnings

Through this project I gained practical experience with:

- Designing scalable backend architectures
- Implementing caching strategies with Redis
- Structuring Go applications using clean architecture principles
- Optimizing API performance and reducing latency

---

Link to project repository: [url-shortener](https://github.com/heliohsilva/url-shortener)