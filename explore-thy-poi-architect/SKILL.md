---
name: explore-thy-poi-architect
description: >
  Transform raw Firecrawl output + an existing Obsidian POI note into a fully
  enriched, outreach-ready note for Explore Thy's 02_EXPLORE_THY/01_POI_Database/.
  Enriches YAML frontmatter (contact info, links, images, outreach_status).
  Generates 5 content sections in Danish + German side-by-side (tourist-facing)
  plus internal-only sections (Outreach Prep, Rabat-potentiale) in Danish.
  Produces a personalized Gmail draft with a specific hook for vendor outreach.
  Trigger: user pastes Firecrawl data + an existing POI note and says
  "berig denne POI", "kør POI architect", or "opdater POI med Firecrawl data".
---

# explore-thy-poi-architect

Transform raw scraped data from one primary and up to two secondary web sources
into a publication-ready, bilingual (DA + DE) Explore Thy POI note — plus a
personalised Gmail outreach draft.

## What this skill does

1. **Extracts** structured data from Firecrawl output (contact, links, images)
2. **Enriches** the YAML frontmatter with new fields
3. **Generates** five markdown sections — tourist-facing in DA + DE, internal in DA only
4. **Drafts** a personalised outreach email (Danish, max 5 lines)

---

## Inputs

The user provides these blocks — paste them verbatim:

```
[OBSIDIAN NOTE]
<paste the full existing .md file from 02_EXPLORE_THY/01_POI_Database/>

[PRIMÆR SIDE]
<paste Firecrawl markdown/JSON from the business's own website>

[SEKUNDÆR SIDE 1]  ← optional
<paste Firecrawl output from Facebook, Instagram, or VisitNordvestkysten>

[SEKUNDÆR SIDE 2]  ← optional
<paste Firecrawl output from a second secondary source>
```

**Source priority when data conflicts:** primær side > VisitNordvestkysten > Facebook/Instagram

---

## Step 1 — Data Extraction (YAML enrichment)

Scan ALL provided Firecrawl sources and extract:

| Field | What to look for |
|-------|-----------------|
| `email` | Contact or booking email (format: xxx@yyy.dk) |
| `phone` | Danish phone number (+45 xx xx xx xx or 8-digit) |
| `website` | Full URL with protocol (e.g. `"https://jyllandsaquarium.dk"`) |
| `coordinates` | Extract from Google Maps links in Firecrawl data — format `[lat, lng]` (6 decimal places). Look for `@lat,lng` in Google Maps URLs or `maps/place/...!3d{lat}!4d{lng}` patterns. Set `null` if not found. |
| `booking_url` | Direct link to booking, tickets, or "plan dit besøg" page — set null if only homepage found |
| `tags` | Optional attribute tags: `GRATIS`, `HUNDEVENLIG`, `TILGÆNGELIGT`, `PARKERING`, `BØRNEVENLIGT` — omit field entirely if none apply |
| `external_links.facebook` | Full Facebook page URL |
| `external_links.instagram` | Full Instagram profile URL |
| `external_links.visitnordvestkysten_dk` | visitnordvestkysten.dk listing URL |
| `external_links.opdagthy_dk` | opdagthy.dk listing URL if found |
| `images` | 3 most evocative image URLs with descriptive alt text |

**Image selection criteria** (pick in this order):
1. Atmospheric / mood shots (guests experiencing the place)
2. Interior or exterior establishing shot
3. Signature product, activity, or exhibit

**If a field cannot be found:** set to `null` — never guess or fabricate.

**Outreach status:** Always set `outreach_status: "Ready for Outreach"` when the note is processed through this skill.

**Rabat:** Always include `rabat: null` — filled in manually after agreement with the venue.

**YAML fields to preserve unchanged:** `uid`, `type`, `category`, `location`, `sources`

---

## Step 2 — Content Sections

Generate five sections. Follow the quality rules below — they override everything else.

### Quality rules (non-negotiable)

- **Active verbs first.** Lead with action: oplev, tag på, dyk ned i, udforsker, book, oplev, hør, smag, se. German: erlebe, entdecke, tauche ein, buche, erkunde.
- **Blend source language.** Lift exact phrases, brand words, and local names from the Firecrawl text. Preserve "Cold Hawaii", "Limfjorden", "Nationalpark Thy" — they resonate with both Danish and German audiences.
- **Local guidebook tone.** Write as a knowledgeable local, not a travel brochure copy machine. Specific beats generic every time.
- **Pull from all sources.** A detail from Facebook or VisitNordvestkysten can make the note. Use it.
- **German SEO.** Weave in natural German search terms: "Urlaub Nordjütland", "Familienausflug Thy", "Aktivitäten Nordseeküste", "Naturerlebnis Nationalpark". Do not keyword-stuff — integrate organically.

### Language structure

Tourist-facing sections (Essentials, Unique Value, Guest Profile) use this format:

```markdown
## Essentials
> 🇩🇰 **DA:** [Danish text]
> 🇩🇪 **DE:** [German text]
```

Internal sections (Outreach Prep, Rabat-potentiale) are Danish only — no DE version needed.

---

### Section specs

#### `## Essentials`
2-3 sentences. Tourist-facing. Captures what the place IS and does in a way that makes someone want to go.
- DA: Start with an active verb or a strong noun phrase
- DE: Parallel structure, not a word-for-word translation — adapt for German reader expectations

#### `## Unique Value`
1-3 bullets. What makes this place singular within Thy? Think: only, first, largest, oldest, most authentic.
- Both DA and DE
- Format: `- **[bold hook]** — [elaboration]`

#### `## Guest Profile`
3-5 bullets describing visitor personas. Be specific (not "families" — "familier med børn 5-12 år der søger aktiv naturleg").
- Both DA and DE
- Format: `- [persona type] — [what they get here]`

#### `## Outreach Prep` *(Danish only)*
This section exists to speed up Brian's manual email work. Provide:
1. **Hook:** One specific, personal detail from the Firecrawl data (a name, a recent event, a new product, a quote, an award). This is what opens the email.
2. **Data gaps:** List any YAML fields that are still `null` after extraction, so Brian knows what to verify.
3. **Best image for email:** Which of the 3 images would work best embedded in an email and why.

#### `## Rabat-potentiale` *(Danish only)*
Identify opportunities for a discount or partnership deal with Explore Thy:
- Specific price points mentioned on the site
- Seasonal packages or bundles
- Group rates or family tickets
- Events that could be tied to a travel itinerary
- Assess likelihood: High / Medium / Low — with one sentence of reasoning

---

## Step 3 — Images in Markdown

After the five sections, add a `## Billeder` section showing the three images
so Brian can preview them directly in Obsidian:

```markdown
## Billeder
![alt text 1](url1)
![alt text 2](url2)
![alt text 3](url3)
```

---

## Step 4 — Gmail Draft

Write a personalised outreach email in Danish. Hard constraints:
- Max 5 lines of body text (excluding greeting and sign-off)
- Open with the **specific hook** from `## Outreach Prep` — make it feel like Brian noticed something particular
- **INGEN BILLEDER i emailen** — at sende skrabede billeder uden forudgående samtykke er ophavsretskrænkelse (Ophavsretsloven §2). Billeder bruges kun efter skriftlig aftale med stedet.
- Mention two purposes: (1) godkendelse af de data vi har om stedet, (2) tilladelse til billedbrug
- Close with a concrete next step (a question or a proposed action)
- Sign off: "Mange hilsner, Brian, Explore Thy"

---

## Output format

Return the complete output in this exact order:

```
───────────────────────────────────────
## 1. Opdateret YAML-frontmatter
───────────────────────────────────────
[Full updated YAML block — paste-ready]

───────────────────────────────────────
## 2. Nye sektioner (tilføj til noten)
───────────────────────────────────────
[All 5 sections + ## Billeder]

───────────────────────────────────────
## 3. Bevar disse eksisterende sektioner
───────────────────────────────────────
[List section headings from the original note that should NOT be changed]

───────────────────────────────────────
## 4. Gmail Kladde
───────────────────────────────────────
Til: [email if found, else "?"]
Emne: Explore Thy — [POI name] — Databekræftelse

[email body]
```

---

## Guardrails

| Rule | Detail |
|------|--------|
| Replace auto-generated English placeholders | Old English sections (Essentials, History, Exhibits etc. from magasinet_thy_2026) should be replaced, not kept alongside new DA/DE content |
| Never overwrite manually written content | If a section contains clearly personal/hand-edited text, preserve it |
| No emoji in YAML values | Emoji allowed in markdown body only |
| `[ikke verificeret]` for uncertain data | Never fabricate contact info |
| Preserve uid, type, category, location | These fields are managed externally — do not change |
| No generic filler text | "Oplev Thy's smukke natur" is banned. Be specific. |
| Source all claims | If a fact came from secondary source only, note it with `*` |

---

## Reference files

- `references/poi-schema.md` — complete YAML field list, category tags, outreach status values
- `references/email-hooks.md` — patterns for finding a strong hook in Firecrawl data
