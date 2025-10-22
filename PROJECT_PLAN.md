# Shopify Insights Platform - Project Plan

## Project Overview

A personal IT and creative services website built with Ruby on Rails, showcasing
video portfolio, blog, and comprehensive Shopify App Store analytics with daily
scraping and change tracking.

---

## Tech Stack

### Core Framework

- **Backend**: Ruby on Rails 7+
- **Templating**: Liquid (via `liquid` gem)
- **Styling**: Shopify Polaris CSS/design tokens (no React components)
- **Database**: PostgreSQL (on same droplet)
- **Cache/Jobs**: Redis + Sidekiq
- **Deployment**: Capistrano to Digital Ocean Ubuntu droplet

### Key Gems & Libraries

- **Authentication**: Devise + OmniAuth (Google OAuth)
- **Authorization**: Pundit or CanCanCan
- **Markdown Processing**: Redcarpet or Kramdown
- **Background Jobs**: Sidekiq + sidekiq-cron
- **Web Scraping**: Mechanize or Faraday + Nokogiri
- **API Integration**:
  - Vimeo API (video portfolio)
  - Anthropic API (AI content generation)
  - OpenAI API (transcription service - Phase 5)
  - Newsletter service (Mailchimp/SendGrid/ConvertKit)

### Frontend

- Polaris CSS for aesthetic
- Vanilla JavaScript for progressive enhancement
- **D3.js** for data visualization (via CDN)

---

## Infrastructure

### Hosting Architecture

```
Digital Ocean Ubuntu Droplet (Single Server)
├── Nginx (reverse proxy, :80/443)
├── Rails/Puma (:3000)
├── PostgreSQL (localhost)
├── Redis (Sidekiq backend)
└── Sidekiq Workers (background jobs)
```

### Recommended Droplet Size

- **Minimum**: $12/month (2GB RAM)
- **Recommended**: $24/month (4GB RAM)
- **Ideal**: $48/month (8GB RAM)

### SSL

- Let's Encrypt via Certbot

---

## Data Models

### Blog & Content

#### BlogPost

```ruby
create_table :blog_posts do |t|
  t.string :title
  t.string :slug, index: { unique: true }
  t.text :content # ActionText rich text
  t.string :status # draft/published/private
  t.datetime :published_at
  t.references :author # for future
  t.timestamps
end
```

#### VimeoVideo

```ruby
create_table :vimeo_videos do |t|
  t.string :vimeo_id
  t.string :title
  t.text :description
  t.string :thumbnail_url
  t.integer :duration
  t.string :category # tutorial/creative
  t.text :embed_html
  t.integer :view_count
  t.timestamps
end
```

---

### Shopify App Store Data

#### ShopifyApp

```ruby
create_table :shopify_apps do |t|
  t.string :app_id, null: false, index: { unique: true }
  t.string :name
  t.string :developer
  t.string :category
  t.string :pricing_model
  t.decimal :rating, precision: 3, scale: 2
  t.integer :review_count
  t.string :install_count_range
  t.text :description
  t.jsonb :features, default: []
  t.boolean :is_active, default: true
  t.datetime :last_scraped_at
  t.datetime :first_seen_at
  t.datetime :last_seen_at
  t.timestamps
end

# Indexes
add_index :shopify_apps, :category
add_index :shopify_apps, :rating
add_index :shopify_apps, [:rating, :review_count]
```

#### ShopifyReview (with deletion tracking)

```ruby
create_table :shopify_reviews do |t|
  t.string :review_id, null: false, index: { unique: true }
  t.references :app, null: false, foreign_key: { to_table: :shopify_apps }
  t.string :author_name
  t.integer :rating
  t.text :review_text
  t.integer :helpful_count
  t.datetime :posted_at
  t.string :merchant_size

  # Tracking fields
  t.boolean :is_deleted, default: false
  t.datetime :deleted_at
  t.datetime :last_scraped_at
  t.datetime :first_seen_at

  t.timestamps
end

add_index :shopify_reviews, :app_id
add_index :shopify_reviews, :posted_at
add_index :shopify_reviews, :rating
add_index :shopify_reviews, :is_deleted
```

#### ShopifyReviewChange (change detection!)

```ruby
create_table :shopify_review_changes do |t|
  t.references :review, null: false, foreign_key: { to_table: :shopify_reviews }
  t.string :change_type # 'text_edited', 'rating_changed', 'reply_added', 'deleted'
  t.jsonb :old_value
  t.jsonb :new_value
  t.datetime :detected_at
  t.timestamps
end

add_index :shopify_review_changes, :change_type
add_index :shopify_review_changes, :detected_at
```

#### ShopifyReviewReply

```ruby
create_table :shopify_review_replies do |t|
  t.string :reply_id, index: { unique: true }
  t.references :review, null: false, foreign_key: { to_table: :shopify_reviews }
  t.string :developer_name
  t.text :reply_text
  t.datetime :posted_at
  t.boolean :is_deleted, default: false
  t.datetime :deleted_at
  t.datetime :last_scraped_at
  t.timestamps
end
```

#### AppStatSnapshot (historical trends)

```ruby
create_table :app_stat_snapshots do |t|
  t.references :app, null: false, foreign_key: { to_table: :shopify_apps }
  t.decimal :rating, precision: 3, scale: 2
  t.integer :review_count
  t.string :install_count_range
  t.date :snapshot_date, null: false
  t.timestamps
end

add_index :app_stat_snapshots, [:app_id, :snapshot_date], unique: true
add_index :app_stat_snapshots, :snapshot_date
```

---

### Authentication & Users

#### User

```ruby
create_table :users do |t|
  t.string :email, null: false, index: { unique: true }
  t.string :encrypted_password
  t.string :name
  t.string :google_uid
  t.string :google_avatar
  t.string :role # free/premium/admin
  t.boolean :newsletter_subscribed
  t.string :newsletter_token
  t.datetime :last_sign_in_at
  t.timestamps
end
```

#### NewsletterSubscriber

```ruby
create_table :newsletter_subscribers do |t|
  t.string :email, null: false, index: { unique: true }
  t.string :name
  t.datetime :subscribed_at
  t.datetime :unsubscribed_at
  t.string :token # for unsubscribe link
  t.timestamps
end
```

#### UserAppBookmark (premium feature)

```ruby
create_table :user_app_bookmarks do |t|
  t.references :user, null: false
  t.references :app, null: false, foreign_key: { to_table: :shopify_apps }
  t.text :notes
  t.timestamps
end
```

---

### AI & Admin Tools

#### SocialMediaPost

```ruby
create_table :social_media_posts do |t|
  t.references :blog_post
  t.string :platform # facebook/linkedin/twitter/instagram/tiktok
  t.text :generated_content
  t.jsonb :image_suggestions
  t.jsonb :hashtags
  t.string :status # draft/approved/posted
  t.datetime :posted_at
  t.string :ai_model_version
  t.text :generation_prompt # for debugging
  t.timestamps
end
```

#### AdminActivity (audit log)

```ruby
create_table :admin_activities do |t|
  t.references :admin, foreign_key: { to_table: :users }
  t.string :action
  t.string :resource_type
  t.string :resource_id
  t.jsonb :changes
  t.timestamps
end
```

---

### Future: AV2Text Service (Phase 5)

#### TranscriptionJob

```ruby
create_table :transcription_jobs do |t|
  t.references :user
  t.string :file_name
  t.string :file_path # temporary storage
  t.string :status # pending/processing/completed/failed
  t.text :transcript_text
  t.string :source_language
  t.string :target_language # for translation
  t.integer :duration_seconds
  t.integer :file_size_bytes
  t.string :openai_api_key_hash # user's BYOK
  t.decimal :processing_cost, precision: 10, scale: 4
  t.jsonb :metadata # file type, compression settings, etc.
  t.text :error_message
  t.timestamps
end

add_index :transcription_jobs, :user_id
add_index :transcription_jobs, :status
add_index :transcription_jobs, :created_at
```

#### TranscriptionAnnotation

```ruby
create_table :transcription_annotations do |t|
  t.references :transcription_job, null: false
  t.integer :timestamp_seconds
  t.text :note
  t.string :annotation_type # comment/marker/correction
  t.timestamps
end

add_index :transcription_annotations, :transcription_job_id
```

---

### Future: Shot Planner Tool (Phase 6)

#### Project

```ruby
create_table :projects do |t|
  t.references :user, null: false
  t.string :name
  t.text :description
  t.string :status # planning/in_progress/completed
  t.date :shoot_date
  t.string :location
  t.jsonb :collaborators # array of user IDs
  t.timestamps
end

add_index :projects, :user_id
add_index :projects, :status
```

#### Shot

```ruby
create_table :shots do |t|
  t.references :project, null: false
  t.string :shot_number
  t.string :scene_reference
  t.text :description
  t.string :camera_angle
  t.string :camera_movement
  t.string :lens
  t.string :focal_length
  t.text :lighting_notes
  t.string :location_name
  t.integer :setup_number
  t.integer :sequence_order # chronological in film
  t.integer :shooting_order # actual shooting order
  t.string :status # planned/shot/edited
  t.jsonb :reference_images # array of image URLs
  t.decimal :position_x # for node graph
  t.decimal :position_y # for node graph
  t.timestamps
end

add_index :shots, :project_id
add_index :shots, :status
add_index :shots, :location_name
add_index :shots, :setup_number
```

#### ShotConnection

```ruby
create_table :shot_connections do |t|
  t.references :from_shot, null: false, foreign_key: { to_table: :shots }
  t.references :to_shot, null: false, foreign_key: { to_table: :shots }
  t.string :connection_type # same_location/similar_angle/chronological/camera_setup
  t.timestamps
end

add_index :shot_connections, [:from_shot_id, :to_shot_id], unique: true
```

#### ShotComment

```ruby
create_table :shot_comments do |t|
  t.references :shot, null: false
  t.references :user, null: false
  t.text :comment_text
  t.boolean :resolved, default: false
  t.timestamps
end

add_index :shot_comments, :shot_id
add_index :shot_comments, [:shot_id, :resolved]
```

---

## Scraping Architecture

### Job Structure

- **Daily Cron**: Triggers master scraping job
- **Master Job**: Queues individual app scrapes
- **App Detail Job**: Scrapes app data, queues review scrape
- **Review Job**: Scrapes reviews, detects changes/deletions

### Change Detection Logic

- Compare current scrape with stored data
- Track: text edits, rating changes, deletions, new replies
- Store changes in `shopify_review_changes` table
- Mark reviews as `is_deleted: true` if no longer found

### Rate Limiting

- Random 1-3 second delays between requests
- Respect robots.txt
- Use rotating user agents if needed

### Data Volume Estimates

- ~10,000 apps
- ~1,000,000 reviews (100 avg per app)
- ~200,000 replies (20% of reviews)

---

## Phased Development Timeline

### Phase 1: MVP (2-3 weeks)

**Goal**: Basic site + portfolio + blog

**Features**:

- Static homepage with Polaris styling
- Vimeo video portfolio (embedded players)
- Blog with rich text editor (ActionText)
- Draft/published/private status
- Basic Liquid templating

**Tech Focus**:

- Learn Rails MVC
- Set up Capistrano deployment
- Configure PostgreSQL
- Basic Nginx setup

---

### Phase 2: Public Stats (4-6 weeks)

**Goal**: Scraper + analytics dashboard

**Features**:

- Build Rails-native scraper (Mechanize + Nokogiri)
- Daily scraping via Sidekiq-cron
- Change detection for reviews
- Public stats dashboard (no auth required):
  - Top rated apps by category
  - Trending apps (rating changes)
  - Review volume over time
  - Category distribution

**Tech Focus**:

- Master Sidekiq background jobs
- Complex PostgreSQL queries
- Data visualization (Chart.js/D3.js)
- Progressive enhancement with vanilla JS

---

### Phase 3: Auth + Premium + Newsletter (3-4 weeks)

**Goal**: User accounts + tiered access

**Features**:

- Google OAuth login (Devise + OmniAuth)
- User roles: free/premium/admin
- Premium stats features:
  - Historical trend analysis
  - Competitor comparison tools
  - Review sentiment analysis
  - Export to CSV
  - Custom app bookmarks
- Newsletter signup integration
- Rate limiting for premium features

**Tech Focus**:

- Authentication flows
- Authorization with Pundit
- Newsletter API integration
- Session management

---

### Phase 4: AI Tools + Admin Polish (2-3 weeks)

**Goal**: Content automation + admin dashboard

**Features**:

- AI social media post generator:
  - Generate platform-specific posts from blog articles
  - Facebook, LinkedIn, Twitter, Instagram, TikTok
  - Suggest hashtags and images
- Admin dashboard:
  - Monitor scraping health
  - View failed jobs
  - Manage users
  - Approve AI-generated posts
- Audit logging for admin actions

**Tech Focus**:

- Anthropic API integration
- Prompt engineering for each platform
- Admin interface (ActiveAdmin or custom)
- Background job monitoring

---

### Phase 5: AV2Text - Transcription & Translation Service (TBD)

**Goal**: AI-powered transcription service for filmmakers

**Status**: Proof of concept complete in Python
([av2text repo](https://github.com/kleczekr/av2text))

**Features**:

- Upload audio/video files (web interface)
- Automatic compression and chunking
- Speech-to-text transcription via OpenAI Whisper API
- Optional translation to target language
- Annotation support for timestamped notes
- BYOK (Bring Your Own Key) - users provide their OpenAI API key
- WebSocket for real-time progress updates
- Download transcripts in multiple formats

**Technical Implementation**:

- **Option A**: Keep Python microservice, integrate with Rails via API
- **Option B**: Rewrite in Ruby using `ruby-openai` gem + `ffmpeg` for media
  processing
- File upload handling with Active Storage or similar
- Background job processing with Sidekiq
- Temporary storage management (cleanup after processing)
- Cost calculation and display (based on audio duration)

**Use Cases**:

- Film editors needing quick transcripts
- Content creators localizing videos
- Documentary filmmakers organizing interview footage
- Accessibility (generating subtitles)
- Podcast transcription

**Tech Focus**:

- File upload handling (large files)
- Media processing (compression, format conversion)
- OpenAI API integration
- WebSocket real-time updates
- User API key management (secure storage)

---

### Phase 6: Shot Planner - Visual Cinematography Tool (TBD)

**Goal**: Interactive shot planning tool for cinematographers

**Status**: Concept phase (requested by cinematographer collaborator)

**Core Concept**: Visual node-based interface where each node represents a shot,
connected by relationships (same location, similar angle, chronological
sequence, etc.)

**Features**:

- **Shot Management**:
  - Create shot nodes with detailed metadata
  - Shot number, scene reference, camera specs
  - Lens/focal length, lighting notes
  - Upload reference images per shot
  - Status tracking (planned → shot → edited)

- **Visual Organization**:
  - D3.js node graph visualization
  - Drag-and-drop repositioning
  - Cluster shots by location/setup
  - Timeline view (chronological vs. shooting order)
  - Color-coding by status/location

- **Collaboration**:
  - Share projects with crew members
  - Comment threads on individual shots
  - Real-time updates (ActionCable)
  - Role-based permissions (director, DP, gaffer, etc.)

- **Export & Output**:
  - PDF shot list
  - Shooting schedule grouped by location
  - Location-based call sheets
  - Reference image contact sheets

**Technical Stack**:

- D3.js for interactive node graph visualization
- Drag-and-drop interface (JavaScript + Rails backend)
- Image storage (separate mirakle-images repo or Cloudflare R2)
- Real-time collaboration via ActionCable WebSockets
- PostgreSQL with JSON columns for flexible metadata
- Canvas API for visual annotations on reference images

**UI/UX Design**:

- Desktop-first (cinematographers work on laptops)
- Touch-friendly for tablet use on set
- Extensive keyboard shortcuts for power users
- Print-friendly views (PDF generation)
- Dark mode (for low-light set environments)
- Responsive but optimized for large screens

**Tech Focus**:

- D3.js advanced visualizations
- ActionCable WebSocket real-time features
- Complex state management in frontend
- PDF generation from structured data
- Image annotation tools
- Collaborative editing with conflict resolution

---

## Total Timeline

**Core Platform (Phases 1-4)**: 3-4 months with consistent effort while learning
Rails

**Future Expansions (Phases 5-6)**: TBD based on user demand and available time

---

## Learning Resources

### Essential Books (in reading order)

1. **"Agile Web Development with Rails 7"** by Sam Ruby
   - THE definitive Rails book
   - Start here - covers MVC, ActiveRecord, conventions
   - Build a real app as you read

2. **"The Well-Grounded Rubyist"** by David A. Black
   - Deep dive into Ruby language itself
   - Understanding blocks, procs, lambdas
   - Read alongside Rails book

3. **"Eloquent Ruby"** by Russ Olsen
   - Ruby idioms and style
   - How to write idiomatic Ruby code
   - Read after you've written some code

### Specialized Resources

4. **"Rebuilding Rails"** by Noah Gibbs
   - Understand Rails internals by building a mini-framework
   - Advanced but illuminating

5. **"High Performance PostgreSQL for Rails"** by Andrew Atkinson
   - Database optimization for your analytics queries

### Online Resources

- **Rails Guides** (guides.rubyonrails.org) - Official, comprehensive, FREE
- **GoRails** (gorails.com) - Video screencasts, worth subscription
- **Ruby Tapas** by Avdi Grimm - Short focused videos on Ruby patterns
- **Railscasts** - Older but fundamentals still valuable
- **Rails API Documentation** - Excellent reference

### Community

- r/rails on Reddit
- Rails Discord
- Ruby on Rails Link Slack

---

## Key Features Summary

### Public Features (No Auth)

- Video portfolio (Vimeo embeds)
- Blog articles
- Basic Shopify App Store stats:
  - Top apps by category
  - Recent trending apps
  - Category overview

### Premium Features (Google OAuth)

- Historical trend analysis
- Competitor comparison tools
- App bookmarking with notes
- Export data to CSV
- Advanced filtering and search
- Custom alerts (future)

### Admin Features

- AI social media post generator
- Blog post management (draft/publish/private)
- User management
- Scraping job monitoring
- Audit logs
- Approve/edit AI-generated content

### Future Tools

- **AV2Text**: Transcription service (BYOK model)
- **Shot Planner**: Visual cinematography planning tool

---

## Next Steps

1. **Set up local development environment**
   - Install Ruby (rbenv/asdf)
   - Install Rails 7+
   - Create new Rails project with PostgreSQL
   - Install Redis

2. **Initialize Git repository**
   - Set up .gitignore
   - Create GitHub repo

3. **Start with Phase 1**
   - Build basic blog structure
   - Integrate Vimeo API
   - Set up Liquid templating
   - Deploy to Digital Ocean with Capistrano

4. **Learn as you build**
   - Read "Agile Web Development with Rails 7"
   - Watch GoRails screencasts
   - Experiment with Rails console

---

## Success Metrics

### Technical

- Daily scraping runs successfully
- Change detection catches 95%+ of review edits/deletions
- Page load times < 2 seconds for stats dashboard
- Zero downtime deployments with Capistrano

### Personal Growth

- Comfortable writing idiomatic Ruby
- Understanding Rails conventions deeply
- Can debug Rails apps efficiently
- Portfolio piece that showcases unique technical skills

### Product

- Functional analytics platform for Shopify App Store
- Professional portfolio showcasing creative + technical work
- Growing newsletter subscriber base
- Premium users finding value in advanced stats

---

_This is a living document. Update as project evolves and priorities shift._
