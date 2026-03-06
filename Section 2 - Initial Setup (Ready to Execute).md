---
type: deployment_guide
section: Initial Setup (Section 2)
status: ready_to_execute_after_section_1
date: 2026-02-25
---

# Section 2: Initial Setup - Ready to Execute

## Prerequisites

Before running these commands, Section 1 must be complete:
- ✓ 50+ GB disk space available
- ✓ Python 3.9+ installed
- ✓ Homebrew installed
- ✓ ffmpeg installed
- ✓ Tailscale installed
- ✓ All API keys present

## Overview

This section creates the podcast project structure and installs all required Python dependencies.

**Estimated Time:** 30 minutes

## Step 2.1: Create Project Directory Structure

Run these commands on the Mac Mini:

```bash
# Create main project directory
mkdir -p ~/podcast

# Create all subdirectories
cd ~/podcast
mkdir -p {episodes,scripts,research,memories,prompts,logs,cache}

# Create nested research structure
mkdir -p research/{daily,selected}

# Verify structure
tree -L 2 ~/podcast
# OR if tree not installed:
ls -R ~/podcast
```

**Expected structure:**
```
~/podcast/
├── episodes/           # Final MP3 files
├── scripts/            # Generated dialogue scripts
├── research/
│   ├── daily/         # Daily research by host (Mon-Sat)
│   └── selected/      # Research for selected articles
├── memories/          # Host memory files
├── prompts/           # Prompt templates
├── logs/              # Application logs
└── cache/             # Temporary audio files
```

## Step 2.2: Create Python Virtual Environment

```bash
cd ~/podcast

# Create virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate

# Verify activation
which python
# Should show: /Users/kyle/podcast/venv/bin/python

# Upgrade pip
pip install --upgrade pip
```

## Step 2.3: Install Python Dependencies

Create `requirements.txt`:

```bash
cat > ~/podcast/requirements.txt << 'EOF'
# API Clients
openai==1.12.0
anthropic==0.18.1
requests==2.31.0

# Audio Processing
pydub==0.25.1

# RSS Feed Generation
feedgen==1.0.0

# Configuration
python-dotenv==1.0.0
PyYAML==6.0.1

# Utilities
python-dateutil==2.8.2

# Logging
colorlog==6.8.0
EOF
```

Install all packages:

```bash
cd ~/podcast
source venv/bin/activate
pip install -r requirements.txt

# Verify installations
pip list | grep -E "(openai|anthropic|pydub|feedgen)"
```

**Expected output:**
```
anthropic         0.18.1
feedgen           1.0.0
openai            1.12.0
pydub             0.25.1
```

## Step 2.4: Create Configuration File

Create `podcast_config.yaml`:

```bash
cat > ~/podcast/podcast_config.yaml << 'EOF'
# Podcast Bot Configuration

# Project Paths
project_root: /Users/kyle/podcast
episodes_dir: /Users/kyle/podcast/episodes
scripts_dir: /Users/kyle/podcast/scripts
research_dir: /Users/kyle/podcast/research
memories_dir: /Users/kyle/podcast/memories
prompts_dir: /Users/kyle/podcast/prompts
logs_dir: /Users/kyle/podcast/logs
cache_dir: /Users/kyle/podcast/cache

# Source Data
news_digest_dir: /Users/kyle/news_bot/digests

# API Configuration (keys loaded from .env)
perplexity:
  model: "sonar-pro"
  max_tokens: 2000
  temperature: 0.3

openai:
  model: "gpt-4-turbo-preview"
  tts_model: "tts-1"
  max_tokens: 4000
  temperature: 0.7
  # Voice assignments
  voices:
    nigel: "onyx"      # Deep British male
    marcus: "echo"     # American male
    emily: "nova"      # British female

anthropic:
  model: "claude-sonnet-4-20250514"
  max_tokens: 16000
  temperature: 0.7

# Podcast Settings
podcast:
  title: "Gaming Industry Podcast"
  description: "Weekly gaming industry analysis featuring three distinct analyst perspectives"
  language: "en-us"
  author: "Nigel Thornbury, Marcus Webb, Emily Bradford"

  # Episode Settings
  target_duration_minutes: 90
  target_word_count: 14500  # ~90 min at 160 wpm
  min_articles: 6
  max_articles: 15

  # Audio Settings
  audio_format: "mp3"
  audio_bitrate: "192k"
  sample_rate: 24000

# Host Memory Settings
memory:
  max_predictions: 20
  max_opinions: 30
  max_running_jokes: 10
  archive_after_months: 6

# Scheduling (for reference - actual scheduling via cron)
schedule:
  daily_research: "Mon-Sat 12:30 PM ET"
  weekly_generation: "Sunday 5:00 AM ET"
  coverage_period_days: 7

# Logging
logging:
  level: "INFO"
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  max_bytes: 10485760  # 10MB
  backup_count: 5

# RSS Feed
rss:
  feed_url: "http://192.168.0.180:8000/feed.xml"
  max_episodes: 52
  image_url: ""  # Optional podcast artwork
EOF
```

## Step 2.5: Create Environment File

```bash
# Copy existing .env from news_bot and add OpenAI key
cp ~/news_bot/.env ~/podcast/.env

# Verify all keys present
cat ~/podcast/.env

# Should contain:
# PERPLEXITY_API_KEY=...
# ANTHROPIC_API_KEY=...
# OPENAI_API_KEY=...

# Set secure permissions
chmod 600 ~/podcast/.env
```

**If OpenAI key is missing:**
```bash
echo "OPENAI_API_KEY=your-actual-key-here" >> ~/podcast/.env
```

## Step 2.6: Create Basic Config Loader

Create `lib/config.py`:

```bash
mkdir -p ~/podcast/lib

cat > ~/podcast/lib/config.py << 'EOF'
"""Configuration loader for Podcast Bot."""

import os
import yaml
from pathlib import Path
from dotenv import load_dotenv

def load_config():
    """Load configuration from YAML and .env files."""

    # Load environment variables
    env_path = Path(__file__).parent.parent / '.env'
    load_dotenv(env_path)

    # Load YAML config
    config_path = Path(__file__).parent.parent / 'podcast_config.yaml'
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)

    # Add API keys from environment
    config['api_keys'] = {
        'perplexity': os.getenv('PERPLEXITY_API_KEY'),
        'openai': os.getenv('OPENAI_API_KEY'),
        'anthropic': os.getenv('ANTHROPIC_API_KEY'),
    }

    return config

def get_project_root():
    """Get the project root directory."""
    return Path(__file__).parent.parent

if __name__ == '__main__':
    # Test configuration loading
    config = load_config()
    print("✓ Configuration loaded successfully")
    print(f"  Project root: {config['project_root']}")
    print(f"  Perplexity API key: {'✓' if config['api_keys']['perplexity'] else '✗'}")
    print(f"  OpenAI API key: {'✓' if config['api_keys']['openai'] else '✗'}")
    print(f"  Anthropic API key: {'✓' if config['api_keys']['anthropic'] else '✗'}")
EOF
```

## Step 2.7: Test Configuration

```bash
cd ~/podcast
source venv/bin/activate
python3 lib/config.py
```

**Expected output:**
```
✓ Configuration loaded successfully
  Project root: /Users/kyle/podcast
  Perplexity API key: ✓
  OpenAI API key: ✓
  Anthropic API key: ✓
```

## Step 2.8: Initialize Host Memory Files

Create initial memory files for all three hosts:

```bash
# Create Nigel's memory
cat > ~/podcast/memories/nigel_thornbury.md << 'EOF'
# Nigel Thornbury - Host Memory

## Personality Traits
- Bombastic and loud
- Profane (frequent British expletives)
- Contrarian and cynical
- References gaming history (1980s-2000s)
- Pattern recognition expert
- Interrupts frequently

## Signature Phrases
- "Bloody hell"
- "Bollocks"
- "Mice nuts"
- "I've seen this all before"
- "Here we go again"

## Active Predictions
(None yet - first episode)

## Recent Opinions
(None yet - first episode)

## Running Jokes
(None yet - first episode)

## Resolved Predictions
(None yet - first episode)
EOF

# Create Marcus's memory
cat > ~/podcast/memories/marcus_webb.md << 'EOF'
# Marcus Webb - Host Memory

## Personality Traits
- Brash know-it-all
- Data-obsessed
- Overconfident predictor
- Fast-paced speaker
- Buzzword enthusiast
- Doubles down when challenged

## Signature Phrases
- "I'm calling it now"
- "The data shows"
- "Paradigm shift"
- "Literally"
- "Ecosystem"

## Active Predictions
(None yet - first episode)

## Recent Opinions
(None yet - first episode)

## Prediction Accuracy
(None yet - first episode)

## Resolved Predictions
(None yet - first episode)
EOF

# Create Emily's memory
cat > ~/podcast/memories/emily_bradford.md << 'EOF'
# Emily Bradford - Host Memory

## Personality Traits
- Optimistic and enthusiastic
- Community-focused
- Defensive of developers
- Thoughtful and measured
- Uses qualifiers frequently
- Persists when interrupted

## Signature Phrases
- "But the community loves"
- "To be fair"
- "I think they're really trying"
- "Actually" (frequent)

## Active Predictions
(None yet - first episode)

## Recent Opinions
(None yet - first episode)

## Community Insights
(None yet - first episode)

## Resolved Predictions
(None yet - first episode)
EOF

# Verify memory files
ls -la ~/podcast/memories/
```

## Step 2.9: Set Up Logging

Create logging configuration:

```bash
cat > ~/podcast/lib/logging_config.py << 'EOF'
"""Logging configuration for Podcast Bot."""

import logging
import sys
from pathlib import Path
from logging.handlers import RotatingFileHandler

def setup_logging(name='podcast_bot', config=None):
    """Set up logging with both file and console handlers."""

    # Get log settings from config or use defaults
    if config:
        log_dir = Path(config['logs_dir'])
        level = config['logging']['level']
        max_bytes = config['logging']['max_bytes']
        backup_count = config['logging']['backup_count']
    else:
        log_dir = Path(__file__).parent.parent / 'logs'
        level = 'INFO'
        max_bytes = 10485760
        backup_count = 5

    # Create logs directory
    log_dir.mkdir(exist_ok=True)
    log_file = log_dir / 'podcast_bot.log'

    # Create logger
    logger = logging.getLogger(name)
    logger.setLevel(getattr(logging, level))

    # File handler with rotation
    file_handler = RotatingFileHandler(
        log_file,
        maxBytes=max_bytes,
        backupCount=backup_count
    )
    file_handler.setLevel(logging.DEBUG)
    file_format = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_handler.setFormatter(file_format)

    # Console handler
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_format = logging.Formatter(
        '%(levelname)s: %(message)s'
    )
    console_handler.setFormatter(console_format)

    # Add handlers
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)

    return logger

if __name__ == '__main__':
    # Test logging
    logger = setup_logging()
    logger.info("Logging system initialized")
    logger.debug("Debug message test")
    logger.warning("Warning message test")
    print(f"✓ Log file created at: logs/podcast_bot.log")
EOF

# Test logging
cd ~/podcast
source venv/bin/activate
python3 lib/logging_config.py
```

## Step 2.10: Verify Complete Setup

Final verification checklist:

```bash
cd ~/podcast

# Check directory structure
echo "=== Directory Structure ==="
ls -la

# Check virtual environment
echo -e "\n=== Virtual Environment ==="
source venv/bin/activate
which python

# Check installed packages
echo -e "\n=== Installed Packages ==="
pip list | grep -E "(openai|anthropic|pydub|feedgen|PyYAML)"

# Check configuration
echo -e "\n=== Configuration Test ==="
python3 lib/config.py

# Check logging
echo -e "\n=== Logging Test ==="
python3 lib/logging_config.py

# Check memory files
echo -e "\n=== Memory Files ==="
ls -la memories/

# Check permissions
echo -e "\n=== .env Permissions ==="
ls -la .env
```

## Section 2 Completion Checklist

- [ ] Project directory structure created
- [ ] Virtual environment created and activated
- [ ] All Python packages installed
- [ ] `podcast_config.yaml` created
- [ ] `.env` file with all API keys (secure permissions)
- [ ] `lib/config.py` working
- [ ] `lib/logging_config.py` working
- [ ] Host memory files initialized
- [ ] All verification tests passing

## Next: Section 3 (Week 1 Implementation)

Once Section 2 is complete, we'll begin Week 1 implementation:
- Copy prompt templates to `prompts/` directory
- Implement `lib/digest_parser.py`
- Implement `lib/research_generator.py`
- Test daily research workflow

---

**Status:** Ready to execute once Section 1 (Prerequisites) is complete

**Estimated Time:** 30 minutes

**Reference:** See `Deployment Guide.md` for full context
