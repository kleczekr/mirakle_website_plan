# Mirakle Development Notes

Running notes, commands, and gotchas for daily development and deployment.

---

## Daily Development Workflow

### Starting Your Development Session

```bash
# 1. Start PostgreSQL (if not running)
sudo systemctl start postgresql
# Check status: sudo systemctl status postgresql

# 2. Start Redis (if not running)
sudo systemctl start redis
# Check status: redis-cli ping  # Should return PONG

# 3. Navigate to project
cd ~/path/to/mirakle

# 4. Start Rails server
rails server
# Or: rails s

# 5. (Optional) Start Sidekiq in another terminal
bundle exec sidekiq

# Visit: http://localhost:3000
```

### Stopping Services

```bash
# Stop Rails: Ctrl+C in terminal

# Stop PostgreSQL (optional, usually leave running)
sudo systemctl stop postgresql

# Stop Redis (optional, usually leave running)
sudo systemctl stop redis
```

---

## Blog Post Workflow

### Writing a New Post

```bash
# 1. Create markdown file
vim blog_posts/2025/10-october/2025-10-22-my-new-post.md

# 2. Add front matter and content
---
title: "My New Post"
slug: my-new-post
published_at: 2025-10-22T12:00:00Z
status: published  # or 'draft' or 'private'
tags: [ruby, rails, meta]
excerpt: "Brief description of the post"
featured_image: /blog_images/my-new-post/hero.jpg
---

# Your markdown content here...

# 3. Test locally
# Visit: http://localhost:3000/blog/my-new-post

# 4. Commit and push
git add blog_posts/
git commit -m "New post: My New Post"
git push origin main

# 5. Deploy (once Capistrano is set up)
cap production deploy
```

### Adding Images to Posts

```bash
# 1. Switch to image repo
cd ~/path/to/mirakle-images

# 2. Add images
mkdir -p blog/my-new-post
cp ~/Downloads/hero.jpg blog/my-new-post/

# 3. Commit and push
git add blog/
git commit -m "Images for: My New Post"
git push origin main

# 4. In your blog post markdown, reference as:
# ![Hero Image](/blog_images/my-new-post/hero.jpg)
```

### Clearing Blog Cache

If you update a post and don't see changes:

```bash
# Rails console
rails console

# Clear blog cache
Rails.cache.delete('blog_posts:all')

# Or restart Rails server
# Ctrl+C, then: rails s
```

---

## Database Commands

### Common PostgreSQL Tasks

```bash
# Access PostgreSQL console
psql

# List databases
\l

# Connect to Mirakle database
\c mirakle_development

# List tables
\dt

# Describe a table
\d table_name

# Exit psql
\q

# Create databases (if needed)
rails db:create

# Run migrations
rails db:migrate

# Rollback last migration
rails db:rollback

# Reset database (DESTRUCTIVE!)
rails db:reset

# Seed database
rails db:seed

# Check migration status
rails db:migrate:status
```

### Redis Commands

```bash
# Connect to Redis CLI
redis-cli

# Check if Redis is running
PING  # Should return PONG

# View all keys
KEYS *

# Get a specific key
GET blog_posts:all

# Delete a specific key
DEL blog_posts:all

# Clear all keys (DESTRUCTIVE!)
FLUSHALL

# Exit Redis CLI
EXIT
```

---

## Rails Console Tips

```bash
# Start Rails console
rails console
# Or: rails c

# Reload console after code changes
reload!

# Test blog post loading
BlogPost.all
BlogPost.published
BlogPost.find_by_slug('hello-mirakle')

# Check cache
Rails.cache.read('blog_posts:all')

# Clear cache
Rails.cache.clear

# Exit console
exit
```

---

## Testing & Debugging

### Check What's Running

```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Check if Redis is running
sudo systemctl status redis

# Check Rails server process
ps aux | grep rails

# Check if port 3000 is in use
lsof -i :3000

# Kill process on port 3000 (if stuck)
kill -9 $(lsof -t -i:3000)
```

### View Logs

```bash
# Rails development log (tail -f follows in real-time)
tail -f log/development.log

# Rails production log
tail -f log/production.log

# PostgreSQL logs (Arch Linux)
sudo journalctl -u postgresql -f

# Redis logs
sudo journalctl -u redis -f

# Nginx logs (on production server)
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### Common Issues & Fixes

**Issue: "Could not connect to server: No such file or directory"**

```bash
# PostgreSQL isn't running
sudo systemctl start postgresql
```

**Issue: "Error connecting to Redis"**

```bash
# Redis isn't running
sudo systemctl start redis
```

**Issue: "A server is already running"**

```bash
# Remove PID file
rm tmp/pids/server.pid
```

**Issue: "Bundler version mismatch"**

```bash
# Update bundler
gem install bundler
bundle install
```

**Issue: Blog post not showing up**

```bash
# Clear Rails cache
rails console
Rails.cache.delete('blog_posts:all')
```

---

## Gem Management

### Installing New Gems

```bash
# 1. Add gem to Gemfile
vim Gemfile

# 2. Install
bundle install

# 3. Restart Rails server
# Ctrl+C, then: rails s
```

### Updating Gems

```bash
# Update all gems
bundle update

# Update specific gem
bundle update gem_name

# Check for outdated gems
bundle outdated
```

---

## Deployment (Production Server)

### Prerequisites on Droplet

```bash
# SSH into droplet
ssh user@your-droplet-ip

# Ensure services are running
sudo systemctl status postgresql
sudo systemctl status redis
sudo systemctl status nginx

# Check Puma status (after first deploy)
sudo systemctl status puma
```

### Capistrano Deployment (Once Configured)

```bash
# From local machine:

# Check deployment configuration
cap production deploy:check

# Deploy to production
cap production deploy

# Rollback to previous release
cap production deploy:rollback

# View deployed releases
cap production releases

# Restart Puma
cap production puma:restart

# View logs
cap production logs:tail
```

### Manual Deployment (Before Capistrano Setup)

```bash
# 1. SSH into droplet
ssh user@your-droplet-ip

# 2. Navigate to app directory
cd /var/www/mirakle

# 3. Pull latest code
git pull origin main

# 4. Install dependencies
bundle install --deployment --without development test

# 5. Run migrations
RAILS_ENV=production bundle exec rails db:migrate

# 6. Precompile assets (if needed)
RAILS_ENV=production bundle exec rails assets:precompile

# 7. Restart Puma/Rails
sudo systemctl restart puma
# Or: touch tmp/restart.txt (if using Passenger)

# 8. Pull latest images
cd /var/www/mirakle-images
git pull origin main
```

### Post-Deployment Checks

```bash
# Check if site is up
curl -I https://your-domain.com

# Check Rails logs for errors
tail -f /var/www/mirakle/log/production.log

# Check Nginx logs
sudo tail -f /var/log/nginx/error.log

# Verify database connection
cd /var/www/mirakle
RAILS_ENV=production bundle exec rails console
# Try: BlogPost.all

# Check Sidekiq (for background jobs in Phase 2)
ps aux | grep sidekiq
```

---

## Environment Variables

### Local Development

```bash
# Create .env file (gitignored)
vim .env

# Add secrets:
DATABASE_URL=postgresql://localhost/mirakle_development
REDIS_URL=redis://localhost:6379/0
VIMEO_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here  # For AV2Text later

# Rails automatically loads .env in development
```

### Production

```bash
# On droplet, use systemd environment file or Rails credentials

# Option 1: Rails encrypted credentials
EDITOR=vim rails credentials:edit --environment production

# Option 2: Environment file for Puma
vim /etc/systemd/system/puma.service
# Add: Environment="VARIABLE=value"

# Reload systemd after changes
sudo systemctl daemon-reload
sudo systemctl restart puma
```

---

## Future: Sidekiq & Background Jobs (Phase 2)

### Starting Sidekiq Locally

```bash
# In separate terminal
bundle exec sidekiq

# Or with specific config
bundle exec sidekiq -C config/sidekiq.yml
```

### Sidekiq on Production

```bash
# Create systemd service
sudo vim /etc/systemd/system/sidekiq.service

# Start service
sudo systemctl start sidekiq
sudo systemctl enable sidekiq

# Check status
sudo systemctl status sidekiq

# View Sidekiq logs
sudo journalctl -u sidekiq -f

# Restart after code changes
sudo systemctl restart sidekiq
```

### Monitoring Sidekiq

```bash
# View Sidekiq web UI (add to routes.rb in Phase 2)
# Visit: http://localhost:3000/sidekiq

# Redis CLI - check job queues
redis-cli
LLEN "queue:default"  # Number of jobs in default queue
LLEN "queue:scraping"  # Number of scraping jobs
```

---

## Quick Reference: File Locations

```
mirakle/
├── app/
│   ├── controllers/        # HTTP request handlers
│   ├── models/             # Business logic & data models
│   ├── views/              # ERB/HTML templates
│   └── jobs/               # Background jobs (Phase 2)
├── blog_posts/             # Markdown blog posts (Git-based CMS)
│   ├── 2025/
│   ├── _drafts/
│   └── _private/
├── config/
│   ├── routes.rb           # URL routing
│   ├── database.yml        # Database config
│   └── environments/       # Environment-specific config
├── db/
│   ├── migrate/            # Database migrations
│   └── schema.rb           # Current database structure
├── log/                    # Application logs
├── tmp/                    # Temporary files & PIDs
└── Gemfile                 # Ruby dependencies

mirakle-images/             # Separate Git repo
├── blog/                   # Blog post images
├── portfolio/              # Video thumbnails
└── shot-planner/           # Future: reference images
```

---

## Emergency Commands

### Site is Down

```bash
# 1. Check if services are running
sudo systemctl status nginx
sudo systemctl status puma
sudo systemctl status postgresql
sudo systemctl status redis

# 2. Restart everything
sudo systemctl restart nginx
sudo systemctl restart puma

# 3. Check logs
sudo tail -50 /var/log/nginx/error.log
tail -50 /var/www/mirakle/log/production.log

# 4. If database issue
cd /var/www/mirakle
RAILS_ENV=production bundle exec rails db:migrate
```

### Accidentally Committed Secrets

```bash
# 1. Remove from Git history (nuclear option)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret/file" \
  --prune-empty --tag-name-filter cat -- --all

# 2. Force push (careful!)
git push origin main --force

# 3. Rotate the compromised secrets immediately!
```

### Database is Corrupted

```bash
# 1. Backup first!
pg_dump mirakle_production > backup_$(date +%Y%m%d).sql

# 2. Try to repair
RAILS_ENV=production bundle exec rails db:migrate:status
RAILS_ENV=production bundle exec rails db:migrate

# 3. Last resort: restore from backup
psql mirakle_production < backup_20251022.sql
```

---

## Helpful Aliases (Add to ~/.bashrc or ~/.zshrc)

```bash
# Rails shortcuts
alias rs='rails server'
alias rc='rails console'
alias rdb='rails db:migrate'
alias rgm='rails generate migration'

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push origin main'

# Mirakle specific
alias mirakle='cd ~/path/to/mirakle'
alias mirakle-images='cd ~/path/to/mirakle-images'
alias mirakle-services='sudo systemctl start postgresql redis'
alias mirakle-logs='tail -f log/development.log'

# After adding aliases, reload:
source ~/.bashrc  # or ~/.zshrc
```

---

## Regular Maintenance Tasks

### Daily (When Developing)

- [ ] Pull latest from `main` before starting work
- [ ] Start PostgreSQL & Redis
- [ ] Check logs for any overnight errors

### Weekly

- [ ] Review and clear old logs: `rails log:clear`
- [ ] Check for gem updates: `bundle outdated`
- [ ] Backup local database: `pg_dump mirakle_development > backup.sql`

### Monthly

- [ ] Update gems: `bundle update`
- [ ] Review disk space on droplet: `df -h`
- [ ] Check SSL certificate expiry (Let's Encrypt auto-renews)
- [ ] Review and archive old blog drafts

### Before Major Changes

- [ ] Create git branch: `git checkout -b feature-name`
- [ ] Backup production database
- [ ] Test locally first
- [ ] Deploy to staging (if you set one up later)

---

## When to Ask for Help

- PostgreSQL won't start after system update
- Bundler conflicts that `bundle update` can't resolve
- Mysterious 500 errors with no helpful logs
- Database migrations failing
- Capistrano deployment errors
- Asset pipeline issues (CSS/JS not loading)

**Pro tip:** Always check logs first! 90% of issues are explained in the logs.

---

## Learning Resources for Debugging

- **Rails Guides - Debugging**:
  https://guides.rubyonrails.org/debugging_rails_applications.html
- **PostgreSQL Documentation**: https://www.postgresql.org/docs/
- **Redis Commands Cheatsheet**: https://redis.io/commands
- **Stack Overflow**: Tag your questions with `ruby-on-rails` and `ruby`
- **Rails Discord**: Real-time help from the community

---

**Last Updated:** October 22, 2025 **Rails Version:** 8.1.0 **Ruby Version:**
3.4.7 **PostgreSQL Version:** 15+ **Redis Version:** 7+

---

_Keep this file updated as you discover new workflows and gotchas!_
