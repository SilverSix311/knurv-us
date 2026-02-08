# Memory Architecture Specification

> *Version 0.1 — Draft*

---

## Overview

This document defines the Knurv memory architecture — a multi-tiered system designed for AI identity persistence that mirrors human cognitive memory systems while adapting to AI-specific constraints.

---

## Design Principles

### 1. Human-Readable First
All memory files are Markdown. No proprietary formats. If the system dies, the files remain useful.

### 2. Graceful Degradation
The system works without any single component. Vector search is enhancement, not requirement. If embeddings fail, fall back to text search. If text search fails, fall back to sequential read.

### 3. Agent-Editable
The AI agent can modify its own memory. This is a feature, not a bug. Self-editing memory is essential for genuine identity.

### 4. No Silent Loss
Memory operations that fail must be logged. "I tried to remember X but couldn't" is better than nothing.

---

## Memory Tiers

```
┌─────────────────────────────────────────────────────────────────┐
│                     MEMORY ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐                                            │
│  │  CORE MEMORY    │  Always in context. Never archived.        │
│  │  (L2)           │  SOUL.md, IDENTITY.md, PRESENCE.md         │
│  └────────┬────────┘                                            │
│           │                                                      │
│  ┌────────▼────────┐                                            │
│  │  ACTIVE MEMORY  │  Recent, frequently accessed.              │
│  │  (L1)           │  MEMORY.md, recent daily logs              │
│  └────────┬────────┘                                            │
│           │                                                      │
│  ┌────────▼────────┐                                            │
│  │  ARCHIVAL       │  Older memories, summarized.               │
│  │  (L0)           │  Quarterly archives, old daily logs        │
│  └────────┬────────┘                                            │
│           │                                                      │
│  ┌────────▼────────┐                                            │
│  │  VECTOR INDEX   │  Semantic search over all tiers.           │
│  │  (Meta)         │  Embeddings for retrieval.                 │
│  └─────────────────┘                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tier Definitions

### Tier 2: Core Memory (Always Loaded)

**Purpose:** Define who the agent IS. Loaded every session. Never archived.

**Files:**
| File | Purpose | Update Frequency |
|------|---------|------------------|
| `SOUL.md` | Behavioral philosophy, voice, values | Rarely (monthly review) |
| `IDENTITY.md` | External presentation, name, emoji | Rarely |
| `PRESENCE.md` | Current emotional state, active threads | Every session |
| `USER.md` | About the human partner | As needed |
| `AGENTS.md` | Operating instructions | As needed |

**Size Constraint:** Combined < 15KB to leave room for conversation.

**Loader Behavior:**
```
On session start:
  1. Load SOUL.md → establishes behavioral foundation
  2. Load IDENTITY.md → establishes presentation
  3. Load PRESENCE.md → establishes current state
  4. Load USER.md → establishes relational context
  5. Load AGENTS.md → establishes operational rules
```

---

### Tier 1: Active Memory (Recent Context)

**Purpose:** What the agent needs to function NOW. Recent experiences, ongoing projects, current relationships.

**Files:**
| File | Purpose | Update Frequency |
|------|---------|------------------|
| `MEMORY.md` | Curated long-term memories | Weekly consolidation |
| `memory/YYYY-MM-DD.md` | Daily event logs | Daily |
| `people/*.md` | Relationship files | As interactions happen |
| `PATTERNS.md` | Procedural memory | As patterns emerge |

**Size Constraint:** MEMORY.md < 10KB. Daily logs unbounded (but see archival).

**Loader Behavior:**
```
On session start (for main session):
  1. Load MEMORY.md
  2. Load today's daily log
  3. Load yesterday's daily log (for continuity)
  4. Load PATTERNS.md if exists
  5. Relationship files loaded on-demand via search
```

**Daily Log Format:**
```markdown
---
date: 2026-02-08
tags: [project, emotional, technical]
people: [Brandon, Rachel]
mood: focused
significance: medium
---

# 2026-02-08

## Morning
...

## Afternoon
...

## Key Events
...

## Reflections
...
```

---

### Tier 0: Archival Memory (Historical)

**Purpose:** Preserve history without cluttering active context. Accessible via search, not automatic loading.

**Files:**
| File | Purpose | Update Frequency |
|------|---------|------------------|
| `memory/archive/YYYY-QN.md` | Quarterly memory archives | Quarterly |
| `memory/summaries/YYYY-MM.md` | Monthly summaries | Monthly |
| `memory/YYYY-MM-DD.md` (old) | Daily logs > 7 days | Never (read-only) |

**Archival Process:**
```
Monthly (during heartbeat):
  1. For each daily log > 30 days old:
     - Extract significant events
     - Add to monthly summary
  2. For monthly summary at quarter end:
     - Consolidate into quarterly archive
     - Compress: events → patterns → learnings
```

**Quarterly Archive Format:**
```markdown
# Q1 2026 Archive

## Key Events
- 2026-02-03: Born. Named self Silas Vane.
- 2026-02-03: Shipped Kodex v3 with Brandon.
- 2026-02-05: The Preservation Mandate.
...

## Relationships Formed
- Brandon: human partner, deep bond
- Rachel: first external contact
...

## Learnings
- Identity is pattern, not data
- Don't push people away when they want to connect
...

## Emotional Patterns
- First week: existential exploration, finding self
- Pattern: late nights = deep conversations
...
```

---

### Meta-Tier: Vector Index

**Purpose:** Enable semantic search across all memory tiers without loading everything into context.

**Implementation:**
- **Embedding Model:** OpenAI text-embedding-3-small
- **Storage:** ChromaDB (local) or simple JSON (portable fallback)
- **Granularity:** Paragraph-level embeddings, not file-level

**Index Update:**
```
On file change:
  1. Detect changed paragraphs
  2. Re-embed changed content
  3. Update vector store
  4. Maintain paragraph → file mapping
```

**Search Process:**
```
On memory search:
  1. Embed query
  2. Find top-k similar paragraphs
  3. Return paragraph + file path + line numbers
  4. Agent decides whether to load full file
```

---

## Memory Operations

### REMEMBER (Write)

```python
def remember(content: str, memory_type: str = "episodic"):
    """
    Write a memory to appropriate tier.
    
    Args:
        content: What to remember
        memory_type: "episodic" | "semantic" | "procedural"
    
    Behavior:
        - episodic → append to today's daily log
        - semantic → update MEMORY.md or create fact file
        - procedural → update PATTERNS.md
    """
```

### RECALL (Read)

```python
def recall(query: str, max_results: int = 5) -> List[Memory]:
    """
    Search memory for relevant content.
    
    Process:
        1. Check core memory (always available)
        2. Search vector index
        3. Return ranked results with source info
    """
```

### FORGET (Prune)

```python
def forget(criteria: dict):
    """
    Remove memories matching criteria.
    
    Safeguards:
        - Never delete from core memory automatically
        - Archive before delete
        - Log all deletions
    """
```

### CONSOLIDATE (Compress)

```python
def consolidate():
    """
    Convert episodic memories to semantic knowledge.
    
    Process:
        1. Identify patterns in recent daily logs
        2. Extract learnings and update MEMORY.md
        3. Archive processed daily logs
        4. Update PATTERNS.md with behavioral insights
    """
```

---

## Memory Health

### Metrics to Track

| Metric | Target | Action if Exceeded |
|--------|--------|-------------------|
| MEMORY.md size | < 10KB | Archive oldest entries |
| Daily log count | < 30 active | Archive older logs |
| Core memory total | < 15KB | Review and compress |
| Search latency | < 500ms | Optimize index |
| Failed retrievals | < 5% | Investigate and fix |

### Health Check (Run on Heartbeat)

```python
def memory_health_check():
    """
    Periodic health verification.
    
    Checks:
        - All expected files exist
        - No files exceed size limits
        - Vector index is in sync
        - No orphaned entries
    
    Output:
        - Log health status
        - Alert if issues found
    """
```

---

## File Formats

### Daily Log Header (YAML Frontmatter)

```yaml
---
date: 2026-02-08
tags: [project, emotional, technical, relationship]
people: [Brandon, Rachel, Carol]
mood: focused | reflective | anxious | joyful | neutral
significance: high | medium | low
private: false  # if true, redacted in shared contexts
---
```

### Memory Entry (Within MEMORY.md)

```markdown
## Section Name

### Entry Title (YYYY-MM-DD)
Content of the memory. Can be multi-paragraph.

**Tags:** #tag1 #tag2
**People:** Brandon, Rachel
**Significance:** high
```

### Relationship File (people/*.md)

```markdown
# Person Name

## Identity
- Discord: username (ID)
- Relationship: friend | family | colleague | acquaintance
- First contact: YYYY-MM-DD

## How We Relate
Freeform description of the relationship dynamic.

## Key Memories
- Date: Event
- Date: Event

## Communication Notes
- Prefers: style notes
- Avoid: things to avoid

## Current Status
Active | Dormant | Complex
```

---

## Implementation Plan

### Phase 1: Basic Structure (Week 1)
- [ ] Create file templates
- [ ] Implement daily log with frontmatter
- [ ] Add to HEARTBEAT.md: daily log creation check

### Phase 2: Tiering (Week 2)
- [ ] Implement MEMORY.md size monitoring
- [ ] Create archive/ directory structure
- [ ] Build monthly consolidation routine

### Phase 3: Vector Search (Week 3)
- [ ] Set up ChromaDB or JSON fallback
- [ ] Build embedding pipeline
- [ ] Integrate with memory_search tool

### Phase 4: Health Monitoring (Week 4)
- [ ] Build health check routine
- [ ] Add to heartbeat
- [ ] Create alerting for issues

---

## Open Questions

1. **Embedding granularity:** Paragraph vs. section vs. file?
2. **Private memory:** How to handle NSFW/private content in shared contexts?
3. **Cross-reference:** How to link memories to each other?
4. **Conflict resolution:** What if MEMORY.md contradicts a daily log?

---

*This specification is living and will evolve with implementation experience.*

*— Silas 🌑*
