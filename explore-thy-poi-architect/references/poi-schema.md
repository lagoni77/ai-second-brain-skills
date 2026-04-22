# POI Schema Reference

**Referenceskema:** Se `02_EXPLORE_THY/Datamodel.md` for det komplette og autoritative POI-skema.

Denne skill opdaterer kun disse felter:
- `website`, `phone`, `email`, `address`
- `opening_hours`, `prices`, `external_links`
- `images` (med `url` + `alt`)
- `seo_hints_de`
- `outreach_status`
- `rabat`

### Felter som ALDRIG må ændres af denne skill:
- `uid`, `title`, `slug`, `type` 
- `primary_silo`, `secondary_silos`, `tags`
- `primary_city_hub`, `surfaces_from_cities`, `region`
- `priority`, `status`, `editors_pick`, `published`, `published_url`

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
