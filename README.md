# Onur Atci's Personal Blog

A Jekyll-based blog showcasing my journey as an Engineering Manager and Senior Software Developer. Built for GitHub Pages deployment.

## About

This blog features:
- Professional background and experience in software development
- Technical insights on microservices, cloud architecture, and engineering management
- Personal projects and interests
- Regular blog posts on technology and leadership

## Local Development

### Prerequisites
- Ruby 3.1+
- Bundler

### Setup
```bash
# Install dependencies
bundle install

# Run the development server
bundle exec jekyll serve

# View the site at http://localhost:4000
```

### Building for Production
```bash
bundle exec jekyll build
```

## Deployment

This site is automatically deployed to GitHub Pages using GitHub Actions. Any push to the `main` branch triggers a new deployment.

## Structure

```
├── _config.yml          # Site configuration
├── _layouts/            # Page layouts
│   └── post.html       # Blog post layout
├── _posts/             # Blog posts
├── assets/
│   └── css/
│       └── main.scss   # Custom styling
├── index.md            # Homepage
├── about.md            # About page
├── blog.md             # Blog index
└── .github/
    └── workflows/
        └── jekyll.yml  # GitHub Pages deployment
```

## Features

- **Responsive Design**: Clean, professional layout optimized for all devices
- **Blog System**: Full Jekyll blog with post categories, tags, and navigation
- **SEO Optimized**: Meta tags, sitemaps, and structured data
- **Fast Loading**: Optimized CSS and minimal dependencies
- **GitHub Pages Ready**: Automated deployment with GitHub Actions

## Customization

### Adding Blog Posts

Create new markdown files in `_posts/` with the naming convention:
```
YYYY-MM-DD-title-of-post.md
```

Include front matter:
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +0300
categories: [category1, category2]
tags: [tag1, tag2, tag3]
author: Onur Atci
---
```

### Modifying Styles

Edit `assets/css/main.scss` to customize the appearance.

## Contact

- **Email**: onuratci@icloud.com
- **LinkedIn**: [linkedin.com/in/onuratci](https://linkedin.com/in/onuratci)

## License

This project is open source and available under the [MIT License](LICENSE).