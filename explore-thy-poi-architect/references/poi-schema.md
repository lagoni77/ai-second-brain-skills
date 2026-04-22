# POI Schema Reference

Complete field reference for `02_EXPLORE_THY/01_POI_Database/` notes.
**SSOT:** Always defer to `02_EXPLORE_THY/Datamodel.md` for the authoritative schema.

---

## YAML Frontmatter

### Core fields (always present — never change via this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `uid` | string | `LOC-001051` | Assigned by Antigravity Agent. Tracked in `98_SYSTEM/ID_Registry.json` |
| `title` | string | `"Bulbjerg"` | Display name |
| `slug` | string | `bulbjerg` | Kebab-case, URL-safe. Used as DB primary key |
| `type` | string | `place` or `business` | `business` = commercial operator; `place` = non-commercial site |
| `updated` | ISO date | `2026-04-20` | Update when note is meaningfully changed |
| `sources` | array | `[nationalparkthy.dk]` | Source domain names |

### Silo & kategori-felter (never change via this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `primary_silo` | string | `NATUR` | URL-hjem: NATUR, OPLEVELSER, AKTIVITETER, SPISESTEDER, OVERNATNING, STEDER |
| `secondary_silos` | array | `[STEDER]` | Surfer POI'en på disse siloers listesider |
| `tags` | array | `[kyst, udsigt, fugle]` | Finkornet filtrering på tværs af siloer (lowercase kebab) |

### Geografi (never change via this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `primary_city_hub` | string | `Hanstholm` | Primær kystby |
| `surfaces_from_cities` | array | `[Hanstholm, Klitmøller]` | Alle kystby-hubs der linker til denne POI |
| `region` | string | `Nordthy` | Overordnet geografisk region |
| `coordinates` | object | `{lat: 57.1003, lng: 8.7537}` | **YAML object** — ikke array. `lat`/`lng` som decimaler |

### Kurations-felter (never change via this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `priority` | string | `high` | `high` / `medium` / `low` |
| `status` | string | `keep` | `keep` / `cut` / `uncertain` |
| `editors_pick` | boolean | `false` | `true` for ~20 signatursteder |
| `published_url` | string | `/natur/bulbjerg` | Planlagt URL-sti |
| `published` | boolean | `false` | Sættes til `true` ved go-live |

### Enrichment fields (updated by this skill)

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `website` | string or null | `"https://bulbjerg.dk/"` | Full URL med https://, ingen trailing slash |
| `phone` | string or null | `"+45 97 12 34 56"` | Quote hvis starter med + |
| `email` | string or null | `info@stedet.dk` | Primær kontakt-email |
| `address` | string or null | `"Bulbjerg, Nationalpark Thy"` | Menneskelig adresse |
| `opening_hours` | string or null | `"Frit tilgængeligt hele året"` | Fritekst |
| `prices` | object | `{}` | Tom dict hvis gratis/ukendt |
| `images` | array | se nedenfor | Max 3 entries |
| `seo_hints_de` | string or null | `"Bulbjerg Kliff Dänemark, ..."` | Kommaseparerede tyske søgeord |
| `external_links` | object | se nedenfor | Alle kendte keys |
| `outreach_status` | string | `"Ready for Outreach"` | Se status-værdier nedenfor |
| `rabat` | string or null | `"10% på entre"` | Udfyldes efter aftale med stedet |

### `external_links` sub-fields

```yaml
external_links:
  opdagthy_dk: null                          # eller full URL
  visitnordvestkysten_dk: "https://..."      # eller null
  facebook: "https://www.facebook.com/..."   # eller null
  instagram: "https://www.instagram.com/..." # eller null
  nationalparkthy: null                      # eller full URL
```

### `images` array structure

```yaml
images:
  - url: "https://example.com/image1.jpg"
    alt: "Gæster på Bulbjerg med udsigt over Nordsøen"
  - url: "https://example.com/image2.jpg"
    alt: "Skarreklit set fra klinten"
  - url: null
    alt: null
```

Alt text: Beskrivende dansk sætning. Hvad er i billedet? Ikke "billede af stedet".

---

## Silo-tags (primary_silo + secondary_silos)

| Tag | Bruges til |
|-----|-----------|
| `NATUR` | Strande, skove, nationalpark, fuglereservater |
| `AKTIVITETER` | Sport, udendørs, vandreture, kajak, ridning |
| `OPLEVELSER` | Museer, attraktioner, events, rundvisninger |
| `OVERNATNING` | Hoteller, camping, sommerhus, B&B |
| `SPISESTEDER` | Restauranter, cafeer, is, street food |
| `STEDER` | Historiske steder, kirker, havne, monumenter |

---

## Outreach Status Values

| Status | Betydning |
|--------|-----------|
| `"Ready for Outreach"` | Note er beriget, klar til email |
| `"Contacted"` | Første email sendt |
| `"Follow-up Sent"` | Rykkere afsendt |
| `"Confirmed"` | Stedet har godkendt data og billedbrug |
| `"Declined"` | Stedet ønsker ikke at deltage |
| `"On Hold"` | Afventer svar / sæson / andet |

---

## Standard Markdown Section Order

1. `## Beskrivelse` — redaktionel tekst på dansk
2. `## Praktisk` — adgang, parkering, åbningstider, hunde
3. `## Essentials` — turistvendt intro (DA + DE)
4. `## Unique Value` — hvad gør stedet unikt i Thy (DA + DE)
5. `## Guest Profile` — målgruppe-personas (DA + DE)
6. `## Billeder` — tre preview-billeder (`![alt](url)`)
7. `## Outreach Prep` — intern, kun DA
8. `## Rabat-potentiale` — intern, kun DA

Eksisterende noter der følger andet rækkefølge: rør dem ikke. Append nye sektioner i bunden.
