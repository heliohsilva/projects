# DevGOnic

---

## Overview

DevGOnic is a lightweight personal blog platform built from scratch using Node.js and vanilla JavaScript. The goal of the project is to provide a simple and maintainable way to publish technical articles while keeping full control over the architecture and tooling.

Instead of relying on large static site generators, DevGOnic uses a minimal custom engine that converts Markdown articles into static HTML pages and deploys them as a static website.

The blog is available at:
[devgonic.com](https://devgonic.com)

---

## Problem

I wanted a personal blog to publish technical content and store tutorials that I may need to revisit in the future. Initially, I considered using existing static site generators such as [Hugo](https://gohugo.io/).

However, most existing solutions include many features that I did not need and introduce additional complexity. My goal was to build something minimal, fully customizable, and as easy as possible to maintain.

---

## Solution

To solve this problem, I built a custom static blog engine tailored specifically to my needs.

The engine focuses on simplicity and maintainability. It implements only the necessary features required to build and publish the blog. Because the system is intentionally small, it is easy to extend with new features if needed.

The build process reads Markdown articles, converts them into HTML, injects them into predefined layouts, and generates the final static pages.

---

## Architecture

The project follows a simple architecture composed of three main parts:

1. Engine Source Code
2. Static Pages
3. Raw Articles

### Engine Source Code

The engine contains the logic responsible for building the website. It includes:

- Layouts: HTML templates used as the base structure for pages.
- Modules: Reusable components such as the navbar, footer, and head section.
- Builder: A Node.js script that assembles the pages.

During the build process, the builder:

1. Loads the layouts.
2. Injects reusable modules.
3. Reads Markdown articles.
4. Converts Markdown to HTML.
5. Generates the final static pages.

### Static Pages

After the build process completes, the final HTML pages are generated in the static pages directory. These files are then deployed as a static website using GitHub Pages.

### Raw Articles

All articles are written in Markdown and stored in the articles directory. During the build step, these files are parsed and transformed into HTML pages.

---

## Project Structure

```
.
├── articles
├── src
│   ├── engine
│   │   ├── layouts
│   │   ├── modules
│   │   └── build.js
│   └── pages
│   │   ├── articles
│   │   ├── assets
│   │   │   ├── fonts
│   │   │   ├── icons
│   │   │   ├── images
│   │   │   ├── script
│   │   │   └── style
│   └── misc
└── test

```

---

## Technologies Used

The project was implemented using a minimal technology stack:

- Node.js – used to implement the build engine
- Vanilla JavaScript – used for client-side interactions and animations
- HTML & CSS – used to build layouts and components

---

## Technical Decisions

Several design decisions were made to keep the system simple and maintainable:

- Avoid using large frameworks or blog engines.
- Implement a custom static site generator to have full control over the build process.
- Use a modular HTML structure to allow reusable components.
- Keep the runtime as simple as possible by generating fully static pages.

This approach results in a lightweight system that is easy to maintain and extend.

---

## Technical Challenges

One of the main challenges was implementing the search functionality.

Since the project does not rely on external services or backend APIs, the search engine had to work entirely on static content. Designing an efficient way to index and search articles required experimenting with different approaches before arriving at a working solution.

---

## Results

The final result is a lightweight and easy-to-deploy personal blog platform.

The system allows me to write articles in Markdown, run a simple build process, and generate a complete static website ready for deployment.

Because the engine is small and purpose-built, it is easy to understand, maintain, and extend when new features are required.

---

## Key Learnings

Through this project I gained practical experience with:

- Designing a custom static site generator
- Implementing a content pipeline from Markdown to HTML
- Structuring a maintainable frontend project without frameworks
- Deploying static websites using GitHub Pages

---
