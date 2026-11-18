# ResumeIQ AI

ResumeIQ AI is a local-first resume analysis MVP for exploring resume structure, catalogued skills, explainable readiness heuristics, job-description overlap, and actionable next steps. It does **not** estimate hiring probability or provide an official ATS score.

## What works in this version

- A complete local demo path: start the API and frontend, open the site without signing in, upload a PDF or DOCX, review extracted details and categorized skills, and download a report.
- FastAPI API with automatic browser guest sessions, signed bearer tokens, per-session history, and ownership checks. No sign-in or registration is required.
- PDF and DOCX text extraction with file-signature checks and a 5 MB limit. Image-only PDFs use Tesseract OCR (Windows Tesseract installation required). Files are parsed in memory; extracted analysis results (not uploaded documents or full extracted text) are saved.
- Section and categorized skill detection with source-text evidence and normalized aliases, including common languages such as C, C++, C#, Java, JavaScript, Python, Go, R, and Rust. The catalog also covers popular frameworks, data tools, databases, and cloud/DevOps tools.
- Deterministic 100-point readiness breakdown, evidence-based recommendations, job skill gap analysis, dashboard history, and paginated PDF reports that wrap long skill evidence and extracted resume sections.
- Protected API endpoints for professional summary drafts and conservative bullet cleanup that does not invent achievements.
- React + TypeScript dashboard connected to the API, with responsive layout, upload, history, charts, and report download.
- SQLite by default, with PostgreSQL supported through `DATABASE_URL`.

The automated checks cover PDF and DOCX analysis, scanned-PDF OCR fallback, language/skill aliases, guest data isolation, history, and detailed PDF report contents. The same upload-to-report workflow has also been manually exercised with a generated sample resume PDF.

The optional `requirements-ml.txt` installs Sentence Transformers and the configured `all-MiniLM-L6-v2` CPU model. Without that optional package, matching explicitly uses a TF-IDF lexical similarity fallback (`semantic_method: tfidf-fallback`). The model weights may download on first use. Neither method is a hiring probability.

## Project structure

```text
backend/
  app/
    core.py
    database.py
    main.py
    models.py
    services/
      analysis.py
      document_parser.py
  tests/
    test_analysis.py
    test_api.py
    test_document_parser.py
frontend/
  src/
    App.tsx
    main.tsx
    report.css
    style.css
```

## Windows setup

From the project root in PowerShell:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn backend.app.main:app --reload --env-file .env
```

In another PowerShell window:

```powershell
Set-Location .\frontend
npm install
npm run dev
```

Open the Vite URL (normally `http://localhost:5173`). The app starts a guest workspace automatically and keeps its access token in this browser's local storage for up to 30 days. Clearing browser storage or switching browsers creates a new workspace; guest history cannot be recovered. Configure a strong `RESUMEIQ_SECRET_KEY` in `.env`. Set `APP_ENV=production` in deployed environments; startup then rejects the default or short signing secret. Set `DATABASE_URL` to a PostgreSQL SQLAlchemy URL and install the matching driver (for example `psycopg[binary]`) to use PostgreSQL. Tables are created at startup in this MVP; Alembic migrations are not yet included.

For scanned/image-only PDFs, install the Tesseract OCR engine on Windows (for example, `winget install --id UB-Mannheim.TesseractOCR`) and restart the API. Text-based PDFs and DOCX files do not require OCR.

To enable CPU Sentence Transformers matching:

```powershell
python -m pip install -r requirements-ml.txt
```

## Tests and API docs

```powershell
python -m pytest backend\tests -q
Set-Location .\frontend
npm run build
```

Interactive API documentation: `http://localhost:8000/docs`  
Health check: `http://localhost:8000/api/v1/health`

Main routes: `/api/v1/auth/guest`, `/api/v1/auth/me`, `/api/v1/analysis/{analyze,history}`, `/api/v1/dashboard/stats`, `/api/v1/jobs/analyze`, `/api/v1/resume/{generate-summary,optimize-bullet}`, and `/api/v1/reports/{analysis_id}/pdf`.

## Limitations

This is a functioning foundation, not the full product scope from the supplied specification. It does not yet include Alembic migrations, password reset/email verification, persisted job descriptions, comparison, role recommendations, frontend optimizer/settings/history pages, rate limiting, deployment manifests, or broad integration/accessibility tests. Skill dictionaries and section heuristics are intentionally transparent and finite. Summary/bullet tools are conservative deterministic drafts rather than an external LLM. Secure production deployment also requires HTTPS, production secret management, appropriate database backups/retention, and strict production CORS settings.
