# FightPKD — Agent Reference Guide

This file is a structured guide for AI agents to understand, update, and maintain the FightPKD website data.

**IMPORTANT:** Before making changes, read `CONSTITUTION.md` — it defines update rules, the sources log to maintain, and validation checklist.

## Project Overview

A static website (Astro + Tailwind CSS) deployed to GitHub Pages that tracks the drug development pipeline and therapeutic research for Polycystic Kidney Disease (PKD/ADPKD). Requires Node.js 22+.

## Data Files

### `src/data/drugs.json`
Pipeline drugs currently in clinical trials for PKD.

**Schema:**
```json
{
  "id": "string (kebab-case slug)",
  "name": "string (display name)",
  "company": "string (sponsor/company name)",
  "companyLocation": "string (city, country)",
  "phase": "number (1, 2, or 3)",
  "status": "string (see values below)",
  "trialName": "string (trial nickname or study name)",
  "startDate": "string (YYYY-MM format)",
  "estimatedCompletion": "string (YYYY or YYYY-MM)",
  "mechanism": "string (brief mechanism of action)",
  "participants": "number (target enrollment)",
  "nctId": "string (ClinicalTrials.gov NCT number, empty if unknown)",
  "notes": "string (additional context)"
}
```

**Status values:**
- `recruiting` — actively enrolling patients
- `not_yet_recruiting` — approved but not yet open
- `active_not_recruiting` — ongoing but enrollment closed
- `completed` — trial finished
- `enrolling_by_invitation` — not publicly open

### `src/data/therapies.json`
Non-drug therapeutic interventions studied for PKD.

**Schema:**
```json
{
  "id": "string (kebab-case slug)",
  "name": "string (display name)",
  "category": "string (dietary | supplement | lifestyle | repurposed_drug)",
  "status": "string (completed | in_progress | observational)",
  "studyName": "string (trial or study name)",
  "institution": "string (lead institution)",
  "resultsSummary": "string (brief results or current status)",
  "link": "string (internal link like /keto, or empty)",
  "publicationDate": "string (YYYY or YYYY-MM)",
  "researchers": "string (key researchers)"
}
```

## How to Update Drug Information

1. **Search ClinicalTrials.gov API:**
   ```
   GET https://clinicaltrials.gov/api/v2/studies?query.cond=polycystic+kidney+disease&query.intr={DRUG_NAME}&fields=NCTId,BriefTitle,OverallStatus,StartDateStruct,PrimaryCompletionDateStruct,Phase,EnrollmentInfo
   ```

2. **Check key fields:** `OverallStatus`, `StartDateStruct`, `Phase`, `EnrollmentInfo`

3. **Update the corresponding entry** in `src/data/drugs.json`

4. **If a drug moves to a new phase**, update the `phase` field

5. **If enrollment status changes**, update `status` using the values listed above

## How to Add a New Drug

1. Search ClinicalTrials.gov for new PKD/ADPKD interventional studies:
   ```
   GET https://clinicaltrials.gov/api/v2/studies?query.cond=polycystic+kidney+disease&filter.overallStatus=RECRUITING,NOT_YET_RECRUITING&fields=NCTId,BriefTitle,OverallStatus,LeadSponsorName,InterventionName,Phase
   ```

2. Create a new entry in `src/data/drugs.json` following the schema

3. The `DrugCard` component (`src/components/DrugCard.astro`) will automatically render it

4. Assign an appropriate `id` (kebab-case of the drug name)

## How to Update Therapy Information

1. Search PubMed for recent publications:
   - Query: `"polycystic kidney disease" AND "{therapy_name}"`
   - Filter: last 12 months

2. Check ClinicalTrials.gov for trial status updates

3. Update the corresponding entry in `src/data/therapies.json`

4. If a new therapy study is published, add a new entry

## How to Update the Keto Page

The keto page (`src/pages/keto.astro`) is a long-form article. To update it:

1. Search for new publications from Weimbs lab, Mueller group, or DIPAK consortium
2. Check for new clinical trial registrations with "ketogenic" or "ketone" + "polycystic kidney"
3. Edit the relevant section in the Astro file directly
4. Key sections: KETO-ADPKD results, KetoCitra/Ren-Nu data, preclinical evidence

## Key Search Sources

| Source | URL | Use For |
|--------|-----|---------|
| ClinicalTrials.gov API v2 | https://clinicaltrials.gov/api/v2/studies | Trial status, enrollment, phases |
| PubMed | https://pubmed.ncbi.nlm.nih.gov/ | Published results, new research |
| PKD Foundation | https://pkdcure.org/ | Patient-facing news, research updates |
| EudraCT/CTIS | https://euclinicaltrials.eu/ | European trials not yet on CT.gov |
| ANZCTR | https://www.anzctr.org.au/ | Australian/NZ trials (PYC-003) |
| Company press releases | Various | Pipeline updates, acquisition news |

## Drug Names & Aliases

When searching, use these alternative names:

| Drug | Aliases |
|------|---------|
| ABBV-CLS-628 | CLS-628, Calico compound |
| AL01211 | AceLink GCS inhibitor |
| AZD1613 | AstraZeneca PAPPA-1 inhibitor, PIONEER-PKD |
| Farabursen | RGLS8429, CYX082, anti-miR-17, Regulus, Novartis |
| PYC-003 | PYC Therapeutics antisense, peptide-PMO |
| VX-407 | Vertex PKD modulator, AGLOW |
| JMKX003142 | Jemincare |
| Bempedoic Acid | BEAT-PKD, bempedoic |
| Dapagliflozin | STOP-PKD, Farxiga/Forxiga |
| Empagliflozin | EMPA-PKD, SIDIA, Jardiance |
| Metformin | IMPEDE-PKD, TAME-PKD |

## How to Update Homepage Stats

The homepage (`src/pages/index.astro`) displays 4 stat cards. Three are computed automatically from the JSON data. The fourth — "Studies Published in 2026" — must be updated manually:

1. **Fetch the count from PubMed:**
   ```
   https://pubmed.ncbi.nlm.nih.gov/?term=polycystic+kidney+disease&filter=dates.2026%2F1%2F1-2026%2F12%2F31
   ```
   Look for the total results count at the top of the search results page.

2. **Update the number** in `src/pages/index.astro` (search for the `<p>` tag with the number, around line 96).

3. **Update the `lastUpdated` variable** at the top of the file to the current date.

The stat card links to this same PubMed URL so users can verify the number themselves.

## Update Frequency Recommendation

- **Drug pipeline:** Monthly — trial statuses change frequently
- **Therapy research:** Quarterly — publications come in waves
- **Keto page:** Quarterly or when major new publications appear
- **Full audit:** Every 6 months — check for removed/cancelled trials

## Site Architecture

```
src/
├── pages/
│   ├── index.astro          (home page with stats)
│   ├── pipeline.astro       (drug pipeline dashboard)
│   ├── therapies.astro      (therapeutic research dashboard)
│   └── keto.astro           (dedicated keto research page)
├── components/
│   ├── DrugCard.astro       (renders one drug entry)
│   ├── PipelineTracker.astro (phase overview visualization)
│   ├── TherapyCard.astro    (renders one therapy entry)
│   ├── Header.astro         (navigation)
│   └── Footer.astro
├── data/
│   ├── drugs.json           ← UPDATE THIS
│   └── therapies.json       ← UPDATE THIS
├── layouts/
│   └── BaseLayout.astro
└── styles/
    └── global.css
```

## Build & Deploy

```bash
npm install          # install dependencies
npm run dev          # local development server
npm run build        # build static site to ./dist/
```

Deployment is automated via GitHub Actions (`.github/workflows/deploy.yml`) — push to `main` triggers a build and deploy to GitHub Pages.
