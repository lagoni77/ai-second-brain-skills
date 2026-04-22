# POI Schema Reference

Complete field reference for `02_EXPLORE_THY/01_POI_Database/` notes.

---

## YAML Frontmatter

### Core fields (always present — never change via this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `uid` | string | `LOC-001009` | Assigned by Antigravity Agent. Tracked in `98_SYSTEM/ID_Registry.json` |
| `type` | string | `"business"` or `"place"` | `business` = commercial operator; `place` = non-commercial site |
| `category` | array | `[AKTIVITET, OPLEVELSER]` | See category tags below |
| `location` | string | `Agger` | Town/area name |
| `updated` | ISO date | `2026-04-15` | Update when note is meaningfully changed |
| `sources` | array | `[magasinet_thy_2026.md]` | Source file names |

### Enrichment fields (added/updated by this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `website` | string or null | `"https://jyllandsaquarium.dk"` | Full URL with https://, no trailing slash |
| `coordinates` | array or null | `[57.1208, 8.6165]` | `[lat, lng]` — bruges af Obsidian Map View (konfig: property = `coordinates`) |
| `booking_url` | string or null | `"https://jyllandsaquarium.dk/book/"` | Direct link to booking, tickets, or "plan dit besøg" page |
| `tags` | array or null | `[GRATIS, HUNDEVENLIG]` | Cross-cutting attributes — se tag-oversigt nedenfor |
| `email` | string or null | `info@stedet.dk` | Primary contact email |
| `phone` | string or null | `"+45 97 12 34 56"` | Quote if starts with + |
| `outreach_status` | string | `"Ready for Outreach"` | See status values below |
| `rabat` | string or null | `"10% på entre"` | Udfyldes efter aftale — fx "Gratis entre for børn" eller "15% med Explore Thy-kode" |
| `external_links` | object | see below | All keys required; null if not found |
| `images` | array | see below | 3 entries; null entries allowed |

### `external_links` sub-fields

```yaml
external_links:
  opdagthy_dk: null                          # or full URL
  visitnordvestkysten_dk: "https://..."      # or null
  facebook: "https://www.facebook.com/..."  # or null
  instagram: "https://www.instagram.com/..." # or null
  nationalparkthy_dk: null                   # or full URL
```

### `images` array structure

```yaml
images:
  - url: "https://example.com/image1.jpg"
    alt: "Gæster på delfinsafari i Agger Tange"
  - url: "https://example.com/image2.jpg"
    alt: "Berøringsbassinet med rokker og kattehaj"
  - url: "https://example.com/image3.jpg"
    alt: "Eksteriør af JyllandsAkvariet ved Limfjorden"
```

Alt text: Descriptive Danish phrase. What is in the image? Not "billede af stedet".

---

## Attribute Tags (`tags` field)

Tværgående attributter — kombineres frit. Adskilt fra `category` da de beskriver *egenskaber*, ikke *type*.

| Tag | Hvornår |
|-----|---------|
| `GRATIS` | Gratis adgang — stærkt USP på platformen |
| `HUNDEVENLIG` | Hunde velkomne |
| `TILGÆNGELIGT` | Kørestolsvenligt / handicapvenligt |
| `PARKERING` | Gratis parkering på stedet |
| `BØRNEVENLIGT` | Særligt egnet til børn |
| `NATTELIV` | Åbent om aftenen / nataktivitet |

Feltet udelades helt hvis ingen tags er relevante (sæt ikke `tags: []`).

---

## Category Tags

Use uppercase. Multiple categories allowed.

| Tag | Bruges til |
|-----|-----------|
| `NATUR` | Strande, skove, nationalpark-natur, fuglereservater |
| `AKTIVITET` | Sport, udendørs aktiviteter, vandreture, kajak, ridning |
| `OPLEVELSER` | Museer, attraktioner, events, shows, rundvisninger |
| `OVERNATNING` | Hoteller, camping, sommerhus, B&B |
| `SPISESTEDER` | Restauranter, cafeer, is, street food |
| `STEDER` | Historiske steder, kirker, havne, monumenter |
| `PRAKTISK` | Turistinfo, parkering, transport, apoteker |
| `VELVÆRE` | Spa, yoga, behandlinger, wellness |

---

## Outreach Status Values

Progressive workflow — update manually as outreach proceeds.

| Status | Betydning |
|--------|-----------|
| `"Ready for Outreach"` | Note er beriget, data er verificeret nok til at sende email |
| `"Contacted"` | Første email sendt |
| `"Follow-up Sent"` | Rykkere afsendt |
| `"Confirmed"` | Stedet har godkendt data og billedbrug |
| `"Declined"` | Stedet ønsker ikke at deltage |
| `"On Hold"` | Afventer svar / sæson / andet |

---

## Standard Markdown Section Order

When building or reviewing a POI note, sections should appear in this order:

1. `## Essentials` — kort opsummering (DA + DE for nye noter)
2. `## [Aktiviteter / Udstillinger / Tilbud]` — hvad tilbyder stedet (varierer efter type)
3. `## Unique Value` — unikt for Thy (DA + DE)
4. `## Guest Profile` — målgruppe-personas (DA + DE)
5. `## Åbningstider & Sæson` — når det er relevant
6. `## Priser` — når det er relevant
7. `## Billeder` — tre preview-billeder (`![alt](url)`)
8. `## Contact` — adresse, telefon, email, website
9. `## Outreach Prep` — intern, kun DA
10. `## Rabat-potentiale` — intern, kun DA

Eksisterende noter der følger et andet rækkefølge: rør dem ikke. Append nye sektioner i bunden.
