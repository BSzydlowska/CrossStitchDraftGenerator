---
project: CrossStitchDraftGenerator
context_type: greenfield
updated: 2026-05-27
checkpoint:
  current_phase: 8
  frs_drafted: 9
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  quality_check_status: accepted
---

## Vision & Problem Statement

Sprzedawczyni wzorów haftowanych krzyżykowych na Etsy nie ma dostępnego narzędzia, które sprawnie konwertuje AI-generated images (proste grafiki wektorowe, max 10 kolorów, białe tło, wyraźny kontur) na gotowe patterny do sprzedaży. Istniejące programy (np. flossCross) są płatne, przestarzałe i nie były projektowane z AI-obrazem jako punktem wejścia — co oznacza żmudne ręczne poprawki i niską jakość outputu.

**Insight:** Stare programy zakładały, że użytkownik rysuje wzór samodzielnie lub konwertuje zdjęcie realistyczne. AI-generated images o cechach cross-stitch-friendly (solid colors, clean outline, simple shapes) to strukturalnie inny input — można go obsłużyć znacznie lepiej.

## User & Persona

**Główna persona:** Etsy seller — projektantka wzorów haftowanych krzyżykowych (solo)

- Tworzy wzory do sprzedaży cyfrowej na Etsy
- Generuje bazowe obrazy w narzędziach AI (prompt-based), np. z parametrami: `max 10 colors, white background, vector illustration for cross stitch conversion, simple shapes, solid colors, clean dark outline`
- Oczekuje szybkiej konwersji AI-obrazu → pattern gotowy do publikacji
- Jedna użytkowniczka na MVP (brak współdzielenia, brak ról)

## Access Control

**MVP:** Brak autoryzacji — pojedynczy użytkownik, dane lokalne, bez serwera auth.

> Forward note: Login (email/OAuth) planowany jako rozszerzenie v2 — umożliwi dostęp z wielu urządzeń lub udostępnianie wzorów. Nie wchodzi do MVP.

## Success Criteria

### Primary
MVP działa gdy: użytkowniczka uploaduje AI-generated image → ustawia szerokość (stitches), max colors, Aida count → aplikacja generuje podgląd patternu na siatce z paletą DMC, estimated size i difficulty score → projekt zostaje zapisany i można go ponownie otworzyć.

### Secondary (nice-to-have)
Etsy Listing Readiness: aplikacja generuje SEO title, tagi i krótki opis na podstawie parametrów patternu (liczba kolorów, rozmiar, poziom trudności).

### Guardrails
- Odwzorowanie kolorów DMC musi być wizualnie wierne — paleta nie może się losowo zmieniać między sesjami
- Preview generuje się bez przeładowania strony (płynne UX)
- Zapisany projekt otwiera się ponownie bez utraty danych

### Timeline
`mvp_weeks: 3` (after-hours, solo)

## Functional Requirements

### Core Generation
- FR-001: User can upload an image file. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.
- FR-002: User can set pattern width in stitches. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.
- FR-003: User can set max number of colors. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.
- FR-004: User can generate a cross-stitch draft preview from the uploaded image. Priority: must-have
  > Note: Aida count defaults to 16ct — not configurable in MVP.
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.

### Output & Palette
- FR-005: User can view the DMC palette assigned to the generated pattern. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.
- FR-006: User can view the estimated physical size of the pattern. Priority: must-have
  > Socrates: Kontrargument przyjęty częściowo: estimated size to wyliczenie, można pokazać jako tooltip/inline info zamiast osobnej sekcji UI — UX decyzja do podjęcia przy implementacji. FR stoi.

### Projects
- FR-007: User can save a project for later access. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.
- FR-008: User can reopen a saved project. Priority: must-have
  > Socrates: Kontrargument rozważony: brak kontrargumentu — FR stoi.

### Etsy Listing (nice-to-have)
- FR-009: User can generate Etsy listing assets (SEO title, tags, short description) based on pattern parameters. Priority: nice-to-have
  > Socrates: Kontrargument przyjęty: FR-009 jest niezależny od core flow — można go wyciąć do v2 bez szkody dla MVP. Pozostaje nice-to-have z ryzykiem scope creep.

## Business Logic

Application converts an uploaded image into a simplified cross-stitch draft by reducing colors, mapping them to DMC floss, and calculating pattern size.

**Three domain mechanisms in MVP:**

1. **Palette Optimization** — app performs color quantization on the uploaded image, reduces to user-specified max colors, then maps each resulting color to the nearest DMC floss by perceptual distance. The output is a stable, deterministic DMC palette for the pattern.

2. **Pattern Size Intelligence** — app calculates the physical size of the pattern (in cm/inches) based on stitch count and 16ct Aida default. Outputs: width × height in stitches, physical dimensions, hoop compatibility note.

3. **Stitch Quality Analysis** — confetti detection, color fragmentation warnings, isolated stitch count. **Deferred to v2.**

> Etsy Listing Readiness (SEO title/tags generation) — nice-to-have, deferred to v2 if time allows.

## Non-Functional Requirements

- Application runs in the browser without installation (web app — no local install required)
- Project data persists after browser close (local storage or equivalent)
- UI is usable on tablet and desktop (responsive layout)

## Non-Goals

- **PDF / OXS / Pattern Keeper export** — pełny eksport pattern'u to osobny projekt; MVP pokazuje podgląd on-screen
- **Integracja z Etsy API (auto-listing)** — przesyłanie do Etsy wymaga OAuth i API; poza zakresem MVP
- **Cover photo generation** — grafika sprzedażowa to osobny feature; nie blokuje core flow
- **Pixel editing (edycja kratka po kratce)** — pełny edytor to narzędzie innej klasy (jak Photoshop); MVP to generator, nie edytor
- **Backstitch / half-stitch** — zaawansowane typy ściegów; MVP obsługuje tylko full cross stitch
- **AI image generation** — generowanie obrazu bazowego zostaje po stronie zewnętrznych narzędzi AI; app przyjmuje gotowy obraz jako input
- **Multi-user / współpraca** — solo tool; bez udostępniania, bez kont zespołowych

## Forward: tech-stack

Brak twardych preferencji stacku — do ustalenia w `/10x-tech-stack-selector`.

## Quality cross-check

All 5 elements present — status: accepted.
- Access Control: present (no auth, local single user)
- Business Logic: present (one-sentence domain rule)
- Project artifacts: present (shape-notes.md with full frontmatter)
- Timeline-cost ack: present (3 weeks after-hours, accepted)
- Non-Goals: present (7 explicit non-goals)

## User Stories

### US-01: Generate cross-stitch draft
- **Given:** user has uploaded an image and set pattern width and max colors
- **When:** user triggers generation
- **Then:** app displays a grid preview with DMC palette and estimated physical size

### US-02: Save and reopen project
- **Given:** user has generated a cross-stitch draft
- **When:** user saves the project
- **Then:** project appears in the project list and can be reopened with all settings and preview intact

### US-03: Generate Etsy listing assets
- **Given:** user has a saved project with pattern parameters (size, color count)
- **When:** user requests Etsy listing generation
- **Then:** app outputs a suggested SEO title, tags, and short description ready to copy
