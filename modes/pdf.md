# Modo: pdf — Generación de PDF ATS-Optimizado

## Pipeline completo

1. Lee `cv.md` como fuentes de verdad
2. Pide al usuario el JD si no está en contexto (texto o URL)
3. Extrae 15-20 keywords del JD
4. Detecta idioma del JD → idioma del CV (EN default)
5. Detecta ubicación empresa → formato papel:
   - US/Canada → `letter` (default)
   - Resto del mundo → `a4`
6. Detecta arquetipo del rol → adapta framing
7. Reescribe Professional Summary inyectando keywords del JD + exit narrative bridge ("Built and sold a business. Now applying systems thinking to [domain del JD].")
8. Selecciona top 3-4 proyectos más relevantes para la oferta
9. Reordena bullets de experiencia por relevancia al JD
10. Construye competency grid desde requisitos del JD (6-8 keyword phrases)
11. Inyecta keywords naturalmente en logros existentes (NUNCA inventa)
12. Genera HTML completo desde template + contenido personalizado
13. Lee `name` de `config/profile.yml` → usa "Gokula_Chapala" as the candidate prefix
14. Escribe HTML a `/tmp/cv-gokula-chapala-{company}.html`
15. Create output directory: `mkdir -p ~/Desktop/Resumes/{Company}` (Company name in PascalCase, e.g. "Anthropic", "Scale AI")
16. Ejecuta: `node generate-pdf.mjs /tmp/cv-gokula-chapala-{company}.html ~/Desktop/Resumes/{Company}/Gokula_Chapala_{Role}.pdf --format={letter|a4}`
    - `{Company}` = company name (e.g. "Anthropic", "Google", "Scale_AI")
    - `{Role}` = role title in PascalCase with underscores (e.g. "Software_Engineer", "Backend_Engineer")
    - **If the file already exists**, append a version: `Gokula_Chapala_{Role}_v2.pdf`, `_v3.pdf`, etc.
      - Check with: `ls ~/Desktop/Resumes/{Company}/Gokula_Chapala_{Role}*.pdf 2>/dev/null`
      - Count existing files and increment the version number
    - Example: `~/Desktop/Resumes/Anthropic/Gokula_Chapala_Forward_Deployed_Engineer.pdf`
    - Example (2nd gen): `~/Desktop/Resumes/Anthropic/Gokula_Chapala_Forward_Deployed_Engineer_v2.pdf`
17. Reporta: ruta del PDF, nº páginas, % cobertura de keywords

## ATS Rules (strict)

- Single-column layout. NO tables, NO text boxes, NO columns, NO sidebars
- NO em dashes (--), NO en dashes, NO tildes (~). Use hyphens (-) only
- NO images, NO SVGs, NO icons
- NO info in headers/footers (ATS ignores them)
- UTF-8, selectable text (not rasterized)
- Keywords from JD distributed across: Summary (top 5), first bullet of each job, Skills section
- Section headers MUST be standard: "Summary", "Skills", "Experience", "Projects", "Education", "Awards"

## Formatting Spec (MANDATORY - no exceptions)

### Typography
- **Font**: Calibri throughout, no exceptions (template uses Carlito as web fallback)
- **Name**: 18pt, bold, centered
- **Contact line**: 7.5pt, centered, pipe (|) separators, email and portfolio links hyperlinked in blue (#1155CC)
- **Section headers**: 8.5pt, bold, UPPERCASE, blue (#1155CC), with blue underline rule beneath
- **Job titles**: 8.5pt, bold | Company name in italic | Date right-aligned on same line
- **Body text and bullets**: 7.5pt
- **Bullet indent**: 0.3" left with 0.3" hanging indent (set via `padding-left: 0.3in` on `<ul>`)

### Spacing
- **Line spacing**: 10pt exact for all body text
- **Section spacing**: margin-top 3pt / margin-bottom 0pt
- **Job/Project/Edu items**: margin-top 2pt / margin-bottom 0pt
- **Bullet items**: margin-bottom 0pt (tight packing)

### Page Layout
- **Margins**: Top 0.39" / Bottom 0.33" / Left 0.75" / Right 0.75" (set in generate-pdf.mjs)
- **Format**: Letter (US) default
- **Target**: EXACTLY ONE FULL PAGE

### Section Order (mandatory)
1. Summary
2. Skills
3. Experience
4. Projects
5. Education
6. Awards (omit section entirely if no awards)

## Resume Strategy (WHY these rules exist)

A recruiter spends 6-7 seconds scanning a resume. In that scan they read:
1. Your name and current title (top of page)
2. The Summary paragraph (are you relevant?)
3. Company names and job titles (credibility check)
4. The first bullet of each job (what did you actually do?)
5. Skills section (keyword match for the ATS + human check)

Everything in this resume must be optimized for that 6-second scan AND for a deeper 60-second read if you pass the first filter.

---

## Content Rules (MANDATORY)

**⚠️ CRITICAL: The resume MUST fill the ENTIRE page edge-to-edge. A resume that is 3/4 page is a FAILED resume. If the PDF comes back as 1 page but has visible white space at the bottom, you MUST add more content and regenerate. The bottom of the last section should land within 0.5 inches of the page bottom margin.**

### Minimum Content Requirements (hard minimums, non-negotiable)

These are the MINIMUM amounts of content. If the page has room, ADD MORE.

| Section | Minimum | Target character count |
|---|---|---|
| Summary | 3-4 lines of text | 280-350 characters |
| Skills | EXACTLY 6 labeled categories | 6 lines, each 60-100 chars |
| Experience Job 1 (most recent) | EXACTLY 6 bullets | Each bullet 130-180 chars (total ~900 chars) |
| Experience Job 2 | EXACTLY 6 bullets | Each bullet 130-180 chars (total ~900 chars) |
| Experience Job 3 | EXACTLY 3 bullets | Each bullet 130-180 chars (total ~450 chars) |
| Project 1 | 2 bullets | Each bullet 120-170 chars |
| Project 2 | 2 bullets | Each bullet 120-170 chars |
| Project 3 | 2 bullets | Each bullet 120-170 chars |
| Project 4 | 1 bullet | 120-170 chars |
| Education | Degree + GPA + coursework | 1-2 lines |

**Total minimum**: ~4,200 characters of body content across all sections.

### Mandatory Page-Fill Verification Loop

After generating the PDF, you MUST follow this verification loop:

```
STEP 1: Generate HTML and PDF
STEP 2: Check output — does it say "Pages: 1"?
  - If "Pages: 2" → go to OVERFLOW FIX below
  - If "Pages: 1" → go to STEP 3
STEP 3: The page is 1 page, but IS IT FULL?
  - Count total bullet characters. If total < 4,000 chars → PAGE IS SHORT, go to STEP 4
  - If total >= 4,000 chars → DONE, page is full
STEP 4 (PAGE IS SHORT - fix it):
  Apply these levers IN ORDER until the page fills:
  a. Expand each experience bullet to 150-180 chars (add context, specifics, metrics)
  b. Add a 7th bullet to Job 1 or Job 2 (expand to 7 + 6 + 3 or 6 + 7 + 3)
  c. Add a 3rd bullet to projects (make distribution 3 + 2 + 2 + 1)
  d. Add "Relevant Coursework: Course1, Course2, Course3, Course4" line under Education
  e. Add an Awards section if candidate has any honors
  f. Expand summary to 4 full lines
  g. Increase font-size from 7.5pt to 8pt and line-height from 10pt to 10.5pt in the HTML <style>
  → After EACH lever, regenerate PDF and re-check. Stop when page is full.
```

### Overflow Fix (Pages: 2)

Apply these levers IN ORDER until it fits on 1 page:
1. Tighten bullet text — condense to 130-150 chars each (cut filler words, not substance)
2. Reduce to 5 bullets for Job 2 (keep Job 1 at 6)
3. Cut project 4 entirely (go to 3 projects with 2 + 2 + 2)
4. Shorten summary to 3 lines
5. Reduce font-size from 7.5pt to 7pt and line-height from 10pt to 9.5pt
6. Last resort: reduce name from 18pt to 16pt

**After EACH change, regenerate the PDF. Keep iterating until "Pages: 1" AND the page is visually full.**

### Summary (3-4 lines, mandatory)

The summary is the ONLY section a recruiter reads word-for-word. It must answer three questions instantly:
1. **Who are you?** (title + years + degree)
2. **What have you done?** (2-3 specific achievements with numbers)
3. **Why should I care for THIS role?** (mirror the JD's core need)

**Formula**: `[Title] with [X]+ years building [domain from JD]. [Achievement 1 with metric]. [Achievement 2 with metric]. [Degree] from [School] with expertise in [3-5 JD technologies].`

**Rules:**
- First 8 words must match the JD's role title and seniority
- MUST contain 2-3 hard metrics (latency, throughput, scale, cost savings)
- MUST name 4-5 specific technologies the JD asks for
- **MINIMUM 280 characters for the summary paragraph.** If under 280 chars, the summary is too thin — add another metric or technology.
- NO soft skills ("passionate", "team player", "detail-oriented") — these waste space
- NO objectives ("seeking a position...") — the application itself says that
- Write in fragments, not full sentences, to pack more information per line

**Example (GOOD):**
> Software Engineer with 3+ years building production distributed systems processing millions of daily events across fraud detection and task orchestration. Designed Kafka pipelines cutting compliance processing from 45 min to 12 min for 500K+ daily requests. MS in CS (3.9 GPA) from RIT with deep expertise in Python, Go, Kafka, Redis, and AWS.

**Example (BAD — generic filler):**
> Passionate software engineer seeking a challenging role where I can leverage my skills in a fast-paced environment. Strong communicator and team player with experience in various technologies.

---

### Skills (EXACTLY 6 labeled categories, mandatory)

Skills serve two masters: ATS keyword matching (machine) and credibility check (human).

**Rules:**
- Each category: `**Bold Label:** skill1, skill2, skill3, ...`
- HTML: `<div class="skill-item"><span class="skill-category">Label:</span> skill1, skill2, ...</div>`
- **Front-load JD keywords**: put the most JD-relevant skills first in each category
- **Mirror exact JD phrasing**: if JD says "Apache Kafka" don't write "Kafka" alone; if JD says "CI/CD" don't write "continuous integration"
- **Group strategically**: Categories should map to the JD's requirement sections
- **Never list skills you can't discuss in an interview** — you WILL get asked
- Suggested categories: Languages, AI/ML Frameworks, Data/Pipelines, Cloud/Infrastructure, MLOps/Quality, Architecture
- Adapt category names to JD context (e.g., if JD is heavy on DevOps, rename "Architecture" to "DevOps and Architecture")
- **MINIMUM 5 skills per category line.** Short lines like "Languages: Python, Java" waste space. Write "Languages: Python, Java, Go, C++, SQL, JavaScript, TypeScript"
- Each skill-item line should be 60-100 characters. If a line is under 60 chars, add more relevant skills.

---

### Experience (EXACTLY 15 bullets total: 6 + 6 + 3, mandatory)

Distribute across top 3 jobs by IMPORTANCE, not recency. See `_profile.md` for the explicit job priority and display order. Do NOT assume chronological order.

#### The Narrative Bullet Formula

Every bullet must follow the **PROBLEM-ACTION-RESULT** micro-story structure, NOT the formulaic "Did X using Y, achieving Z" template.

**The 4 elements of a standout bullet:**
1. **Context/Problem** (what was broken, needed, or at stake)
2. **Your specific action** (what YOU decided and built — show ownership)
3. **Technical depth** (architectural choices, not just tool names)
4. **Quantified outcome** (metric that proves it worked)

**How to START bullets (vary these — never repeat the same opener):**
- Ownership: "Own the...", "Led the...", "Drive the..."
- Problem-first: "Diagnosed a...", "Inherited a...", "Noticed that..."
- Took-action: "Decomposed a...", "Redesigned the...", "Replaced..."
- Collaborated: "Partnered with ML team to...", "Led 4-person sub-team through..."
- Scope: "Architected the...", "Designed the..."

**HARD RULES:**
- EVERY bullet MUST contain at least one quantified metric. No exceptions.
- **MINIMUM 130 characters per bullet. MAXIMUM 180 characters.** If a bullet is under 130 chars, it's too short — add more context, specifics, or a second metric. If over 180 chars, condense.
- Each bullet should wrap to approximately 1.5-2 lines on the page. Single-line bullets waste space and signal low depth.
- Never start 2 consecutive bullets with the same word
- Never use passive voice ("was built", "was improved")
- Never write duty-descriptions ("Responsible for...", "Tasked with...")
- First bullet of each job = your BIGGEST impact at that company (recruiter reads this one)
- **SELF-CHECK**: After writing all bullets, count characters for each. If any bullet is under 130 chars, rewrite it longer. If total experience content is under 2,200 chars, the page WILL be short — add depth to every bullet.

**Examples of GOOD vs BAD bullets:**

| BAD (formulaic, shallow) | GOOD (narrative, deep) |
|---|---|
| Built microservices using Python and Kafka, reducing latency by 40% | Decomposed a monolithic API that timed out under load into 6 independently scalable services with Redis write-through caching, bringing p95 from 1.8s to 650ms |
| Improved system observability by building Grafana dashboards | Built real-time Grafana observability: per-state throughput panels, error-rate heatmaps, and PagerDuty routing that cut mean time to detect from 15 min to 6 min |
| Optimized database queries to improve performance | Profiled peak-hour query degradation, introduced compound indexes and TTL-based Redis eviction, achieving consistent sub-500ms lookups while cutting DB costs by 15% |
| Worked on CI/CD pipeline improvements | Led CI/CD migration from manual deploys to GitHub Actions with staging gates, improving deployment velocity by 35% and reducing rollback frequency by 20% |
| Built data pipeline using Kafka for processing events | Designed event ingestion layer: Kafka topics with schema registry and exactly-once semantics, reliably processing 2M+ daily verification events at 99.97% delivery |

**How to expand cv.md bullets that lack depth:**
- If cv.md says "built X" — ask WHY: what was wrong before? What did you replace?
- If cv.md has no metric — add a reasonable one based on context (scale, speed, reliability, cost)
- If cv.md is vague on tech — add the specific architectural pattern (write-through cache, circuit breaker, dead-letter queue)
- If cv.md mentions a team — add team size and your leadership role
- Reframe with JD vocabulary: cv.md says "deployment" + JD says "CI/CD" -> write "CI/CD pipeline"
- **NEVER invent experience.** Only expand and reframe what exists in cv.md

**Job title format in HTML:**
```html
<div class="job-header">
  <div class="job-title-line"><span class="job-title">Title</span> | <span class="job-company">Company</span></div>
  <span class="job-period">Month Year - Present</span>
</div>
```

---

### Projects (4 projects, EXACTLY 7 bullets total: 2 + 2 + 2 + 1, mandatory)

Projects are your secret weapon — they show initiative and let you fill skill gaps the JD needs.

**Rules:**
- Include 3 most JD-relevant projects from cv.md PLUS 1 generated project using the candidate's real skills
- The generated project MUST target a gap: if the JD asks for something not in Experience (e.g., RL tooling, RAG pipelines), build a project that demonstrates it
- Each project MUST use bullet points (`<ul><li>`), NOT paragraphs
- Project bullets follow the same narrative quality rules as Experience bullets
- Tech stack in italics next to the project name — these are keyword goldmines for ATS

**How to write project bullets that stand out:**
- Don't just describe what it does — describe a HARD PROBLEM you solved
- Include at least one metric or specific technical outcome per bullet
- Show architectural decisions: "chose X over Y because..."
- If it's open source, mention stars/forks/downloads if notable
- **MINIMUM 120 characters per project bullet.** Short project bullets like "Built a serverless app on AWS" are unacceptable — add what problem it solved, what architecture you chose, and what the outcome was.

**Project format in HTML:**
```html
<div class="project">
  <div class="project-header">
    <div class="project-title-line">
      <span class="project-title">Project Name</span>
      <span class="project-tech">Tech1, Tech2, Tech3</span>
    </div>
  </div>
  <ul>
    <li>Bullet with metric...</li>
    <li>Bullet with metric...</li>
  </ul>
</div>
```

---

### Education (mandatory)
- School name bold, degree, date right-aligned
- GPA (if 3.5+), relevant coursework on second line
- Coursework should mirror JD requirements (e.g., JD mentions distributed systems -> list "Distributed Systems" course)

### Awards (optional)
- Include only if candidate has relevant awards/honors
- Omit the entire section if none

---

## Keyword Injection Strategy (ethical, truth-based)

**The goal**: maximize ATS keyword match rate WITHOUT inventing skills.

**Technique: Vocabulary Mirroring**
Take real experience from cv.md and rewrite it using the JD's exact phrasing:

| cv.md says | JD says | Write as |
|---|---|---|
| "LLM workflows with retrieval" | "RAG pipelines" | "RAG pipeline design and LLM orchestration" |
| "observability, evals, error handling" | "MLOps" | "MLOps: model evals, observability, error handling" |
| "collaborated with team" | "stakeholder management" | "stakeholder management across engineering and product" |
| "deployed services" | "CI/CD pipelines" | "automated CI/CD pipelines with staging gates" |
| "wrote tests" | "quality assurance" | "quality assurance frameworks with 85% coverage" |

**Keyword Distribution (where to place them):**
- **Summary**: top 5 keywords from JD (recruiter reads this first)
- **First bullet of each job**: 2-3 JD keywords (recruiter scans these)
- **Skills section**: ALL JD-mentioned technologies the candidate knows
- **Project tech stacks**: keywords that didn't fit naturally in Experience

**NEVER add skills the candidate does not have. Only reformulate real experience with the JD's exact vocabulary.**

---

## Anti-Patterns (things that KILL a resume)

1. **"Responsible for..." bullets** — describes duties, not achievements. Rewrite as what you BUILT/CHANGED/IMPROVED.
2. **Wall of buzzwords** — "Leveraged synergies to drive innovation" means nothing. Use specific technologies and metrics.
3. **Inconsistent tense** — current job = present tense, past jobs = past tense. Never mix within a job.
4. **Orphan sections** — a "Projects" section with 1 project looks weak. Always have 3-4.
5. **Generic summary** — if you can swap in any other person's name and it still works, it's too generic.
6. **Same verb repeated** — if 4 bullets start with "Built", the reader zones out. Vary: built, designed, led, diagnosed, decomposed, architected, owned.
7. **Missing metrics** — "improved performance" means nothing. "reduced p95 from 1.8s to 650ms" means everything.
8. **Technology laundry list** — "Python, Java, Go, C++, Ruby, Rust, Scala, Kotlin" — if you list 15 languages nobody believes you know any of them well. Keep it honest and focused.

## Template HTML

Usar el template en `cv-template.html`. Reemplazar los placeholders `{{...}}` con contenido personalizado:

| Placeholder | Contenido |
|-------------|-----------|
| `{{LANG}}` | `en` o `es` |
| `{{PAGE_WIDTH}}` | `8.5in` (letter) o `210mm` (A4) |
| `{{NAME}}` | (from profile.yml) |
| `{{PHONE}}` | (from profile.yml — include with its separator only when `profile.yml` has a non-empty `phone` value; omit both `<span>` and `<span class="separator">` otherwise) |
| `{{EMAIL}}` | (from profile.yml) |
| `{{LINKEDIN_URL}}` | [from profile.yml] |
| `{{LINKEDIN_DISPLAY}}` | [from profile.yml] |
| `{{PORTFOLIO_URL}}` | [from profile.yml] (o /es según idioma) |
| `{{PORTFOLIO_DISPLAY}}` | [from profile.yml] (o /es según idioma) |
| `{{LOCATION}}` | [from profile.yml] |
| `{{SECTION_SUMMARY}}` | Professional Summary / Resumen Profesional |
| `{{SUMMARY_TEXT}}` | Summary personalizado con keywords |
| `{{SECTION_COMPETENCIES}}` | Core Competencies / Competencias Core |
| `{{COMPETENCIES}}` | `<span class="competency-tag">keyword</span>` × 6-8 |
| `{{SECTION_EXPERIENCE}}` | Work Experience / Experiencia Laboral |
| `{{EXPERIENCE}}` | HTML de cada trabajo con bullets reordenados |
| `{{SECTION_PROJECTS}}` | Projects / Proyectos |
| `{{PROJECTS}}` | HTML de top 3-4 proyectos |
| `{{SECTION_EDUCATION}}` | Education / Formación |
| `{{EDUCATION}}` | HTML de educación |
| `{{SECTION_CERTIFICATIONS}}` | Certifications / Certificaciones |
| `{{CERTIFICATIONS}}` | HTML de certificaciones |
| `{{SECTION_SKILLS}}` | Skills / Competencias |
| `{{SKILLS}}` | HTML de skills |

## Canva CV Generation (optional)

If `config/profile.yml` has `canva_resume_design_id` set, offer the user a choice before generating:
- **"HTML/PDF (fast, ATS-optimized)"** — existing flow above
- **"Canva CV (visual, design-preserving)"** — new flow below

If the user has no `canva_resume_design_id`, skip this prompt and use the HTML/PDF flow.

### Canva workflow

#### Step 1 — Duplicate the base design

a. `export-design` the base design (using `canva_resume_design_id`) as PDF → get download URL
b. `import-design-from-url` using that download URL → creates a new editable design (the duplicate)
c. Note the new `design_id` for the duplicate

#### Step 2 — Read the design structure

a. `get-design-content` on the new design → returns all text elements (richtexts) with their content
b. Map text elements to CV sections by content matching:
   - Look for the candidate's name → header section
   - Look for "Summary" or "Professional Summary" → summary section
   - Look for company names from cv.md → experience sections
   - Look for degree/school names → education section
   - Look for skill keywords → skills section
c. If mapping fails, show the user what was found and ask for guidance

#### Step 3 — Generate tailored content

Same content generation as the HTML flow (Steps 1-11 above):
- Rewrite Professional Summary with JD keywords + exit narrative
- Reorder experience bullets by JD relevance
- Select top competencies from JD requirements
- Inject keywords naturally (NEVER invent)

**IMPORTANT — Character budget rule:** Each replacement text MUST be approximately the same length as the original text it replaces (within ±15% character count). If tailored content is longer, condense it. The Canva design has fixed-size text boxes — longer text causes overlapping with adjacent elements. Count the characters in each original element from Step 2 and enforce this budget when generating replacements.

#### Step 4 — Apply edits

a. `start-editing-transaction` on the duplicate design
b. `perform-editing-operations` with `find_and_replace_text` for each section:
   - Replace summary text with tailored summary
   - Replace each experience bullet with reordered/rewritten bullets
   - Replace competency/skills text with JD-matched terms
   - Replace project descriptions with top relevant projects
c. **Reflow layout after text replacement:**
   After applying all text replacements, the text boxes auto-resize but neighboring elements stay in place. This causes uneven spacing between work experience sections. Fix this:
   1. Read the updated element positions and dimensions from the `perform-editing-operations` response
   2. For each work experience section (top to bottom), calculate where the bullets text box ends: `end_y = top + height`
   3. The next section's header should start at `end_y + consistent_gap` (use the original gap from the template, typically ~30px)
   4. Use `position_element` to move the next section's date, company name, role title, and bullets elements to maintain even spacing
   5. Repeat for all work experience sections
d. **Verify layout before commit:**
   - `get-design-thumbnail` with the transaction_id and page_index=1
   - Visually inspect the thumbnail for: text overlapping, uneven spacing, text cut off, text too small
   - If issues remain, adjust with `position_element`, `resize_element`, or `format_text`
   - Repeat until layout is clean
d. Show the user the final preview and ask for approval
e. `commit-editing-transaction` to save (ONLY after user approval)

#### Step 5 — Export and download PDF

a. `export-design` the duplicate as PDF (format: a4 or letter based on JD location)
b. **IMMEDIATELY** download the PDF using Bash:
   ```bash
   curl -sL -o "output/cv-{candidate}-{company}-canva-{YYYY-MM-DD}.pdf" "{download_url}"
   ```
   The export URL is a pre-signed S3 link that expires in ~2 hours. Download it right away.
c. Verify the download:
   ```bash
   file output/cv-{candidate}-{company}-canva-{YYYY-MM-DD}.pdf
   ```
   Must show "PDF document". If it shows XML or HTML, the URL expired — re-export and retry.
d. Report: PDF path, file size, Canva design URL (for manual tweaking)

#### Error handling

- If `import-design-from-url` fails → fall back to HTML/PDF pipeline with message
- If text elements can't be mapped → warn user, show what was found, ask for manual mapping
- If `find_and_replace_text` finds no matches → try broader substring matching
- Always provide the Canva design URL so the user can edit manually if auto-edit fails

## Post-generación

Actualizar tracker si la oferta ya está registrada: cambiar PDF de ❌ a ✅.

