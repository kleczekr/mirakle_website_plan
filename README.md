# ✨ Mirakle

**IT Services & Creative Storytelling Platform**

Mirakle is a Ruby on Rails-powered platform showcasing IT services, film editing
work, and data analytics - with a focus on building useful tools for creatives
and developers.

---

## 🎯 What is Mirakle?

A personal portfolio and services platform featuring:

- **📝 Git-based Blog** - Write in terminal, push to publish (Markdown + YAML
  front matter)
- **🎬 Film Portfolio** - Video work and tutorials via Vimeo integration
- **📊 Shopify App Store Analytics** - Deep data insights with daily scraping,
  review tracking, and D3.js visualizations
- **🎙️ Audio/Video Transcription Service** - AI-powered speech-to-text with
  translation (coming soon)
- **🎥 Shot Planning Tool** - Visual flowchart app for cinematographers (coming
  soon)

---

## 🚀 Current Status: Phase 1 - MVP

### ✅ Completed

- [x] Rails 8 setup with PostgreSQL & Redis
- [x] Git-based blog system (Markdown with front matter)
- [x] Blog post rendering with Redcarpet
- [x] Custom shortcode processing for embedded content
- [x] Basic routing and views
- [x] Liquid templating integration
- [x] Redis caching for blog posts

### 🚧 In Progress

- [ ] Polaris CSS styling
- [ ] Vimeo API integration for portfolio
- [ ] Blog index/show page polish

### 📋 Next Up (Phase 2)

- [ ] Shopify App Store scraper (Rails-native with Sidekiq)
- [ ] PostgreSQL models for apps, reviews, and review changes
- [ ] Change detection system (track edits, deletions, replies)
- [ ] Public analytics dashboard with D3.js visualizations
- [ ] Interactive data exploration

---

## 🎬 Future Projects

### 1. AV2Text - Transcription & Translation Service

**Status:** Proof of concept complete in Python
([av2text repo](https://github.com/kleczekr/av2text))

A web service for filmmakers and content creators to transcribe and translate
audio/video files using OpenAI's Whisper API.

**Features:**

- Upload audio/video files
- Automatic compression and chunking
- Speech-to-text transcription
- Optional translation to target language
- Annotation support for timestamped notes
- BYOK (Bring Your Own Key) - users provide their OpenAI API key for free usage
- Web-based interface (migrate from Python CLI to Rails web app)

**Technical Approach:**

- Option A: Keep Python microservice, integrate with Rails via API
- Option B: Rewrite in Ruby using `ruby-openai` gem + `ffmpeg` for media
  processing
- File upload handling (temporary storage during processing)
- Background job processing with Sidekiq
- WebSocket for real-time progress updates

**Use Cases:**

- Film editors needing quick transcripts
- Content creators localizing videos
- Documentary filmmakers organizing interview footage
- Accessibility (generating subtitles)

---

### 2. Shot Planner - Visual Cinematography Tool

**Status:** Concept phase (requested by cinematographer collaborator)

An interactive flowchart/planning tool for cinematographers to organize shot
lists, group by location/setup, and plan shooting schedules.

**Core Concept:** Visual node-based interface where each node represents a shot,
connected by relationships (same location, similar angle, chronological
sequence, etc.)

**Features:**

- **Shot Nodes:**
  - Shot number/name
  - Scene reference
  - Camera angle/movement notes
  - Lens/focal length
  - Lighting setup notes
  - Reference images upload
- **Visual Grouping:**
  - Cluster shots by location
  - Group by camera setup
  - Timeline view (chronological in film vs. shooting order)
- **Collaboration:**
  - Share shot plans with crew
  - Comment threads on individual shots
  - Status tracking (planned → shot → edited)
- **Export:**
  - PDF shot list
  - Shooting schedule
  - Location-based call sheets

**Technical Stack:**

- D3.js for interactive node graph visualization
- Drag-and-drop interface for rearranging shots
- Image storage solution (TBD - separate repo, Cloudflare R2, or local droplet
  storage)
- Real-time collaboration (ActionCable WebSockets)
- PostgreSQL for shot/project data
- Canvas API for visual annotations on reference images

**UI/UX Considerations:**

- Desktop-first (cinematographers work on laptops)
- Touch-friendly for tablet use on set
- Keyboard shortcuts for power users
- Print-friendly views
- Dark mode (for low-light environments)

---

## 🛠️ Tech Stack

### Core

- **Ruby on Rails 8.1** - Backend framework
- **PostgreSQL** - Primary database
- **Redis** - Caching & Sidekiq backend
- **Liquid** - Templating engine
- **Tailwind CSS** - Styling (with Polaris design tokens)

### Frontend

- **D3.js** - Data visualizations
- **Vanilla JavaScript** - Progressive enhancement
- **Stimulus** - Minimal JavaScript framework

### Background Jobs & Scraping

- **Sidekiq** - Background job processing
- **sidekiq-cron** - Scheduled jobs (daily scraping)
- **Mechanize + Nokogiri** - Web scraping
- **Faraday** - HTTP requests

### APIs & Integrations

- **Vimeo API** - Video portfolio
- **OpenAI API** - Transcription service (future)
- **Anthropic API** - AI content generation (Phase 4)

### Deployment

- **Capistrano** - Deployment automation
- **Digital Ocean Droplet** - Ubuntu server
- **Nginx** - Reverse proxy & static file serving
- **Puma** - Rails web server

---

## 📂 Project Structure

```
mirakle/
├── app/                    # Rails application code
├── blog_posts/             # Markdown blog posts (Git-based CMS)
│   ├── 2025/
│   │   ├── 01-january/
│   │   ├── 02-february/
│   │   └── ...
│   ├── _drafts/           # Unpublished posts
│   └── _private/          # Admin-only posts
├── config/                # Rails configuration
├── db/                    # Database migrations & schema
└── ...

mirakle-images/            # Separate Git repository
├── blog/                  # Blog post images
├── portfolio/             # Film work screenshots
└── shot-planner/          # Reference images for shot planning
```

---

## 🗺️ Roadmap

### Phase 1: MVP (Current - 2-3 weeks)

- Git-based blogging system
- Vimeo video portfolio
- Basic site structure with Polaris styling

### Phase 2: Shopify Analytics (4-6 weeks)

- Daily scraper with change detection
- Public analytics dashboard
- D3.js visualizations
- Historical trend tracking

### Phase 3: Authentication & Premium Features (3-4 weeks)

- Google OAuth login
- Tiered access ~~(free/premium)~~ (non-logged-in/logged-in)
- Newsletter integration
- ~~Premium~~ Logged-in analytics features

### Phase 4: AI & Admin Tools (2-3 weeks)

- Anthropic API integration
- Social media post generator
- Admin dashboard
- Audit logging

### Phase 5: AV2Text Service (TBD)

- Migrate Python proof-of-concept to Rails
- Web-based file upload interface
- Background transcription processing
- BYOK (Bring Your Own Key) system

### Phase 6: Shot Planner Tool (TBD)

- D3.js node-based interface
- Shot management system
- Image upload & annotation
- Collaboration features
- Export functionality

---

## 🚦 Getting Started

### Prerequisites

- Ruby 3.3+
- PostgreSQL 15+
- Redis 7+

### Installation

```bash
# Clone the repos
git clone https://github.com/kleczekr/mirakle.git
git clone https://github.com/kleczekr/mirakle-images.git

cd mirakle

# Install dependencies
bundle install

# Setup database
rails db:create
rails db:migrate

# Start Redis (if not running)
redis-server

# Start Rails server
rails server

# Visit http://localhost:3000
```

### Writing Blog Posts

```bash
# Create a new post
vim blog_posts/2025/10-october/2025-10-22-my-post.md

# Add front matter + content
---
title: "My Post Title"
slug: my-post-title
published_at: 2025-10-22T12:00:00Z
status: published
tags: [ruby, rails]
excerpt: "A brief description"
---

# Your content here...

# Commit and push
git add blog_posts/
git commit -m "New post: My Post Title"
git push

# Deploy (once Capistrano is set up)
cap production deploy
```

---

## 🎨 Design Philosophy

- **Developer happiness** - Optimize for workflow, not theoretical scale
- **Build in public** - Document the journey
- **Practical tools** - Solve real problems for creatives and developers
- **Progressive enhancement** - Start simple, add complexity when needed
- **Git-based content** - Version control everything

---

## 📚 Learning Resources

This project is a learning journey into Ruby on Rails. Key resources:

- **"Agile Web Development with Rails 7"** by Sam Ruby
- **"The Well-Grounded Rubyist"** by David A. Black
- **"Eloquent Ruby"** by Russ Olsen
- [Rails Guides](https://guides.rubyonrails.org)
- [GoRails Screencasts](https://gorails.com)

---

## 📝 License

MIT License - feel free to fork and adapt for your own projects!

---

## 🤝 Contributing

This is primarily a personal learning project, but suggestions and ideas are
welcome! Open an issue or reach out.

---

**Built in Arch Linux + Neovim (it works on my system)**
