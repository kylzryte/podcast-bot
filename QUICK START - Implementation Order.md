---
type: quick_start
status: ready_for_implementation
version: 1.0
date: 2026-03-01
---

# Gaming Industry Podcast Bot - Quick Start Guide

## Overview

This document provides the implementation order for deploying the Gaming Industry Podcast Bot system. All detailed documentation has been created - this guide shows you the path from zero to production.

## Prerequisites Checklist

Before starting implementation:
- [ ] Read `PRD - Gaming Industry Podcast Bot.md` (understand system)
- [ ] Read `Implementation Guide.md` (understand architecture)
- [ ] Read `Deployment Guide.md` (understand deployment)
- [ ] Read all 3 Character Guides (understand hosts)
- [ ] Review all 5 Prompt Templates (understand AI interactions)

## Implementation Path (6-8 Weeks)

### Week 1: Core Infrastructure

**Goal:** Set up project structure and basic components

**Tasks:**
1. Create project directory on Mac Mini: `~/podcast/`
2. Set up Python virtual environment
3. Install dependencies from `requirements.txt`
4. Create all subdirectories (episodes, scripts, research, memories, etc.)
5. Configure API keys in `.env` file (secure permissions)
6. Create `podcast_config.yaml` with correct paths
7. Implement `lib/config.py` to load configuration
8. Set up logging infrastructure
9. Test API connections (Perplexity, OpenAI, Claude)

**Deliverable:** Working project structure with API access

**Validation:**
```bash
cd ~/podcast
source venv/bin/activate
python3 -c "from lib.config import load_config; print('Config loaded')"
python3 -c "import openai; print('OpenAI imported')"
```

---

### Week 2: Daily Research Phase

**Goal:** Implement daily article research workflow

**Tasks:**
1. Implement `lib/digest_parser.py`
   - Parse daily digest markdown
   - Extract articles with metadata
   - Handle frontmatter and sections
2. Implement `lib/research_generator.py`
   - Load prompt templates for each host
   - Call Perplexity API with article content
   - Save research files by date/article/host
3. Create main orchestrator: `podcast_bot.py --mode daily-research`
4. Copy prompt templates to `prompts/` directory
5. Test with existing digests (Feb 27, 28, Mar 1)

**Deliverable:** Daily research generates 3 files per article

**Validation:**
```bash
python3 podcast_bot.py --mode daily-research
ls research/$(date +%Y-%m-%d)/
# Should show: article-NNN-nigel-research.md, article-NNN-marcus-research.md, article-NNN-emily-research.md
```

---

### Week 3: Article Selection & Memory

**Goal:** Implement article selection and host memory systems

**Tasks:**
1. Implement `lib/article_selector.py`
   - Collect articles from past 7 days
   - Build selection prompt with all articles
   - Call OpenAI GPT-4 for selection
   - Parse JSON response
   - Load research files for selected articles
2. Implement `lib/memory_manager.py`
   - Load host memory markdown files
   - Parse personality, opinions, predictions
   - Update memories with new content
   - Archive old entries (>6 months)
3. Initialize memory files for all three hosts
4. Test article selection with mock data

**Deliverable:** Article selection returns 6-15 articles with research

**Validation:**
```bash
# Create test with 7 days of mock digests
python3 -c "from lib.article_selector import ArticleSelector; ..."
# Verify: Returns ordered list of articles with memories loaded
```

---

### Week 4: Script Generation

**Goal:** Generate first podcast script

**Tasks:**
1. Implement `lib/script_generator.py`
   - Build massive prompt with articles + research + memories
   - Call Claude Sonnet 4 API
   - Parse script into structured format
   - Extract predictions and notable moments
   - Save script to file
2. Fine-tune script generation prompt based on test output
3. Test with selected articles from week 3
4. **Critical:** Review generated script for:
   - Character consistency
   - Natural dialogue
   - Appropriate length (13k-16k words)
   - Predictions formatted correctly

**Deliverable:** First complete 90-minute script

**Validation:**
```bash
python3 podcast_bot.py --mode weekly-generate
# Should generate: scripts/YYYY-MM-DD - Episode 001 - Script.md
wc -w scripts/*.md  # Check word count
```

**Manual Review Required:** Read script, verify it sounds natural

---

### Week 5: Audio Production

**Goal:** Convert script to audio

**Tasks:**
1. Implement `lib/audio_producer.py`
   - Parse script into dialogue lines + stage directions
   - Call OpenAI TTS for each line
   - Save temporary MP3 files
   - Stitch audio with appropriate pauses
   - Export final MP3
2. Test voice selection for each character
3. Fine-tune pause durations for natural flow
4. Handle stage directions (`[INTERRUPTS]`, `[LAUGHS]`, etc.)
5. Test with short script excerpt first
6. Generate full episode audio

**Deliverable:** First complete 90-minute MP3 episode

**Validation:**
```bash
# After script generation:
ls episodes/*.mp3
afplay episodes/*.mp3  # Listen on Mac
# Verify: ~90 minutes, voices distinct, pauses natural
```

**Manual Review Required:** Listen to episode, check quality

---

### Week 6: RSS Feed & Distribution

**Goal:** Set up podcast feed and HTTP server

**Tasks:**
1. Implement `lib/rss_generator.py`
   - Create RSS 2.0 feed XML
   - Add episode metadata
   - Include audio URL and duration
   - Maintain last 52 episodes
2. Create `server.py` - simple HTTP server
3. Set up LaunchAgent for auto-start
4. Configure Tailscale for remote access
5. Test RSS feed validation
6. Test access from Android device

**Deliverable:** Podcast accessible via RSS on Android

**Validation:**
```bash
# Start server
python3 server.py &
# Check RSS feed
curl http://localhost:8000/feed.xml
# On Android: Add RSS URL to podcast app
```

---

### Week 7: Automation & Scheduling

**Goal:** Full automated workflow

**Tasks:**
1. Configure cron jobs:
   - Daily research: Mon-Sat 12:30pm ET
   - Weekly generation: Sunday 5:00am ET
2. Grant Full Disk Access permissions (macOS)
3. Test automated daily research run
4. Test automated weekly generation run (with mock data)
5. Implement `lib/prediction_tracker.py` for auto-resolution
6. Create health check script
7. Set up log rotation

**Deliverable:** Fully automated podcast system

**Validation:**
```bash
# Check cron jobs
crontab -l

# Simulate daily run
# (Wait for 12:30pm or trigger manually)

# Check logs
tail -f logs/podcast_bot.log
```

---

### Week 8: Character Refinement & Polish

**Goal:** Optimize character voices and script quality

**Tasks:**
1. Listen to first 2-3 episodes
2. Refine character prompts based on output:
   - Is Nigel bombastic enough?
   - Does Marcus make bold predictions?
   - Is Emily optimistic without being annoying?
3. Adjust TTS settings if needed (voice, speed, tone)
4. Fine-tune script generation prompt:
   - More/less interruptions
   - Better natural flow
   - Stronger character voices
5. Test prediction tracking accuracy
6. Optimize token usage for cost reduction
7. Document lessons learned

**Deliverable:** Production-ready system with refined characters

**Validation:**
- Listen to full episode
- Verify characters feel consistent
- Check prediction tracking works
- Review monthly costs
- Confirm automated runs successful

---

## Production Deployment

Once testing is complete:

### Pre-Launch Checklist

- [ ] All tests passing
- [ ] Characters sound natural and distinct
- [ ] Episode length target met (70-110 min)
- [ ] Automated cron jobs working
- [ ] HTTP server accessible via Tailscale
- [ ] RSS feed validates
- [ ] Android playback works
- [ ] Logs show no errors
- [ ] Cost tracking functional

### Go-Live Steps

1. **First Week:** Daily research runs automatically Mon-Sat
2. **First Sunday:** Weekly generation at 5am ET
3. **Sunday Morning:** Listen to Episode 001
4. **Sunday Afternoon:** Review quality, note improvements needed
5. **Week 2:** Monitor daily, make adjustments as needed
6. **Week 3-4:** System stabilizes, make final refinements

### Post-Launch Monitoring

**Daily:**
- Check logs for errors
- Verify research files generated

**Weekly:**
- Listen to new episode
- Review character consistency
- Check prediction accuracy
- Monitor costs

**Monthly:**
- Generate cost report
- Archive old episodes (>1 year)
- Update host memories if needed
- Review system performance

---

## File Reference

### Documentation
- `PRD - Gaming Industry Podcast Bot.md` - Full product requirements
- `Implementation Guide.md` - Technical implementation details
- `Deployment Guide.md` - Mac Mini deployment instructions

### Character Guides
- `Character Guides/Nigel Thornbury - Character Guide.md`
- `Character Guides/Marcus Webb - Character Guide.md`
- `Character Guides/Emily Bradford - Character Guide.md`

### Prompt Templates
- `Prompt Templates/Research Prompt - Nigel.md`
- `Prompt Templates/Research Prompt - Marcus.md`
- `Prompt Templates/Research Prompt - Emily.md`
- `Prompt Templates/Article Selection Prompt.md`
- `Prompt Templates/Script Generation Prompt.md`

### Code Structure
```
~/podcast/
├── podcast_bot.py              # Main orchestrator
├── podcast_config.yaml         # Configuration
├── lib/
│   ├── digest_parser.py        # Week 2
│   ├── research_generator.py   # Week 2
│   ├── article_selector.py     # Week 3
│   ├── memory_manager.py       # Week 3
│   ├── script_generator.py     # Week 4
│   ├── audio_producer.py       # Week 5
│   ├── rss_generator.py        # Week 6
│   └── prediction_tracker.py   # Week 7
└── server.py                   # Week 6
```

---

## Getting Help

**If stuck:**
1. Check relevant documentation section
2. Review prompt templates
3. Check logs: `tail -f logs/podcast_bot.log`
4. Verify API keys and configuration
5. Test individual components in isolation

**Common Issues:**
- API authentication → Check `.env` file permissions (chmod 600)
- Parsing errors → Review digest structure
- Audio stitching → Verify ffmpeg installation
- Cron not running → Check Full Disk Access permissions

---

## Next Steps

1. Set up development environment (Week 1 tasks)
2. Begin implementation following week-by-week plan
3. Test thoroughly at each stage
4. Document any deviations or improvements
5. Deploy to production when ready

**Estimated Timeline:** 6-8 weeks to production-ready system

**Ready to Start?** Begin with Week 1: Core Infrastructure

---

**Status:** 📋 All documentation complete, ready for implementation

[[Podcast Bot]] | [[Implementation]] | [[Quick Start]]
