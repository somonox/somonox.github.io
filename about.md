---
layout: default
title: About
permalink: /about
---

<section class="about-page" markdown="1">
  <div class="about-header" markdown="1">
    <h1 class="about-title">About Me</h1>
    <p class="about-subtitle">{{ site.author.bio | default: "Security Researcher & Developer" }}</p>
  </div>

  <div class="about-content" markdown="1">

### 👋 Hi, I'm {{ site.author.name | default: "somonox" }}

보안 연구와 시스템 해킹에 관심이 많은 학생입니다.  
이 블로그에서는 CTF 풀이, 보안 연구, 개발 관련 글을 공유합니다.

---

### 🔬 Interests
- **Binary Exploitation** — Pwn, ROP, Heap
- **Reverse Engineering** — x86/x64, Ruis binary
- **Tool Development** — Python, Rust, C++

---

### 🔗 Links 
- **GitHub**: [{{ site.author.github | default: "somonox" }}](https://github.com/{{ site.author.github | default: "somonox" }})

---

### 📬 Contact
Discord: a6a6_

  </div>
</section>
