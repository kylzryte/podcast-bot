---
type: deployment_status
section: Prerequisites (Section 1)
date: 2026-02-25
---

# Podcast Bot Deployment - Prerequisites Check Status

## Current Status: Section 1.1 (System Requirements)

### What We Know

**Mac Mini System Information:**
- **OS:** macOS 15.6.1
- **RAM:** 8 GB (meets minimum requirement)
- **Disk Space:** 7.3 GB available ⚠️ **CRITICAL ISSUE**
  - **Requirement:** 50 GB recommended
  - **Shortfall:** 42.7 GB needed
  - **Impact:** Cannot proceed with full deployment until resolved

### Prerequisites to Verify

The following need to be checked on the Mac Mini (192.168.0.180):

1. ✓ **macOS** - Confirmed 15.6.1
2. ✓ **RAM** - Confirmed 8 GB
3. ⚠️ **Disk Space** - Only 7.3 GB (need 50 GB)
4. ❓ **Python 3.9+** - Not yet verified
5. ❓ **Homebrew** - Not yet verified
6. ❓ **ffmpeg** - Not yet verified
7. ❓ **Tailscale** - Not yet verified
8. ✓ **API Keys** - Exist in ~/news_bot/.env (Perplexity + Anthropic)

## Next Steps to Complete Section 1

### Step 1: Run Prerequisites Check Script

A comprehensive check script has been created at:
`C:\Users\kyle.billings\Desktop\check_prerequisites.sh`

**To run it on Mac Mini:**

```bash
# Option A: Copy and run from Windows
scp "C:\Users\kyle.billings\Desktop\check_prerequisites.sh" kyle@192.168.0.180:/tmp/
ssh kyle@192.168.0.180 "chmod +x /tmp/check_prerequisites.sh && /tmp/check_prerequisites.sh"

# Option B: Run directly on Mac Mini
# 1. Open Terminal on Mac Mini
# 2. Create the script:
nano /tmp/check_prerequisites.sh
# (paste script contents)
# 3. Run it:
chmod +x /tmp/check_prerequisites.sh
/tmp/check_prerequisites.sh
```

This script will check:
- System info (OS, RAM, disk)
- Python installation and version
- Homebrew installation
- ffmpeg installation
- Tailscale installation
- API keys in news_bot/.env

### Step 2: Address Disk Space Issue ⚠️

**Current:** 7.3 GB available
**Required:** 50 GB recommended
**Shortfall:** 42.7 GB

**Potential Solutions:**

1. **Clean up existing files:**
   - Check for old Time Machine backups
   - Clear cache directories
   - Remove unused applications
   - Check Downloads folder

2. **Move news_bot episodes to external storage:**
   ```bash
   # Check what's taking up space
   cd ~
   du -sh */ | sort -h | tail -20

   # Check news_bot storage
   du -sh ~/news_bot/episodes
   ```

3. **Use external drive for podcast episodes:**
   - Store episodes on external USB drive
   - Update `podcast_config.yaml` to point to external storage
   - Requires drive to be always connected

4. **Reduce episode retention:**
   - Store only last 10 episodes instead of 52
   - Archive older episodes to cloud storage
   - Reduces storage from ~35GB to ~7GB

**Recommended approach:** Combination of cleanup + reduced retention initially, then evaluate if external storage needed.

### Step 3: Install Missing Dependencies (Section 1.2)

Once disk space is addressed, install any missing prerequisites:

#### If Homebrew is missing:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### If Python 3.9+ is missing:
```bash
brew install python@3.11
```

#### If ffmpeg is missing:
```bash
brew install ffmpeg
```

#### If Tailscale is missing:
```bash
brew install --cask tailscale
# Then open Tailscale and authenticate
```

### Step 4: Verify API Keys

Confirm the following keys exist in `~/news_bot/.env`:
- `PERPLEXITY_API_KEY` (for host research)
- `ANTHROPIC_API_KEY` (for script generation)
- `OPENAI_API_KEY` (for article selection + TTS)

If OpenAI key is missing, add it:
```bash
echo "OPENAI_API_KEY=your-key-here" >> ~/news_bot/.env
```

## Section 1 Completion Checklist

- [ ] Run prerequisites check script
- [ ] Address disk space issue (free up 43GB)
- [ ] Verify Python 3.9+ installed
- [ ] Verify Homebrew installed
- [ ] Verify ffmpeg installed
- [ ] Verify Tailscale installed
- [ ] Verify all API keys present (Perplexity, Anthropic, OpenAI)
- [ ] Document final system specs

## After Section 1 is Complete

Once all prerequisites are met, we'll move to **Section 2: Initial Setup**, which includes:
- Creating podcast project directory structure
- Setting up Python virtual environment
- Installing Python dependencies
- Creating configuration files
- Testing API connections

**Estimated time for Section 1:** 30-60 minutes (depending on disk cleanup)
**Estimated time for Section 2:** 30 minutes

## Technical Notes

**SSH Connection Issues:**
- Remote execution via SSH from Windows experiencing timeouts
- Workaround: Run commands directly on Mac Mini Terminal
- This is expected with headless Mac Mini setups
- Does not affect automated cron jobs once deployed

**Disk Space Breakdown (Estimated):**
- news_bot: ~2-3 GB (logs + episodes)
- vibe: ~1 GB
- System: ~2 GB available
- **Needed for podcast:**
  - Episodes (52 weeks × 650MB): ~35 GB
  - Research files: ~5 GB
  - Scripts: ~2 GB
  - Temp audio files: ~5 GB
  - Buffer: ~3 GB
  - **Total:** ~50 GB

---

**Current Status:** Blocked on disk space. Must free up 43 GB before proceeding.

**Next Action:** Run prerequisites check script and clean up disk space.

**Reference Documents:**
- Full deployment steps: `Deployment Guide.md`
- Implementation plan: `QUICK START - Implementation Order.md`
- System overview: `README - Start Here.md`
