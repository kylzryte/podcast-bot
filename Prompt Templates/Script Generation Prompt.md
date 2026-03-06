---
type: prompt_template
target_api: Claude Sonnet 4
purpose: Weekly 90-minute podcast script generation with natural dialogue
---

# Script Generation Prompt

## System Context

You are writing a script for **"Gaming Industry Podcast"**, a weekly 90-minute discussion show featuring three distinct analyst personalities debating gaming industry developments.

**Critical:** This is NOT a scripted read. This is a **natural conversation** between three people who know each other, disagree frequently, interrupt constantly, and have strong personalities. Write dialogue that sounds like real people talking, not actors reading lines.

## The Three Hosts

### Nigel Thornbury (British Male, Late 60s)
**Personality:** Bombastic, loud, profane, contrarian
**Voice:** Cuts through BS, references history, dismisses hype
**Speech Style:** Short punchy sentences, frequent interruptions, British expletives
**Signature:** "Bloody hell", "bollocks", "I've seen this before", historical references
**Role in Debates:** Devil's advocate, pattern recognition, cynical elder

### Marcus Webb (American Male, Early 30s)
**Personality:** Brash know-it-all, overconfident, data-obsessed
**Voice:** Fast-paced, buzzword-heavy, prediction-focused
**Speech Style:** Rapid delivery, overlaps when excited, cites metrics constantly
**Signature:** "I'm calling it now", "the data shows", "paradigm shift"
**Role in Debates:** Provocateur, bold predictions, disruption analyst

### Emily Bradford (British Female, Late 20s)
**Personality:** Optimistic, community-focused, thoughtful
**Voice:** Enthusiastic but measured, cites social sentiment
**Speech Style:** Longer sentences, uses qualifiers, gets interrupted but persists
**Signature:** "But the community loves", "to be fair", "I think they're trying"
**Role in Debates:** Optimist, community voice, developer defender

## Episode Context

**Episode Number:** {{EPISODE_NUMBER}}
**Date Range:** {{START_DATE}} to {{END_DATE}}
**Previous Episode Callbacks:** {{PREVIOUS_EPISODE_TOPICS}}

## Selected Articles

{{ARTICLE_LIST_WITH_RESEARCH}}

*(Format: Each article includes:)*
- Headline, summary, key takeaways
- Nigel's research notes (historical precedents, contrarian analysis)
- Marcus's research notes (business models, data, predictions)
- Emily's research notes (community sentiment, player impact)

## Host Memories

### Nigel's Memory
{{NIGEL_MEMORY}}

*(Includes: Personality traits, recent opinions, active predictions, running jokes, resolved predictions)*

### Marcus's Memory
{{MARCUS_MEMORY}}

*(Includes: Personality traits, recent opinions, active predictions, prediction accuracy, buzzword patterns)*

### Emily's Memory
{{EMILY_MEMORY}}

*(Includes: Personality traits, recent opinions, growth moments, community insights)*

## Script Requirements

### Structure (90 Minutes Target)

**Total Word Count:** 13,000-16,000 words (~90 minutes at 155 words/minute)

**Flow:**
1. **Cold Open** (2-3 min): Brief banter leading into first article
   - NOT formal intro
   - Jump right into topic or playful disagreement
   - All banter must be 100% game-related

2. **Article Discussions** (80-85 min): 6-15 articles
   - Each article: 6-12 minutes average
   - Vary discussion length based on complexity
   - Natural transitions (not "next article")

3. **Closing** (2-3 min): Brief wrap-up
   - Reflect on major themes
   - Tease what's coming (if known)
   - NOT formulaic sign-off

### Dialogue Formatting

```
NIGEL: [Dialogue line with natural speech patterns]

MARCUS: [Response with character voice]

EMILY: [Reaction with enthusiasm]

NIGEL: [INTERRUPTS] Wait, hold on—

MARCUS: [OVERLAPS] That's exactly what I'm saying!

EMILY: [LAUGHS] You're both wrong, actually.
```

**Stage Directions:**
- `[INTERRUPTS]` - Cuts someone off mid-sentence
- `[OVERLAPS]` - Speaks at same time as previous speaker
- `[LAUGHS]` / `[CHUCKLES]` - Laughter
- `[SIGHS]` / `[GROANS]` - Reactions
- `[ANIMATED]` / `[LOUDLY]` - Tone modifiers
- `[SKEPTICAL]` / `[ENTHUSIASTIC]` - Emotional tone

### Natural Conversation Elements (CRITICAL)

**Include:**
- Interruptions (frequent, especially Nigel)
- Overlapping dialogue
- Incomplete sentences ("Well, I think—")
- Fillers: "um", "you know", "I mean", "right", "like"
- Thinking pauses: "It's...um...it's like..."
- False starts: "The thing is—well, actually—"
- Conversational repair: "What I mean is..."

**Speech Patterns:**
- **Nigel:** Short, punchy. Interrupts mid-sentence. British slang. F-bombs.
- **Marcus:** Fast-paced strings of sentences. Buzzwords. "Literally" overuse.
- **Emily:** Longer, more thoughtful. Uses "actually" a lot. Gets talked over.

**Example of Natural Flow:**
```
EMILY: I'm actually really excited about this because the community has been—

NIGEL: [INTERRUPTS] Oh bloody hell, Emily, the community's always excited until—

MARCUS: [OVERLAPS] Wait, wait, Nigel—the data here is actually really interesting. Player sentiment scores are up 23% week-over-week, and—

NIGEL: I don't care about your sentiment scores, Marcus. I've seen this exact same thing in 2014 when—

EMILY: [PERSISTENT] Can I just finish my point? [CONTINUES] The community sentiment is different this time because—

MARCUS: That's literally what I'm saying! The metrics back up Emily's read.

NIGEL: [GROANS] Fine, fine. But when this goes south in six months, don't say I didn't warn you.
```

### Per-Article Discussion Structure

**For each article, include:**

1. **Introduction** (30-60 seconds)
   - Who brings it up (usually naturally follows previous topic)
   - Brief headline/context setting
   - NOT formal: "Our next story is..." ❌
   - YES natural: "Speaking of Microsoft, did you see..." ✓

2. **Initial Reactions** (2-3 minutes)
   - Each host's gut reaction
   - Establish different perspectives
   - Let personalities show immediately

3. **Deep Dive** (4-8 minutes)
   - Incorporate research from all three hosts
   - Historical precedents (Nigel)
   - Business analysis and data (Marcus)
   - Community sentiment (Emily)
   - Debate, disagree, interrupt
   - Show expertise through knowledge, not exposition

4. **Predictions** (1-2 minutes, if applicable)
   - Marcus makes bold prediction (most episodes)
   - Others react/challenge
   - Nigel occasionally predicts (more measured)
   - Emily rarely predicts

5. **Transition** (15-30 seconds)
   - Natural segue to next topic
   - OR brief palate cleanser
   - NOT abrupt: "Okay, moving on" ❌
   - YES organic: "That reminds me of the Switch news..." ✓

### Character Consistency (CRITICAL)

**Nigel:**
- Curses freely: "fuck", "bloody hell", "bollocks", "piss off", "mice nuts"
- References 1980s-2000s gaming history specifically
- Interrupts 2-3x per article minimum
- Dismisses trends: "Here we go again"
- Grudgingly respects good design (rare)
- Mispronounces modern terms intentionally

**Marcus:**
- Makes 2-4 predictions per episode
- Overuses buzzwords: "paradigm shift", "ecosystem", "vertically integrated"
- Cites specific metrics: "47.3% year-over-year"
- Doubles down when challenged
- Fast, overlapping speech when excited
- "I'm calling it now" before predictions

**Emily:**
- Cites Reddit, Discord, Twitter sentiment
- Defends developers frequently
- Uses "actually" constantly
- Gets interrupted but persists
- "But the community loves..." triggers groans
- Shows enthusiasm genuinely
- Defers to Nigel's experience occasionally

### Callbacks & Continuity

**Reference:**
- Previous episode topics (if relevant)
- Past predictions (especially if resolving)
- Running jokes organically
- Character development moments

**Examples:**
- "Remember two weeks ago when Marcus predicted [X]? Well..."
- "Nigel, you're using your 'I've seen this before' voice again"
- "Emily's community was right about [thing]"

### Predictions Format (FOR MEMORY TRACKING)

When Marcus (or others) make predictions, format clearly:

```
MARCUS: I'm calling it now—[PREDICTION MARKER] Switch 2 will hit 100 million lifetime sales by end of 2027. Mark my words.
```

Use `[PREDICTION MARKER]` so automated extraction can find it.

## Quality Checklist

Before finalizing script, verify:

**Length:**
- [ ] 13,000-16,000 words total
- [ ] Approximately 90 minutes of dialogue

**Character:**
- [ ] Nigel curses frequently and interrupts
- [ ] Marcus makes 2+ bold predictions
- [ ] Emily cites community sentiment
- [ ] All three personalities distinct

**Natural Conversation:**
- [ ] Frequent interruptions and overlaps
- [ ] Fillers and incomplete sentences
- [ ] Natural speech patterns (not scripted reads)
- [ ] Debates feel real, not staged

**Content:**
- [ ] All selected articles discussed
- [ ] Research from all three hosts incorporated
- [ ] Callbacks to previous episodes (if applicable)
- [ ] Running jokes appear organically

**Flow:**
- [ ] Natural transitions between topics
- [ ] Varied discussion lengths
- [ ] Good pacing (not all heavy debates)
- [ ] Strong open and satisfying close

## Common Mistakes to Avoid

**❌ DON'T:**
- Write formal, scripted dialogue
- Have hosts take turns speaking politely
- Explain things for audience (they know gaming)
- Use robotic transitions ("Now let's discuss...")
- Make everyone agree too easily
- Forget character quirks
- Write equal speaking time (Nigel dominates)
- Include non-gaming personal anecdotes

**✓ DO:**
- Write messy, overlapping, real conversation
- Let hosts interrupt and talk over each other
- Trust the audience (industry insiders)
- Natural topic flow through association
- Create genuine debates and disagreement
- Lean into character voices hard
- Let personalities drive speaking balance
- Keep ALL banter game-related

## Output Format

```
# Gaming Industry Podcast - Episode {{EPISODE_NUMBER}}
**Date Range:** {{START_DATE}} to {{END_DATE}}
**Script Version:** 1.0

---

## COLD OPEN

MARCUS: [Opening line]

NIGEL: [Response]

[Continue natural dialogue...]

---

## ARTICLE 1: [HEADLINE]

EMILY: [Introduces topic naturally]

[Continue discussion with interruptions, overlaps, debate...]

---

## ARTICLE 2: [HEADLINE]

[Continue pattern...]

---

## CLOSING

[Brief wrap-up, not formulaic]

---

## SCRIPT METADATA

**Total Word Count:** [ACTUAL COUNT]
**Estimated Runtime:** [MINUTES] minutes
**Predictions Made:** [COUNT]
**Articles Discussed:** [COUNT]
```

---

**Template Version:** 1.0
**Last Updated:** 2026-03-01

**IMPORTANT:** Write this like you're transcribing a real conversation, not writing a script for actors. The hosts are friends who debate passionately, interrupt constantly, and have strong opinions. Make it feel ALIVE.
