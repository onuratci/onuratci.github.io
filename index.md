---
layout: default
title: Home
---

<div class="hero">
  <h1>Onur Atci</h1>
  <p class="hero-subtitle">Engineering Manager & Senior Software Developer</p>
  <p class="hero-description">15+ years of experience leading teams and building scalable, cloud-native solutions</p>
  
  <div class="hero-links">
    <a href="/about" class="btn btn-primary">About Me</a>
    <a href="/blog" class="btn btn-secondary">Blog</a>
  </div>
</div>

<div class="section">
  <h2>Current Role</h2>
  <div class="current-role">
    <h3>Engineering Manager at Getir</h3>
    <p class="role-period">June 2023 - Present</p>
    <p>Currently working in the Advertising Tech team at Getir, developing a B2B ad management platform for in-app advertising. Leading teams to build and optimize ad server solutions that enhance campaign performance and user engagement.</p>
  </div>
</div>

<div class="section">
  <h2>Core Expertise</h2>
  <div class="skills-grid">
    <div class="skill-category">
      <h4>Languages & Frameworks</h4>
      <ul>
        <li>Java, Golang, Kotlin</li>
        <li>Spring Boot, Android SDK</li>
        <li>REST, SOAP, JPA</li>
      </ul>
    </div>
    <div class="skill-category">
      <h4>Cloud & Infrastructure</h4>
      <ul>
        <li>AWS (Kinesis, EC2, RDS, S3)</li>
        <li>Docker, Kubernetes</li>
        <li>Serverless Architecture</li>
      </ul>
    </div>
    <div class="skill-category">
      <h4>Databases & Search</h4>
      <ul>
        <li>PostgreSQL, MongoDB</li>
        <li>ElasticSearch, Redis</li>
        <li>Aurora, OpenSearch</li>
      </ul>
    </div>
  </div>
</div>

<div class="section">
  <h2>Recent Blog Posts</h2>
  <div class="recent-posts">
    {% for post in site.posts limit:3 %}
      <article class="post-preview">
        <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
        <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
        <p>{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
      </article>
    {% endfor %}
  </div>
  <p><a href="/blog" class="view-all">View all posts →</a></p>
</div>