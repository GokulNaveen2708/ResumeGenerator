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
- **Contact line**: 7.25pt, centered, pipe (|) separators, email and portfolio links hyperlinked in blue (#1155CC)
- **Section headers**: 8pt, bold, UPPERCASE, blue (#1155CC), with blue underline rule beneath
- **Job titles**: 8pt, bold | Company name in italic | Date right-aligned on same line
- **Body text and bullets**: 7.25pt
- **Bullet indent**: 0.3" left with 0.3" hanging indent (set via `padding-left: 0.3in` on `<ul>`)

### Spacing
- **Line spacing**: 10.25pt exact for all body text
- **Paragraph spacing**: margin-top 2pt / margin-bottom 0pt for sections
- **Job headers**: margin-top 1.5pt / margin-bottom 0pt (tighter)
- **Project items**: margin-top 1pt / margin-bottom 0pt

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

## Content Rules (MANDATORY minimums)

**The resume MUST fill the ENTIRE page. An empty bottom quarter is UNACCEPTABLE.**

### Summary (3-4 lines, mandatory)
- Keyword-dense paragraph using JD vocabulary
- MUST include: years of experience, degree, 4-5 key technologies from JD, target role type
- MUST include 2-3 quantified achievements (e.g., "reduced latency by 64%", "processing 500K+ daily requests")
- NO generic filler. Every sentence must contain a metric or specific technology

### Skills (5-6 labeled categories, mandatory)
- Each category uses a **bold label** followed by colon and comma-separated skills
- HTML: `<div class="skill-item"><span class="skill-category">Label:</span> skill1, skill2, ...</div>`
- MUST include all JD-mentioned technologies the candidate actually knows
- Categories to consider: Languages, Backend/Frameworks, Databases/Storage, Cloud/Infrastructure, AI/ML, Tools/Practices

### Experience (5-6 bullets per job, mandatory)
- **EVERY bullet MUST contain a quantified metric.** No exceptions. Examples:
  - Response time: "reduced p95 latency from 1.8s to 650ms"
  - Scale: "processing 500K+ daily requests"
  - Improvement: "reducing incidents by 25%"
  - Team/scope: "across 12 microservices", "for 40+ engineering teams"
  - Cost: "reducing infrastructure costs by 15%"
  - Time: "cutting processing from 45 min to 12 min"
- If cv.md has a bullet without a metric, ADD a reasonable metric based on context
- Expand bullets by:
  - Breaking complex bullets into two focused bullets
  - Adding collaboration, code review, testing, documentation context
  - Reframing with JD vocabulary (e.g., cv.md says "deployment" + JD says "CI/CD" → write "CI/CD pipeline")
  - Adding team size, business impact, or scale context
- **NEVER invent experience.** Only expand and reframe what exists in cv.md
- Job title format in HTML:
  ```html
  <div class="job-header">
    <div class="job-title-line"><span class="job-title">Title</span> - <span class="job-company">Company</span></div>
    <span class="job-period">Month Year - Present</span>
  </div>
  ```

### Projects (3-4 projects + 1 generated, mandatory)
- Include 3-4 most JD-relevant projects from cv.md
- **ALWAYS generate one additional JD-aligned project** using candidate's real skills
- Each project MUST use bullet points (`<ul><li>`), NOT paragraphs
- 2-3 bullets per project, each with a metric or specific technical outcome
- Project format in HTML:
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

### Education (mandatory)
- School name bold, degree, date right-aligned
- GPA, relevant coursework on second line

### Awards (optional)
- Include only if candidate has relevant awards/honors
- Omit the entire section if none

## Page-Filling Strategy

After generating HTML, check the PDF output from generate-pdf.mjs:

### If "Pages: 1" but page looks SHORT:
1. Increase body font-size in the HTML `<style>`: try 7.5pt, then 8pt
2. Increase line-height: try 10.5pt, then 11pt
3. Add more bullets to work experience (expand existing ones to 6 per job)
4. Add a 3rd bullet to each project
5. Add "Relevant Coursework" line under Education
6. Add an Awards section if applicable

### If "Pages: 2" (overflow):
1. Decrease body font-size: try 7pt
2. Decrease line-height: try 10pt
3. Trim to 5 bullets per job (remove weakest)
4. Reduce project bullets to 2 each
5. Shorten summary to 3 lines
6. Last resort: reduce name to 16pt

### CRITICAL: You MUST regenerate the PDF after CSS adjustments.
Run `node generate-pdf.mjs` again after each change. Keep iterating until "Pages: 1" AND the page is visually full. If you cannot verify visually, err on the side of MORE content (6 bullets per job, 3 per project).

## Estrategia de keyword injection (ético, basado en verdad)

Ejemplos de reformulación legítima:
- JD dice "RAG pipelines" y CV dice "LLM workflows with retrieval" → cambiar a "RAG pipeline design and LLM orchestration workflows"
- JD dice "MLOps" y CV dice "observability, evals, error handling" → cambiar a "MLOps and observability: evals, error handling, cost monitoring"
- JD dice "stakeholder management" y CV dice "collaborated with team" → cambiar a "stakeholder management across engineering, operations, and business"

**NUNCA añadir skills que el candidato no tiene. Solo reformular experiencia real con el vocabulario exacto del JD.**

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

