---
project: CrossStitchDraftGenerator
version: 1
status: draft
created: 2026-05-27
context_type: greenfield
product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 3
  hard_deadline: null
  after_hours_only: true
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

## Success Criteria

### Primary
MVP działa gdy: użytkowniczka uploaduje AI-generated image → ustawia szerokość (stitches) i max colors → aplikacja generuje podgląd patternu na siatce z paletą DMC i estimated size → projekt zostaje zapisany i można go ponownie otworzyć.

### Secondary
Etsy Listing Readiness: aplikacja generuje SEO title, tagi i krótki opis na podstawie parametrów patternu (liczba kolorów, rozmiar), gotowy do skopiowania.

### Guardrails
- Odwzorowanie kolorów DMC musi być wizualnie wierne — paleta nie może się losowo zmieniać między sesjami
- Podgląd patternu generuje się bez przeładowania strony (płynne UX bez page flash)
- Zapisany projekt otwiera się ponownie bez utraty danych

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

## Non-Functional Requirements

- Aplikacja działa w przeglądarce bez instalacji — użytkownik nie instaluje żadnego oprogramowania lokalnie
- Dane projektów są dostępne po ponownym otwarciu przeglądarki i zamknięciu zakładki
- Interfejs jest użyteczny na tablecie i desktopie (responsywny układ)

## Business Logic

Application converts an uploaded image into a simplified cross-stitch draft by reducing colors, mapping them to DMC floss, and calculating pattern size.

**Two domain mechanisms in MVP:**

1. **Palette Optimization** — the pattern palette is reduced to the user-specified maximum color count; each resulting color is matched to the visually closest DMC thread color. The output is a stable, deterministic DMC palette that does not change between sessions for the same inputs. When the uploaded image contains more distinct colors than the specified maximum, the least visually distinct colors are merged with their neighbors before DMC matching.

2. **Pattern Size Intelligence** — the pattern's physical dimensions (width × height in cm and inches) and standard hoop compatibility are derived from the stitch count using 16ct Aida as the default fabric. The output tells the user what canvas size is needed and whether the pattern fits a standard hoop.

> **Deferred to v2:** Stitch Quality Analysis (confetti detection, color fragmentation warnings, isolated stitch count). Etsy Listing Readiness (SEO title/tags generation) is a nice-to-have included only if time allows within the MVP window.

## Access Control

Single user; no authentication; data lives in the browser only. The application does not require an account and does not send project files or images to any remote service.

## Non-Goals

- **PDF / OXS / Pattern Keeper export** — pełny eksport pattern'u to osobny projekt; MVP pokazuje podgląd on-screen
- **Integracja z Etsy API (auto-listing)** — przesyłanie do Etsy wymaga OAuth i API; poza zakresem MVP
- **Cover photo generation** — grafika sprzedażowa to osobny feature; nie blokuje core flow
- **Pixel editing (edycja kratka po kratce)** — pełny edytor to narzędzie innej klasy; MVP to generator, nie edytor
- **Backstitch / half-stitch** — zaawansowane typy ściegów; MVP obsługuje tylko full cross stitch
- **AI image generation** — generowanie obrazu bazowego zostaje po stronie zewnętrznych narzędzi AI; app przyjmuje gotowy obraz jako input
- **Multi-user / współpraca** — solo tool; bez udostępniania, bez kont zespołowych

## Open Questions

1. **Success Criteria vs FR reconciliation (minor, non-blocking):** The initial Phase 3 success criterion referenced "Aida count" as a user-configurable parameter and "difficulty score" as an output. Both were removed in Phase 4 FR decisions (Aida count fixed at 16ct default; difficulty score dropped as not required). The Primary criterion above reflects the Phase 4 decisions. No action needed — recorded for transparency.
