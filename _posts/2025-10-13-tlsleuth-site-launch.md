---
layout: post
title: "TLSleuth Website Launch"
date: 2025-10-13
tags: [announcement, github-pages, jekyll, project, tlsleuth]
---

# 🎉 TLSleuth Website Launch – Built with Jekyll & GitHub Pages

After weeks of experimenting, tweaking, and breaking things just enough to learn from them — the **TLSleuth** website is now live at [**TLSleuth.com**](https://TLSleuth.com)! 🚀

This marks a big step for the project — moving beyond the PowerShell Gallery and GitHub README into a clean, documented home that’s easy to explore and update.

---

## 🕵️ What TLSleuth Is About

**TLSleuth** is a PowerShell module for quickly inspecting TLS/SSL endpoints and certificates from the shell.
It’s meant for operators, engineers, and scripters who need to know *what’s really happening during a handshake* — without firing up a full TLS scanner.

Now, with its own website, the module finally has:

- A polished **home page** introducing the project
- Full **docs** split into sections from the README
- A **blog** for release notes and technical deep dives
- Clean design using the **Cayman Jekyll theme**
- And, of course, the classic tagline:
  > **“TLS, under the magnifying glass.”**

---

## 🧠 What We Learned Along the Way

Standing up a Jekyll site on GitHub Pages sounds simple — but doing it *well* taught us a few things.

### 1️⃣ Theming with Remote Themes
Using `remote_theme: pages-themes/cayman@v0.2.0` was easy, but making it our own meant learning how to:
- Override layouts and includes without breaking the theme.
- Extend styles through a custom SCSS file (`assets/css/tlsleuth.scss`).
- Rebuild headers using **CSS Grid** for true visual centering (logo + text + menu alignment).

### 2️⃣ Custom Headers and Brand Identity
We designed two headers:
- **Hero Header** — for the homepage, featuring the logo on the left and tagline in the center.
- **Small Header** — for subpages, compact with the logo and title left-aligned, buttons right-aligned.

Building those taught us some great CSS lessons:
- How to use `grid-template-columns: 1fr auto 1fr` to keep text perfectly centered even with side content.
- When to reach for `flexbox` vs `grid` for layout control.
- How to gracefully adapt to mobile with media queries.

### 3️⃣ SCSS and Theme Variables
By importing the Cayman theme’s `_sass/variables.scss`, we could reuse its color palette and spacing directly in our own styles.
That gave TLSleuth its own flavor without clashing with the theme’s look and feel.

### 4️⃣ Structuring Docs from the README
We learned how to:
- Turn a long PowerShell README into a structured documentation site using Jekyll **collections** (`_docs`).
- Create a consistent navigation experience by reusing includes.
- Keep the README as the single source of truth while still maintaining dedicated pages for key topics.

### 5️⃣ Embracing Simplicity
The biggest lesson: **GitHub Pages and Jekyll don’t need to be complex.**
A few small tweaks — good layouts, clean SCSS, and consistent partials — go a long way toward making a project site look professional and lightweight.

---

## 🧩 What’s Next

Now that the site is live, we’ll start posting:
- **Tips & tricks** for TLS debugging in PowerShell.
- **Behind-the-scenes posts** on TLSleuth’s internals.
- **Comparisons** to dedicated TLS scanners like `sslyze`, `sslscan`, and `testssl.sh`.
- And maybe even a few **DevOps workflow articles** about automation, testing, and packaging.

---

## 🙌 Thanks

Big thanks to everyone who’s used, starred, or tested TLSleuth so far — and to the open-source community for making tools like **Jekyll**, **GitHub Pages**, and **PowerShell** so powerful and accessible.

If you want to explore or contribute:
- 🔗 [Visit the site](https://TLSleuth.com)
- 💾 [View the code on GitHub](https://github.com/MadCrabCyder/TLSleuth)
- 🧑‍💻 [Install from PowerShell Gallery](https://www.powershellgallery.com/packages/TLSleuth)

We’ll also be expanding the documentation.
Until now, the GitHub README has served as the main reference. The next step is to:

- Turn that single long file into a structured documentation set using Jekyll **collections** (e.g., `_docs`).
- Add navigation links and sidebars for quick reference.
- Keep the README as the single source of truth — but automatically sync it into the docs site using GitHub Actions.

This will make it easier to browse command usage, parameters, examples, and developer notes directly on **TLSleuth.com**, while keeping GitHub and PowerShell Gallery aligned.


---

*Built with ❤️ on Jekyll, powered by GitHub Pages, and debugged far too many times at 1 a.m.*