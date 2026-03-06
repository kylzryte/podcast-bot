---
type: deployment_guide
status: draft
version: 1.0
date: 2026-03-01
target_environment: Mac Mini (headless, macOS, Eastern Time)
---

# Gaming Industry Podcast Bot - Deployment Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Initial Setup](#initial-setup)
3. [Configuration](#configuration)
4. [Testing](#testing)
5. [Production Deployment](#production-deployment)
6. [Monitoring](#monitoring)
7. [Troubleshooting](#troubleshooting)

---

## 1. Prerequisites

### 1.1 System Requirements

**Hardware:**
- Mac Mini (or similar macOS device)
- Minimum 8GB RAM
- 50GB free disk space (for episodes, cache, logs)
- Active internet connection

**Software:**
- macOS 12+ (Monterey or later)
- Python 3.9 or higher
- Homebrew (package manager)
- Tailscale (for remote access)

**Access Requirements:**
- SSH access to Mac Mini
- Full Disk Access permissions for Python/Terminal
- API keys: Perplexity, OpenAI, Anthropic Claude

### 1.2 Dependencies

```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Python 3.9+
brew install python@3.9

# Install ffmpeg (required by pydub)
brew install ffmpeg

# Install Tailscale
brew install --cask tailscale

# Verify installations
python3 --version  # Should be 3.9+
ffmpeg -version
```

---

## 2. Initial Setup

### 2.1 Create Project Directory

```bash
# Create base directory
mkdir -p ~/podcast
cd ~/podcast

# Create subdirectories
mkdir -p episodes scripts research memories logs temp prompts lib
mkdir -p memories/Archive

# Create placeholder memory files
touch memories/Nigel-Memory.md
touch memories/Marcus-Memory.md
touch memories/Emily-Memory.md
```

### 2.2 Clone/Copy Project Files

**Option A: Copy from development machine**
```bash
# From development machine, copy files to Mac Mini
scp -r /path/to/podcast_bot/ kylebillings@192.168.0.180:~/podcast/
```

**Option B: Manual file creation**
```bash
# Create main script
touch ~/podcast/podcast_bot.py
chmod +x ~/podcast/podcast_bot.py

# Create library modules (see Implementation Guide for code)
touch ~/podcast/lib/__init__.py
touch ~/podcast/lib/digest_parser.py
touch ~/podcast/lib/research_generator.py
touch ~/podcast/lib/article_selector.py
touch ~/podcast/lib/memory_manager.py
touch ~/podcast/lib/script_generator.py
touch ~/podcast/lib/audio_producer.py
touch ~/podcast/lib/rss_generator.py
touch ~/podcast/lib/prediction_tracker.py
touch ~/podcast/lib/config.py

# Create HTTP server
touch ~/podcast/server.py
chmod +x ~/podcast/server.py
```

### 2.3 Install Python Dependencies

```bash
cd ~/podcast

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Create requirements.txt
cat > requirements.txt << EOF
openai>=1.0.0
anthropic>=0.39.0
requests>=2.31.0
PyYAML>=6.0.1
python-dotenv>=1.0.0
pydub>=0.25.1
feedgen>=0.9.0
EOF

# Install dependencies
pip install -r requirements.txt

# Verify installations
python -c "import openai; print('OpenAI:', openai.__version__)"
python -c "import anthropic; print('Anthropic:', anthropic.__version__)"
python -c "from pydub import AudioSegment; print('Pydub: OK')"
```

---

## 3. Configuration

### 3.1 API Keys Setup

```bash
cd ~/podcast

# Create .env file
touch .env
chmod 600 .env  # Secure permissions

# Edit .env file
nano .env
```

**`.env` contents:**
```bash
# Perplexity API
PERPLEXITY_API_KEY=pplx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# OpenAI API
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Anthropic Claude API
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Save and verify:**
```bash
# Test API keys
python3 << EOF
import os
from dotenv import load_dotenv

load_dotenv()

keys = {
    'PERPLEXITY_API_KEY': os.environ.get('PERPLEXITY_API_KEY'),
    'OPENAI_API_KEY': os.environ.get('OPENAI_API_KEY'),
    'ANTHROPIC_API_KEY': os.environ.get('ANTHROPIC_API_KEY')
}

for name, key in keys.items():
    if key and len(key) > 20:
        print(f"✓ {name}: {key[:10]}...{key[-10:]}")
    else:
        print(f"✗ {name}: MISSING OR INVALID")
EOF
```

### 3.2 Configuration File

```bash
# Create podcast_config.yaml
cat > ~/podcast/podcast_config.yaml << 'EOF'
apis:
  perplexity:
    model: "sonar-pro"
    max_tokens: 2000
    temperature: 0.3
    api_key_env: "PERPLEXITY_API_KEY"

  openai:
    model: "gpt-4"
    tts_model: "tts-1"
    tts_voices:
      nigel: "alloy"
      marcus: "onyx"
      emily: "shimmer"
    max_tokens: 4000
    temperature: 0.4
    api_key_env: "OPENAI_API_KEY"

  claude:
    model: "claude-sonnet-4"
    max_tokens: 16000
    temperature: 0.7
    api_key_env: "ANTHROPIC_API_KEY"

hosts:
  - name: "Nigel"
    full_name: "Nigel Thornbury"
    age: "late 60s"
    accent: "British"
    personality: "Bombastic, loud, curses, contrarian"
    voice: "alloy"
  - name: "Marcus"
    full_name: "Marcus Webb"
    age: "early 30s"
    accent: "American"
    personality: "Brash, know-it-all, jumps to conclusions"
    voice: "onyx"
  - name: "Emily"
    full_name: "Emily Bradford"
    age: "late 20s"
    accent: "British"
    personality: "Optimistic, rose-colored glasses, community-focused"
    voice: "shimmer"

episode:
  target_length_minutes: 90
  target_word_count: 14000
  min_word_count: 10000
  max_word_count: 20000
  article_count_range: [6, 15]
  coverage_period_days: 7

scheduling:
  daily_research_time: "12:30"
  weekly_generation_time: "05:00"
  timezone: "America/New_York"

audio:
  format: "mp3"
  bitrate: "128k"
  sample_rate: 24000
  pause_between_speakers: 0.5
  pause_interrupt: 0.1
  pause_laugh: 0.5

memory:
  retention_months: 6
  summarize_every_episodes: 4
  prediction_auto_resolve: true

rss:
  title: "Gaming Industry Podcast"
  description: "Weekly discussion of gaming industry developments by Nigel Thornbury, Marcus Webb, and Emily Bradford."
  language: "en-us"
  max_episodes_in_feed: 52
  base_url: "http://macmini.tailnet:8000"

server:
  port: 8000
  directory: "/Users/kylebillings/podcast"

paths:
  base: "/Users/kylebillings/podcast"
  episodes: "episodes"
  scripts: "scripts"
  research: "research"
  memories: "memories"
  prompts: "prompts"
  temp: "temp"
  logs: "logs"
  digests: "/Users/kylebillings/Desktop/Tanalorr/50_OUTPUT/News Bot/Daily News"

logging:
  level: "INFO"
  file: "/Users/kylebillings/podcast/logs/podcast_bot.log"
  max_bytes: 10485760
  backup_count: 5
EOF
```

### 3.3 Initialize Memory Files

```bash
# Nigel's initial memory
cat > ~/podcast/memories/Nigel-Memory.md << 'EOF'
# Nigel Thornbury Memory

**Last Updated:** 2026-03-01

## Canonical Personality

- Late 60s British gaming industry veteran (since Atari era)
- Bombastic, loud, interrupts frequently
- Curses liberally (F-bombs, "bloody hell", "bollocks")
- Contrarian - plays devil's advocate
- Secretly respects good game design despite cynicism
- References historical precedents constantly
- Cuts through marketing BS instantly

## Recent Opinions

(Will be populated after first episode)

## Active Predictions

(None yet)

## Resolved Predictions

(None yet)

## Running Jokes

- Mispronounces modern gaming terms intentionally
- "Back when I was at [company]..." stories
- Dismisses trends: "Another battle royale, how original"
- "I've seen this all before..."

## Prediction Accuracy

- **Total Predictions:** 0
- **Resolved:** 0
- **Correct:** 0
- **Incorrect:** 0
- **Accuracy:** N/A
EOF

# Marcus's initial memory
cat > ~/podcast/memories/Marcus-Memory.md << 'EOF'
# Marcus Webb Memory

**Last Updated:** 2026-03-01

## Canonical Personality

- Early 30s American tech analyst
- Brash, overconfident, know-it-all energy
- Jumps to conclusions quickly
- Makes bold predictions constantly
- Obsessed with data, metrics, and disruption
- Overuses buzzwords and industry jargon
- Doubles down when challenged

## Recent Opinions

(Will be populated after first episode)

## Active Predictions

(None yet)

## Resolved Predictions

(None yet)

## Running Jokes

- "I'm calling it now..." (prediction marker)
- Cites obscure player metrics constantly
- "The data shows..." even when it doesn't
- Overconfident: "This is obvious..."

## Prediction Accuracy

- **Total Predictions:** 0
- **Resolved:** 0
- **Correct:** 0
- **Incorrect:** 0
- **Accuracy:** N/A
EOF

# Emily's initial memory
cat > ~/podcast/memories/Emily-Memory.md << 'EOF'
# Emily Bradford Memory

**Last Updated:** 2026-03-01

## Canonical Personality

- Late 20s British former community manager
- Optimistic, enthusiastic, rose-colored glasses
- Community-focused perspective
- Defends developers and studios
- Cites Reddit/Discord/Twitter sentiment
- Defers to Nigel's experience
- Genuinely excited about game announcements

## Recent Opinions

(Will be populated after first episode)

## Active Predictions

(Rarely makes predictions - more reactive analysis)

## Resolved Predictions

(None yet)

## Running Jokes

- "But the community loves..." (optimism trigger)
- "I think they're really trying here..."
- Always finds silver lining in bad news
- Gets corrected by Nigel's historical context

## Prediction Accuracy

- **Total Predictions:** 0
- **Resolved:** 0
- **Correct:** 0
- **Incorrect:** 0
- **Accuracy:** N/A
EOF
```

### 3.4 HTTP Server Setup

```bash
# Create simple HTTP server
cat > ~/podcast/server.py << 'EOF'
#!/usr/bin/env python3
"""
Simple HTTP server for podcast RSS feed and audio files
"""

import http.server
import socketserver
import os

PORT = 8000
DIRECTORY = "/Users/kylebillings/podcast"

class PodcastHandler(http.server.SimpleHTTPRequestHandler):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, directory=DIRECTORY, **kwargs)

    def log_message(self, format, *args):
        # Log requests to file
        with open(f"{DIRECTORY}/logs/server.log", "a") as f:
            f.write(f"{self.address_string()} - [{self.log_date_time_string()}] {format % args}\n")

if __name__ == "__main__":
    os.chdir(DIRECTORY)
    with socketserver.TCPServer(("", PORT), PodcastHandler) as httpd:
        print(f"Serving podcast directory on port {PORT}")
        print(f"Access via: http://localhost:{PORT}/feed.xml")
        httpd.serve_forever()
EOF

chmod +x ~/podcast/server.py

# Test server
python3 ~/podcast/server.py &
SERVER_PID=$!
sleep 2
curl -I http://localhost:8000/
kill $SERVER_PID
```

---

## 4. Testing

### 4.1 Unit Tests

```bash
cd ~/podcast

# Activate virtual environment
source venv/bin/activate

# Run unit tests
python3 -m pytest tests/ -v
```

### 4.2 Daily Research Test

```bash
# Test with existing digest
python3 podcast_bot.py --mode daily-research --config podcast_config.yaml

# Verify research files created
ls -lh research/$(date +%Y-%m-%d)/

# Check logs
tail -f logs/podcast_bot.log
```

### 4.3 Weekly Generation Test

```bash
# Requires 7 days of research files to be present

# Run weekly generation
python3 podcast_bot.py --mode weekly-generate --config podcast_config.yaml

# Verify outputs
ls -lh scripts/
ls -lh episodes/
cat feed.xml

# Test audio playback
afplay episodes/*.mp3  # Play on Mac
```

---

## 5. Production Deployment

### 5.1 Cron Job Setup

```bash
# Open crontab editor
crontab -e
```

**Add cron jobs:**
```cron
# Gaming Industry Podcast Bot
# Daily research (Mon-Sat, 12:30pm ET)
30 12 * * 1-6 cd /Users/kylebillings/podcast && /Users/kylebillings/podcast/venv/bin/python3 podcast_bot.py --mode daily-research >> logs/cron.log 2>&1

# Weekly generation (Sunday, 5:00am ET)
0 5 * * 0 cd /Users/kylebillings/podcast && /Users/kylebillings/podcast/venv/bin/python3 podcast_bot.py --mode weekly-generate >> logs/cron.log 2>&1
```

**Save and verify:**
```bash
# List cron jobs
crontab -l

# Test cron execution
# Wait for scheduled time or manually trigger:
cd ~/podcast && venv/bin/python3 podcast_bot.py --mode daily-research
```

### 5.2 HTTP Server LaunchAgent

```bash
# Create LaunchAgent for HTTP server (starts on boot)
cat > ~/Library/LaunchAgents/com.podcast.server.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.podcast.server</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/kylebillings/podcast/venv/bin/python3</string>
        <string>/Users/kylebillings/podcast/server.py</string>
    </array>

    <key>WorkingDirectory</key>
    <string>/Users/kylebillings/podcast</string>

    <key>StandardOutPath</key>
    <string>/Users/kylebillings/podcast/logs/server.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/kylebillings/podcast/logs/server_error.log</string>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
EOF

# Load LaunchAgent
launchctl load ~/Library/LaunchAgents/com.podcast.server.plist

# Verify server is running
lsof -i :8000

# Test access
curl http://localhost:8000/feed.xml
```

### 5.3 Tailscale Setup

```bash
# Start Tailscale
sudo tailscale up

# Get Tailscale IP
tailscale ip -4

# Note your Mac Mini hostname (e.g., macmini.tailnet)
# Access from Android: http://macmini.tailnet:8000/feed.xml
```

**Android Podcast App Setup:**
1. Open podcast app (e.g., Podcast Addict, AntennaPod)
2. Add podcast via RSS URL
3. Enter: `http://macmini.tailnet:8000/feed.xml`
4. Subscribe and download episodes

### 5.4 Full Disk Access (macOS)

```bash
# Grant Full Disk Access to Python and Terminal
# Required for cron jobs to access files

# System Settings → Privacy & Security → Full Disk Access
# Add:
# - /usr/bin/python3
# - /bin/bash
# - Terminal.app
# - SSH (if using remote cron)
```

---

## 6. Monitoring

### 6.1 Health Checks

**Daily Check Script:**
```bash
cat > ~/podcast/health_check.sh << 'EOF'
#!/bin/bash
# Daily health check for podcast bot

PODCAST_DIR="/Users/kylebillings/podcast"
TODAY=$(date +%Y-%m-%d)

echo "=== Podcast Bot Health Check - $TODAY ==="

# Check if daily research ran
RESEARCH_DIR="$PODCAST_DIR/research/$TODAY"
if [ -d "$RESEARCH_DIR" ]; then
    FILE_COUNT=$(ls -1 "$RESEARCH_DIR" | wc -l)
    echo "✓ Daily research: $FILE_COUNT files"
else
    echo "✗ Daily research: MISSING"
fi

# Check HTTP server
if curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/ | grep -q "200"; then
    echo "✓ HTTP server: Running"
else
    echo "✗ HTTP server: DOWN"
fi

# Check disk space
DISK_USAGE=$(df -h "$PODCAST_DIR" | tail -1 | awk '{print $5}')
echo "Disk usage: $DISK_USAGE"

# Check latest episode
LATEST_EPISODE=$(ls -t "$PODCAST_DIR/episodes" | head -1)
if [ -n "$LATEST_EPISODE" ]; then
    echo "Latest episode: $LATEST_EPISODE"
else
    echo "No episodes found"
fi

# Check log file size
LOG_SIZE=$(du -h "$PODCAST_DIR/logs/podcast_bot.log" | awk '{print $1}')
echo "Log size: $LOG_SIZE"
EOF

chmod +x ~/podcast/health_check.sh

# Run manually
~/podcast/health_check.sh

# Or add to daily cron
# 0 18 * * * /Users/kylebillings/podcast/health_check.sh | mail -s "Podcast Bot Health" your@email.com
```

### 6.2 Log Monitoring

```bash
# Watch logs in real-time
tail -f ~/podcast/logs/podcast_bot.log

# Check for errors
grep ERROR ~/podcast/logs/podcast_bot.log | tail -20

# Check cost tracking
grep "Tokens:" ~/podcast/logs/podcast_bot.log | tail -50
```

### 6.3 Cost Monitoring

```bash
# Monthly cost report
cat > ~/podcast/monthly_cost_report.py << 'EOF'
#!/usr/bin/env python3
import re
from datetime import datetime
from pathlib import Path

log_file = Path.home() / "podcast/logs/podcast_bot.log"

# Parse token usage from logs
perplexity_tokens = 0
openai_tokens = 0
claude_tokens = 0

with open(log_file, 'r') as f:
    for line in f:
        if 'Perplexity' in line and 'tokens' in line:
            match = re.search(r'(\d+)\s+tokens', line)
            if match:
                perplexity_tokens += int(match.group(1))
        # Similar for OpenAI and Claude...

# Calculate costs
perplexity_cost = (perplexity_tokens / 1000) * 0.02
openai_cost = (openai_tokens / 1000) * 0.015  # Simplified
claude_cost = (claude_tokens / 1000) * 0.015  # Simplified

print(f"Monthly Token Usage:")
print(f"  Perplexity: {perplexity_tokens:,} tokens (${perplexity_cost:.2f})")
print(f"  OpenAI: {openai_tokens:,} tokens (${openai_cost:.2f})")
print(f"  Claude: {claude_tokens:,} tokens (${claude_cost:.2f})")
print(f"  TOTAL: ${perplexity_cost + openai_cost + claude_cost:.2f}")
EOF

chmod +x ~/podcast/monthly_cost_report.py
python3 ~/podcast/monthly_cost_report.py
```

---

## 7. Troubleshooting

### 7.1 Common Issues

**Issue: Cron job doesn't run**
```bash
# Check cron logs
cat ~/podcast/logs/cron.log

# Check system logs
grep CRON /var/log/system.log

# Verify Full Disk Access permissions
# System Settings → Privacy & Security → Full Disk Access
```

**Issue: API authentication fails**
```bash
# Verify .env file
cat ~/podcast/.env

# Check environment variables
python3 << EOF
import os
from dotenv import load_dotenv
load_dotenv('/Users/kylebillings/podcast/.env')
print(os.environ.get('PERPLEXITY_API_KEY')[:20])
EOF

# Test API directly
curl -H "Authorization: Bearer $PERPLEXITY_API_KEY" \
  https://api.perplexity.ai/chat/completions \
  -d '{"model":"sonar-pro","messages":[{"role":"user","content":"test"}]}'
```

**Issue: Audio stitching fails**
```bash
# Check ffmpeg installation
ffmpeg -version

# Test pydub
python3 << EOF
from pydub import AudioSegment
print("Pydub working correctly")
EOF

# Reinstall if needed
pip install --upgrade pydub
brew reinstall ffmpeg
```

**Issue: HTTP server not accessible**
```bash
# Check if server is running
lsof -i :8000

# Check firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# Restart server
launchctl unload ~/Library/LaunchAgents/com.podcast.server.plist
launchctl load ~/Library/LaunchAgents/com.podcast.server.plist
```

**Issue: Tailscale connection fails**
```bash
# Check Tailscale status
tailscale status

# Restart Tailscale
sudo tailscale down
sudo tailscale up

# Verify DNS
tailscale ip -4
```

### 7.2 Recovery Procedures

**Recover from failed weekly generation:**
```bash
# Check what stage failed
tail -100 ~/podcast/logs/podcast_bot.log

# If article selection failed: Re-run with fewer articles
# Edit config: episode.article_count_range: [4, 8]

# If script generation failed: Check Claude API status
# Retry manually:
cd ~/podcast
source venv/bin/activate
python3 podcast_bot.py --mode weekly-generate
```

**Restore from backup:**
```bash
# Backup strategy (weekly)
tar -czf ~/podcast_backup_$(date +%Y%m%d).tar.gz \
  ~/podcast/memories \
  ~/podcast/scripts \
  ~/podcast/episodes \
  ~/podcast/feed.xml

# Restore
tar -xzf ~/podcast_backup_YYYYMMDD.tar.gz -C ~/
```

---

## 8. Deployment Checklist

**Pre-Deployment:**
- [ ] Mac Mini meets system requirements
- [ ] Python 3.9+ installed
- [ ] ffmpeg installed
- [ ] Tailscale installed and configured
- [ ] API keys obtained (Perplexity, OpenAI, Claude)

**Initial Setup:**
- [ ] Project directory created: `~/podcast/`
- [ ] All subdirectories created
- [ ] Virtual environment created and activated
- [ ] Dependencies installed from `requirements.txt`
- [ ] `.env` file created with API keys (chmod 600)
- [ ] `podcast_config.yaml` configured with correct paths
- [ ] Initial memory files created
- [ ] Prompt templates copied to `prompts/`

**Testing:**
- [ ] Unit tests pass
- [ ] Daily research test successful
- [ ] Weekly generation test successful (with mock data)
- [ ] Audio playback works
- [ ] RSS feed validates
- [ ] HTTP server serves files correctly
- [ ] Tailscale access works from Android

**Production:**
- [ ] Cron jobs configured (daily + weekly)
- [ ] HTTP server LaunchAgent loaded
- [ ] Full Disk Access granted
- [ ] Health check script scheduled
- [ ] Log rotation configured
- [ ] Backup strategy implemented

**Verification:**
- [ ] First daily research runs successfully
- [ ] Research files appear in correct location
- [ ] Logs show no errors
- [ ] Cost tracking logs token usage
- [ ] HTTP server accessible via Tailscale
- [ ] Android podcast app can subscribe to feed

**Post-Deployment:**
- [ ] Monitor logs for first week
- [ ] Verify first episode generation (Sunday)
- [ ] Check episode quality and length
- [ ] Validate character personalities
- [ ] Confirm prediction tracking works
- [ ] Review and optimize costs

---

## 9. Maintenance

**Weekly:**
- Review episode quality
- Check prediction accuracy
- Monitor disk usage
- Review logs for errors

**Monthly:**
- Generate cost report
- Archive old episodes (>1 year)
- Update host memories if needed
- Review character consistency

**Quarterly:**
- Backup all data
- Update dependencies
- Review system performance
- Optimize prompts based on output quality

---

## Support

For issues or questions:
- Check logs: `~/podcast/logs/podcast_bot.log`
- Review PRD: `50_OUTPUT/Podcast Bot/PRD - Gaming Industry Podcast Bot.md`
- Review Implementation Guide: `50_OUTPUT/Podcast Bot/Implementation Guide.md`

**Status:** Ready for deployment once code is implemented.

[[Podcast Bot]] | [[Deployment]] | [[Mac Mini]] | [[Automation]]
