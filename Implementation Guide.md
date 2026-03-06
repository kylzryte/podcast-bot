---
type: implementation_guide
status: draft
version: 1.0
date: 2026-03-01
related: PRD - Gaming Industry Podcast Bot
---

# Gaming Industry Podcast Bot - Implementation Guide

## Table of Contents
1. [System Overview](#system-overview)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [Implementation Details](#implementation-details)
5. [Testing Strategy](#testing-strategy)
6. [Monitoring & Debugging](#monitoring--debugging)

---

## 1. System Overview

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     DAILY PHASE (Mon-Sat)                   │
│                                                              │
│  12:00 PM ET: news_bot.py generates digest                  │
│       ↓                                                      │
│  12:30 PM ET: podcast_bot.py --mode daily-research          │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Digest Parser                                       │   │
│  │  • Load daily digest markdown                        │   │
│  │  • Extract articles with metadata                    │   │
│  │  • Validate article structure                        │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Per-Host Research (3x Perplexity queries/article)  │   │
│  │  • Nigel: Historical precedents & patterns          │   │
│  │  • Marcus: Business models & predictions            │   │
│  │  • Emily: Community sentiment & player impact       │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  Save: research/YYYY-MM-DD/article-NNN-HOST-research.md    │
│                                                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WEEKLY PHASE (Sunday)                     │
│                                                              │
│  5:00 AM ET: podcast_bot.py --mode weekly-generate          │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Article Aggregator                                  │   │
│  │  • Collect 7 days of digests (Sat-Sat)             │   │
│  │  • Load all research files (21 per article)         │   │
│  │  • Build comprehensive article pool                 │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Article Selection (OpenAI GPT-4)                   │   │
│  │  • Simulate host voting on articles                 │   │
│  │  • Score: impact, diversity, discussion potential   │   │
│  │  • Select 6-15 articles with optimal order          │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Memory Loader                                       │   │
│  │  • Load Nigel-Memory.md                             │   │
│  │  • Load Marcus-Memory.md                            │   │
│  │  • Load Emily-Memory.md                             │   │
│  │  • Extract personality, opinions, predictions       │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Script Generation (Claude Sonnet 4)                │   │
│  │  • Inject: articles + research + memories           │   │
│  │  • Generate 14,000-word natural dialogue            │   │
│  │  • Include: debate, predictions, callbacks          │   │
│  │  • Parse: speakers, stage directions, content       │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Memory Updater                                      │   │
│  │  • Extract new opinions/predictions from script     │   │
│  │  • Append to host memory files                      │   │
│  │  • Archive entries >6 months old                    │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Audio Production (OpenAI TTS)                      │   │
│  │  • Parse script into dialogue lines                 │   │
│  │  • Generate audio per line (character voice)        │   │
│  │  • Add pauses based on stage directions             │   │
│  │  • Stitch into final MP3 (~90 minutes)             │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  RSS Feed Generator                                  │   │
│  │  • Update feed.xml with new episode                 │   │
│  │  • Maintain last 52 episodes in feed                │   │
│  │  • Generate episode metadata                         │   │
│  └─────────────────────────────────────────────────────┘   │
│       ↓                                                      │
│  Output: ~/podcast/episodes/YYYY-MM-DD - Episode NNN.mp3   │
│                                                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 CONTINUOUS (Always Running)                  │
│                                                              │
│  HTTP Server (port 8000)                                     │
│  • Serve feed.xml                                           │
│  • Serve episode MP3 files                                  │
│  • Accessible via Tailscale on Android                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Project Structure

```
~/podcast/
├── podcast_bot.py                 # Main orchestrator
├── podcast_config.yaml            # Configuration
├── requirements.txt               # Python dependencies
├── .env                          # API keys (secure)
│
├── lib/                          # Core library modules
│   ├── __init__.py
│   ├── digest_parser.py          # Parse daily digests
│   ├── research_generator.py     # Perplexity research prompts
│   ├── article_selector.py       # OpenAI article selection
│   ├── memory_manager.py         # Host memory CRUD
│   ├── script_generator.py       # Claude script generation
│   ├── audio_producer.py         # OpenAI TTS + stitching
│   ├── rss_generator.py          # RSS feed management
│   └── prediction_tracker.py     # Automated prediction resolution
│
├── prompts/                      # Prompt templates
│   ├── research_nigel.md
│   ├── research_marcus.md
│   ├── research_emily.md
│   ├── article_selection.md
│   └── script_generation.md
│
├── episodes/                     # Generated podcast audio
│   ├── 2026-03-02 - Gaming Industry Podcast - Episode 001.mp3
│   └── ...
│
├── scripts/                      # Generated scripts
│   ├── 2026-03-02 - Episode 001 - Script.md
│   └── ...
│
├── research/                     # Daily research files
│   ├── 2026-02-24/
│   │   ├── article-001-nigel-research.md
│   │   ├── article-001-marcus-research.md
│   │   ├── article-001-emily-research.md
│   │   └── ...
│   └── ...
│
├── memories/                     # Host memory files
│   ├── Nigel-Memory.md
│   ├── Marcus-Memory.md
│   ├── Emily-Memory.md
│   └── Archive/
│       └── ...
│
├── feed.xml                      # RSS feed
├── server.py                     # Simple HTTP server
├── logs/
│   └── podcast_bot.log
└── temp/                         # Temporary TTS files
    └── (cleaned after stitching)
```

---

## 3. Core Components

### 3.1 Main Orchestrator (`podcast_bot.py`)

```python
#!/usr/bin/env python3
"""
Gaming Industry Podcast Bot
Main orchestrator for daily research and weekly podcast generation
"""

import argparse
import sys
from pathlib import Path
from datetime import datetime, timedelta
import logging

from lib.digest_parser import DigestParser
from lib.research_generator import ResearchGenerator
from lib.article_selector import ArticleSelector
from lib.memory_manager import MemoryManager
from lib.script_generator import ScriptGenerator
from lib.audio_producer import AudioProducer
from lib.rss_generator import RSSGenerator
from lib.prediction_tracker import PredictionTracker
from lib.config import load_config

def setup_logging(config):
    """Configure logging"""
    log_config = config['logging']
    logging.basicConfig(
        level=getattr(logging, log_config['level']),
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler(log_config['file']),
            logging.StreamHandler()
        ]
    )

def daily_research(config):
    """
    Daily research phase (Mon-Sat, 12:30pm ET)
    Parse today's digest and generate per-host research
    """
    logger = logging.getLogger('daily_research')
    logger.info("Starting daily research phase")

    try:
        # Load today's digest
        today = datetime.now().strftime("%Y-%m-%d")
        digest_path = Path(config['paths']['digests']) / f"{today} - Gaming Industry Digest.md"

        if not digest_path.exists():
            logger.error(f"Digest not found: {digest_path}")
            return 1

        # Parse digest
        parser = DigestParser(config)
        articles = parser.parse(digest_path)
        logger.info(f"Parsed {len(articles)} articles from digest")

        # Generate research for each article, each host
        research_gen = ResearchGenerator(config)
        hosts = ['Nigel', 'Marcus', 'Emily']

        for article in articles:
            logger.info(f"Researching article: {article['headline']}")
            for host in hosts:
                try:
                    research = research_gen.generate(article, host)
                    research_gen.save(today, article['id'], host, research)
                    logger.info(f"  {host}: Research saved")
                except Exception as e:
                    logger.error(f"  {host}: Research failed - {e}")
                    continue

        logger.info("Daily research phase complete")
        return 0

    except Exception as e:
        logger.error(f"Daily research phase failed: {e}", exc_info=True)
        return 1

def weekly_generate(config):
    """
    Weekly generation phase (Sunday 5am ET)
    Generate podcast episode from past week's research
    """
    logger = logging.getLogger('weekly_generate')
    logger.info("Starting weekly generation phase")

    try:
        # Determine date range (Sat-Sat)
        today = datetime.now()
        end_date = today - timedelta(days=today.weekday() + 2)  # Last Saturday
        start_date = end_date - timedelta(days=6)

        logger.info(f"Episode date range: {start_date.date()} to {end_date.date()}")

        # Collect articles from past week
        selector = ArticleSelector(config)
        articles = selector.collect_week_articles(start_date, end_date)
        logger.info(f"Collected {len(articles)} articles from past week")

        # Select articles for episode (OpenAI)
        selected = selector.select_articles(articles, start_date, end_date)
        logger.info(f"Selected {len(selected)} articles for episode")

        # Load host memories
        memory_mgr = MemoryManager(config)
        memories = {
            'Nigel': memory_mgr.load('Nigel'),
            'Marcus': memory_mgr.load('Marcus'),
            'Emily': memory_mgr.load('Emily')
        }
        logger.info("Host memories loaded")

        # Generate script (Claude)
        script_gen = ScriptGenerator(config)
        episode_num = script_gen.get_next_episode_number()
        script = script_gen.generate(selected, memories, episode_num, start_date, end_date)
        script_path = script_gen.save(script, episode_num)
        logger.info(f"Script generated: {script_path}")

        # Update memories with new content from script
        memory_mgr.update_from_script(script)
        logger.info("Host memories updated")

        # Generate audio (OpenAI TTS)
        audio_producer = AudioProducer(config)
        audio_path = audio_producer.produce(script, episode_num)
        logger.info(f"Audio produced: {audio_path}")

        # Update RSS feed
        rss_gen = RSSGenerator(config)
        rss_gen.add_episode(episode_num, audio_path, start_date, end_date, selected)
        logger.info("RSS feed updated")

        # Track predictions
        tracker = PredictionTracker(config)
        tracker.scan_and_resolve()
        logger.info("Predictions scanned for resolution")

        logger.info(f"Weekly generation complete - Episode {episode_num}")
        return 0

    except Exception as e:
        logger.error(f"Weekly generation failed: {e}", exc_info=True)
        return 1

def main():
    """Main entry point"""
    parser = argparse.ArgumentParser(description='Gaming Industry Podcast Bot')
    parser.add_argument('--mode', choices=['daily-research', 'weekly-generate'],
                       required=True, help='Execution mode')
    parser.add_argument('--config', default='podcast_config.yaml',
                       help='Configuration file path')

    args = parser.parse_args()

    # Load configuration
    config = load_config(args.config)
    setup_logging(config)

    # Execute mode
    if args.mode == 'daily-research':
        return daily_research(config)
    elif args.mode == 'weekly-generate':
        return weekly_generate(config)
    else:
        print(f"Unknown mode: {args.mode}")
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

### 3.2 Digest Parser (`lib/digest_parser.py`)

```python
"""
Digest Parser
Parses daily Gaming Industry Digest markdown files
"""

import re
from pathlib import Path
import yaml

class DigestParser:
    def __init__(self, config):
        self.config = config

    def parse(self, digest_path):
        """
        Parse digest markdown file and extract articles

        Returns: List of article dicts with:
        - id: Unique identifier (generated)
        - headline: Article headline
        - category: Business category
        - impact_level: HIGH/MEDIUM/LOW
        - summary: Executive summary
        - key_takeaways: List of bullet points
        - implications: Strategic implications
        - sources: List of source URLs
        - date: Article date
        """
        with open(digest_path, 'r', encoding='utf-8') as f:
            content = f.read()

        articles = []

        # Extract frontmatter
        frontmatter_match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
        if frontmatter_match:
            frontmatter = yaml.safe_load(frontmatter_match.group(1))
            digest_date = frontmatter.get('date', '')

        # Split by article sections
        # Look for pattern: ### [Headline]
        article_sections = re.split(r'\n### ', content)

        article_id = 1
        for section in article_sections[1:]:  # Skip first split (frontmatter)
            try:
                article = self._parse_article_section(section, article_id, digest_date)
                if article:
                    articles.append(article)
                    article_id += 1
            except Exception as e:
                print(f"Warning: Failed to parse article section: {e}")
                continue

        return articles

    def _parse_article_section(self, section, article_id, digest_date):
        """Parse individual article section"""
        lines = section.split('\n')
        headline = lines[0].strip()

        # Extract metadata
        category = self._extract_field(section, 'Category')
        impact_level = self._determine_impact_level(section)

        # Extract content sections
        summary = self._extract_section(section, 'Executive Summary')
        key_takeaways = self._extract_bullets(section, 'Key Takeaways')
        implications = self._extract_section(section, 'Strategic Implications')
        sources = self._extract_sources(section)

        return {
            'id': f"article-{article_id:03d}",
            'headline': headline,
            'category': category,
            'impact_level': impact_level,
            'summary': summary,
            'key_takeaways': key_takeaways,
            'implications': implications,
            'sources': sources,
            'date': digest_date,
            'full_content': section  # Store for research context
        }

    def _extract_field(self, text, field_name):
        """Extract single-line field"""
        pattern = rf'{field_name}:\s*(.+?)(?:\n|$)'
        match = re.search(pattern, text)
        return match.group(1).strip() if match else ''

    def _determine_impact_level(self, text):
        """Determine impact level from section position or explicit markers"""
        # Articles under "HIGH IMPACT DEVELOPMENTS" header are HIGH
        # Can be refined based on actual digest structure
        if 'HIGH IMPACT' in text or text.startswith('HIGH'):
            return 'HIGH'
        elif 'MEDIUM IMPACT' in text or text.startswith('MEDIUM'):
            return 'MEDIUM'
        else:
            return 'LOW'

    def _extract_section(self, text, section_name):
        """Extract multi-line section content"""
        pattern = rf'{section_name}\s*\n(.*?)(?=\n\n[A-Z]|\Z)'
        match = re.search(pattern, text, re.DOTALL)
        return match.group(1).strip() if match else ''

    def _extract_bullets(self, text, section_name):
        """Extract bullet point list"""
        section = self._extract_section(text, section_name)
        if not section:
            return []

        bullets = re.findall(r'^[-•]\s*(.+?)$', section, re.MULTILINE)
        return [b.strip() for b in bullets]

    def _extract_sources(self, text):
        """Extract source URLs"""
        # Look for URLs in text
        urls = re.findall(r'https?://[^\s\)]+', text)
        return list(set(urls))  # Deduplicate
```

### 3.3 Research Generator (`lib/research_generator.py`)

```python
"""
Research Generator
Generates character-specific research using Perplexity API
"""

import os
import requests
from pathlib import Path
from datetime import datetime

class ResearchGenerator:
    def __init__(self, config):
        self.config = config
        self.api_key = os.environ.get(config['apis']['perplexity']['api_key_env'])
        self.model = config['apis']['perplexity']['model']
        self.base_path = Path(config['paths']['base']) / config['paths']['research']

        # Load prompt templates
        self.prompts = {
            'Nigel': self._load_prompt('research_nigel.md'),
            'Marcus': self._load_prompt('research_marcus.md'),
            'Emily': self._load_prompt('research_emily.md')
        }

    def _load_prompt(self, filename):
        """Load prompt template from file"""
        prompt_path = Path(self.config['paths']['prompts']) / filename
        with open(prompt_path, 'r') as f:
            return f.read()

    def generate(self, article, host_name):
        """
        Generate research for article from host's perspective

        Args:
            article: Article dict from digest parser
            host_name: 'Nigel', 'Marcus', or 'Emily'

        Returns: Research text (markdown)
        """
        # Build prompt with article content
        prompt = self.prompts[host_name]
        prompt = prompt.replace('{{HEADLINE}}', article['headline'])
        prompt = prompt.replace('{{SUMMARY}}', article['summary'])
        prompt = prompt.replace('{{KEY_TAKEAWAYS}}', '\n'.join(f"- {t}" for t in article['key_takeaways']))
        prompt = prompt.replace('{{IMPLICATIONS}}', article['implications'])
        prompt = prompt.replace('{{FULL_CONTENT}}', article['full_content'])

        # Call Perplexity API
        response = requests.post(
            'https://api.perplexity.ai/chat/completions',
            headers={
                'Authorization': f'Bearer {self.api_key}',
                'Content-Type': 'application/json'
            },
            json={
                'model': self.model,
                'messages': [
                    {'role': 'system', 'content': 'You are a gaming industry analyst conducting research.'},
                    {'role': 'user', 'content': prompt}
                ],
                'max_tokens': self.config['apis']['perplexity']['max_tokens'],
                'temperature': self.config['apis']['perplexity']['temperature']
            },
            timeout=60
        )

        if response.status_code != 200:
            raise Exception(f"Perplexity API error: {response.status_code} - {response.text}")

        result = response.json()
        research_text = result['choices'][0]['message']['content']

        # Log token usage
        usage = result.get('usage', {})
        print(f"  Tokens: {usage.get('total_tokens', 0)}")

        return research_text

    def save(self, date_str, article_id, host_name, research_text):
        """Save research to file"""
        date_dir = self.base_path / date_str
        date_dir.mkdir(parents=True, exist_ok=True)

        filename = f"{article_id}-{host_name.lower()}-research.md"
        filepath = date_dir / filename

        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(f"# Research: {host_name} on {article_id}\n\n")
            f.write(f"**Date:** {date_str}\n")
            f.write(f"**Host:** {host_name}\n\n")
            f.write("---\n\n")
            f.write(research_text)

        return filepath
```

### 3.4 Article Selector (`lib/article_selector.py`)

```python
"""
Article Selector
Uses OpenAI to select and order articles for episode
"""

import os
import openai
from pathlib import Path
from datetime import datetime, timedelta
import json

class ArticleSelector:
    def __init__(self, config):
        self.config = config
        openai.api_key = os.environ.get(config['apis']['openai']['api_key_env'])
        self.model = config['apis']['openai']['model']
        self.digest_path = Path(config['paths']['digests'])
        self.research_path = Path(config['paths']['base']) / config['paths']['research']

    def collect_week_articles(self, start_date, end_date):
        """Collect all articles from digests in date range"""
        from lib.digest_parser import DigestParser

        parser = DigestParser(self.config)
        all_articles = []

        current_date = start_date
        while current_date <= end_date:
            date_str = current_date.strftime("%Y-%m-%d")
            digest_file = self.digest_path / f"{date_str} - Gaming Industry Digest.md"

            if digest_file.exists():
                articles = parser.parse(digest_file)
                # Add digest date to each article
                for article in articles:
                    article['digest_date'] = date_str
                all_articles.extend(articles)

            current_date += timedelta(days=1)

        return all_articles

    def select_articles(self, articles, start_date, end_date):
        """
        Use OpenAI to select and order articles for episode

        Returns: List of selected article dicts with order
        """
        # Build selection prompt
        prompt = self._build_selection_prompt(articles, start_date, end_date)

        # Call OpenAI
        response = openai.chat.completions.create(
            model=self.model,
            messages=[
                {'role': 'system', 'content': 'You are a podcast producer selecting articles for discussion.'},
                {'role': 'user', 'content': prompt}
            ],
            max_tokens=self.config['apis']['openai']['max_tokens'],
            temperature=self.config['apis']['openai']['temperature']
        )

        result_text = response.choices[0].message.content

        # Parse selection result (expects JSON with article IDs and order)
        selected_ids = self._parse_selection_result(result_text)

        # Return articles in selected order
        article_map = {a['id']: a for a in articles}
        selected = [article_map[aid] for aid in selected_ids if aid in article_map]

        # Load research for selected articles
        for article in selected:
            article['research'] = self._load_article_research(article)

        return selected

    def _build_selection_prompt(self, articles, start_date, end_date):
        """Build article selection prompt"""
        # Summarize articles
        article_list = []
        for i, article in enumerate(articles, 1):
            article_list.append(
                f"{i}. [{article['impact_level']}] {article['headline']}\n"
                f"   Category: {article['category']}\n"
                f"   Date: {article['digest_date']}\n"
                f"   Summary: {article['summary'][:200]}...\n"
            )

        articles_text = "\n".join(article_list)

        # Load selection prompt template
        template_path = Path(self.config['paths']['prompts']) / 'article_selection.md'
        with open(template_path, 'r') as f:
            template = f.read()

        prompt = template.replace('{{ARTICLE_LIST}}', articles_text)
        prompt = prompt.replace('{{START_DATE}}', start_date.strftime("%Y-%m-%d"))
        prompt = prompt.replace('{{END_DATE}}', end_date.strftime("%Y-%m-%d"))
        prompt = prompt.replace('{{ARTICLE_COUNT}}', str(len(articles)))

        return prompt

    def _parse_selection_result(self, result_text):
        """Parse OpenAI result to extract ordered article IDs"""
        try:
            # Expect JSON response with article IDs
            result = json.loads(result_text)
            if isinstance(result, list):
                return result
            elif 'selected_articles' in result:
                return result['selected_articles']
            else:
                raise ValueError("Unexpected JSON structure")
        except:
            # Fallback: extract article-XXX patterns
            import re
            ids = re.findall(r'article-\d{3}', result_text)
            return ids

    def _load_article_research(self, article):
        """Load all research files for an article"""
        date_dir = self.research_path / article['digest_date']
        research = {}

        for host in ['nigel', 'marcus', 'emily']:
            research_file = date_dir / f"{article['id']}-{host}-research.md"
            if research_file.exists():
                with open(research_file, 'r', encoding='utf-8') as f:
                    research[host.capitalize()] = f.read()

        return research
```

*[Continue with remaining core components...]*

---

## 4. Implementation Details

### 4.1 Memory Manager

**Key Functions:**
- `load(host_name)` - Load host memory file
- `update_from_script(script)` - Extract and append new content
- `archive_old_entries()` - Move 6+ month old entries to archive
- `get_prediction_accuracy(host_name)` - Calculate prediction % correct

**Memory File Format:**
```markdown
# [Host Name] Memory

**Last Updated:** YYYY-MM-DD

## Canonical Personality

- [Core personality trait 1]
- [Core personality trait 2]
- [Evolving characteristics]

## Recent Opinions (Last 6 Months)

### 2026-03-02 - Episode 001
- **Opinion:** Microsoft's AI-focused CEO hire will backfire
  - **Context:** Discussion of Asha Sharma appointment
  - **Rationale:** Gaming needs gaming-native leadership

## Active Predictions

### Prediction #001 (2026-03-02)
- **Statement:** "Switch 2 will outsell PS5 by Q4 2026"
- **Confidence:** High
- **Made by:** Marcus
- **Status:** ACTIVE

## Resolved Predictions

### Prediction #042 (2025-12-15)
- **Statement:** "Game Pass will hit 40M subscribers by March"
- **Resolution:** CORRECT (announced 41M on 2026-03-01)
- **Accuracy Impact:** +1 correct

## Running Jokes

- "Bloody hell, not another battle royale" (Nigel's dismissiveness)
- "I'm calling it now..." (Marcus's overconfidence marker)
- "But the community loves..." (Emily's optimism trigger)

## Prediction Accuracy

- **Total Predictions:** 12
- **Resolved:** 8
- **Correct:** 5
- **Incorrect:** 3
- **Accuracy:** 62.5%
```

### 4.2 Script Generator

**Prompt Structure:**

1. **System Context** (500 words)
   - Podcast format description
   - Host personalities (detailed)
   - Conversation style guidelines

2. **Host Memories** (2,000 words per host = 6,000 words)
   - Full memory files for continuity

3. **Selected Articles** (1,500 words per article × 10 = 15,000 words)
   - Article content + research from all 3 hosts

4. **Episode Instructions** (1,000 words)
   - Article order
   - Required elements (callbacks, predictions)
   - Target length

**Total Context:** ~22,500 words = ~30,000 tokens

**Output Parsing:**
```python
def parse_script(script_text):
    """
    Parse script into structured format

    Returns:
        {
            'lines': [
                {'speaker': 'NIGEL', 'text': '...', 'stage_direction': None},
                {'speaker': 'MARCUS', 'text': '...', 'stage_direction': 'INTERRUPTS'},
                ...
            ],
            'predictions': [
                {'host': 'Marcus', 'statement': '...', 'episode': 1},
                ...
            ],
            'word_count': 14523
        }
    """
    lines = []
    predictions = []

    # Parse dialogue
    dialogue_pattern = r'^([A-Z]+):\s*(?:\[([A-Z\s]+)\])?\s*(.+)$'
    for line in script_text.split('\n'):
        match = re.match(dialogue_pattern, line)
        if match:
            speaker, stage_dir, text = match.groups()
            lines.append({
                'speaker': speaker,
                'text': text,
                'stage_direction': stage_dir
            })

    # Extract predictions
    for line in lines:
        if 'predict' in line['text'].lower() or 'calling it now' in line['text'].lower():
            # Enhanced extraction logic
            predictions.append({
                'host': line['speaker'],
                'statement': extract_prediction_statement(line['text']),
                'episode': episode_num
            })

    return {
        'lines': lines,
        'predictions': predictions,
        'word_count': len(script_text.split())
    }
```

### 4.3 Audio Producer

**TTS Processing:**
```python
def produce_line_audio(text, speaker, voice_id):
    """Generate audio for single dialogue line"""
    response = openai.audio.speech.create(
        model="tts-1",
        voice=voice_id,
        input=text,
        speed=1.0  # Adjust per character if needed
    )

    # Save to temp file
    temp_file = f"temp/line_{timestamp}_{speaker}.mp3"
    response.stream_to_file(temp_file)

    return temp_file
```

**Audio Stitching:**
```python
from pydub import AudioSegment

def stitch_audio(audio_files, stage_directions):
    """Combine audio files with appropriate pauses"""
    combined = AudioSegment.empty()

    for i, (audio_file, direction) in enumerate(zip(audio_files, stage_directions)):
        # Load audio segment
        segment = AudioSegment.from_mp3(audio_file)
        combined += segment

        # Add pause based on stage direction
        if direction == 'INTERRUPTS':
            pause = 100  # 0.1s
        elif direction == 'OVERLAPS':
            pause = 0
        elif direction == 'LAUGHS':
            pause = 500  # 0.5s
        else:
            pause = 500  # Default between speakers

        combined += AudioSegment.silent(duration=pause)

    # Export final
    combined.export(output_path, format="mp3", bitrate="128k")

    return output_path
```

---

## 5. Testing Strategy

### 5.1 Unit Tests

```python
# tests/test_digest_parser.py
def test_parse_single_article():
    parser = DigestParser(config)
    articles = parser.parse('fixtures/sample_digest.md')

    assert len(articles) > 0
    assert 'headline' in articles[0]
    assert 'impact_level' in articles[0]

# tests/test_memory_manager.py
def test_load_memory():
    mgr = MemoryManager(config)
    memory = mgr.load('Nigel')

    assert 'personality' in memory
    assert 'predictions' in memory

# tests/test_script_parser.py
def test_parse_dialogue():
    script = "NIGEL: This is terrible.\nMARCUS: [INTERRUPTS] I disagree!"
    parsed = parse_script(script)

    assert len(parsed['lines']) == 2
    assert parsed['lines'][1]['stage_direction'] == 'INTERRUPTS'
```

### 5.2 Integration Tests

```python
# tests/test_daily_workflow.py
def test_full_daily_research():
    """Test complete daily research workflow"""
    # Setup: Create mock digest
    # Execute: Run daily research
    # Verify: Research files created for all hosts
    pass

# tests/test_weekly_workflow.py
def test_full_weekly_generation():
    """Test complete weekly generation workflow"""
    # Setup: Create week of digests + research
    # Execute: Run weekly generation
    # Verify: Script generated, audio produced, RSS updated
    pass
```

### 5.3 Manual Testing Checklist

**Daily Research:**
- [ ] Digest parsing handles all article formats
- [ ] Research prompts inject article content correctly
- [ ] Perplexity API calls succeed with proper error handling
- [ ] Research files saved with correct naming
- [ ] Token usage logged accurately

**Weekly Generation:**
- [ ] Article collection spans full week (Sat-Sat)
- [ ] Article selection returns 6-15 articles
- [ ] Host memories load correctly
- [ ] Script generation produces natural dialogue
- [ ] Character personalities consistent
- [ ] Predictions extracted correctly
- [ ] Audio stitching produces smooth playback
- [ ] RSS feed validates

**Character Quality:**
- [ ] Nigel: Bombastic, curses, interrupts, references history
- [ ] Marcus: Overconfident, makes predictions, uses buzzwords
- [ ] Emily: Optimistic, cites community, defers to experience
- [ ] Conversations feel natural (not robotic)
- [ ] Interruptions and cross-talk work
- [ ] Running jokes appear organically

---

## 6. Monitoring & Debugging

### 6.1 Logging

**Log Levels:**
- **DEBUG:** API request/response details, token counts
- **INFO:** Phase completions, file operations
- **WARNING:** Retries, partial failures
- **ERROR:** API failures, missing files

**Example Log Output:**
```
2026-03-01 12:30:15 - daily_research - INFO - Starting daily research phase
2026-03-01 12:30:16 - digest_parser - INFO - Parsed 18 articles from digest
2026-03-01 12:30:20 - research_generator - INFO - Researching article: Microsoft Leadership Overhaul
2026-03-01 12:30:22 - research_generator - INFO -   Nigel: Research saved (2,234 tokens)
2026-03-01 12:30:25 - research_generator - INFO -   Marcus: Research saved (1,987 tokens)
2026-03-01 12:30:28 - research_generator - INFO -   Emily: Research saved (2,103 tokens)
2026-03-01 12:45:32 - daily_research - INFO - Daily research phase complete
```

### 6.2 Debugging Tools

**Script Validator:**
```python
def validate_script(script_text):
    """Check script for common issues"""
    issues = []

    # Check length
    word_count = len(script_text.split())
    if word_count < 10000:
        issues.append(f"Script too short: {word_count} words")
    elif word_count > 20000:
        issues.append(f"Script too long: {word_count} words")

    # Check speaker distribution
    lines = parse_script(script_text)['lines']
    speaker_counts = {}
    for line in lines:
        speaker_counts[line['speaker']] = speaker_counts.get(line['speaker'], 0) + 1

    # Each host should speak roughly equally (±20%)
    avg = sum(speaker_counts.values()) / len(speaker_counts)
    for speaker, count in speaker_counts.items():
        if count < avg * 0.8 or count > avg * 1.2:
            issues.append(f"{speaker} speaks {count} times (avg: {avg:.0f})")

    return issues
```

**Audio Validator:**
```python
def validate_audio(audio_path):
    """Check audio file quality"""
    audio = AudioSegment.from_mp3(audio_path)

    duration_minutes = len(audio) / 1000 / 60

    issues = []
    if duration_minutes < 70:
        issues.append(f"Episode too short: {duration_minutes:.1f} min")
    elif duration_minutes > 110:
        issues.append(f"Episode too long: {duration_minutes:.1f} min")

    # Check for silence gaps (indicates stitching issues)
    # ... detection logic ...

    return issues
```

### 6.3 Cost Tracking

**Daily Cost Logger:**
```python
def log_costs(date, phase, api, tokens, cost):
    """Log API usage costs"""
    with open('logs/costs.csv', 'a') as f:
        f.write(f"{date},{phase},{api},{tokens},{cost}\n")
```

**Monthly Cost Reporter:**
```bash
# Generate monthly cost report
python3 -m lib.cost_reporter --month 2026-03
```

---

## Next Steps

1. Review implementation plan
2. Approve technical approach
3. Begin Phase 1: Core infrastructure
4. Set up development environment on Mac Mini

See `Deployment Guide.md` for Mac Mini setup instructions.
