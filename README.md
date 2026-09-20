# BidLens

BidLens extracts procurement fields from mixed RFP packets containing HTML and PDF documents, stores the result as JSON, and provides a small browser dashboard for searching and reviewing the extracted bids.

## Quick start

```powershell
cd "C:\Users\Shiva Teja\Downloads\Emplay Assignment"
python -m pip install -r requirements.txt
python app.py --serve --port 8765
```

Open <http://127.0.0.1:8765>. The API is available at `/api/bids`, `/api/health`, `/api/requirements`, and `/api/export`. The dashboard includes the same live evaluation checklist so a reviewer can verify requirement coverage.

## Rebuild the JSON

The extractor scans every `Bid*` folder, parses HTML with Python's standard-library `HTMLParser`, and extracts PDF text with `pypdf`:

```powershell
python app.py --extract
```

The generated file is `structured_bids.json`. The checked-in sample output contains the curated field mapping for the two supplied bid packets; use `--extract` when processing a new packet or changing the source files.

For a deployment platform that provides a `PORT` environment variable:

```powershell
python app.py --serve --host 0.0.0.0 --port $env:PORT
```

The included `Dockerfile` is ready for container hosting:

```powershell
docker build -t bidlens .
docker run --rm -p 8080:8080 bidlens
```

The container exposes the dashboard at `http://localhost:8080`.

## Project structure

- `app.py` - extraction engine, HTTP API, and dashboard
- `structured_bids.json` - structured output for the supplied documents
- `Bid1/`, `Bid2/` - source HTML and PDF bid packets
- `requirements.txt` - runtime dependency
- `Dockerfile` - production container definition
- `.dockerignore` - container build exclusions
- `render.yaml` - Render deployment blueprint

## Validation

```powershell
python -m py_compile app.py
python -c "import json; print(len(json.load(open('structured_bids.json', encoding='utf-8'))))"
```

The extractor is intentionally deterministic and offline. It uses labeled fields from the BidNet HTML page, regular expressions for dates and identifiers, and full text from the related PDFs. This makes the result repeatable and easy to inspect: values are reported as `Not stated` when the source packet does not provide enough evidence.

## Requirement coverage

| Assignment requirement | Status |
|---|---|
| Parse HTML documents | Complete: standard-library `HTMLParser` |
| Parse PDF documents | Complete: `pypdf` |
| Map the requested fields | Complete: every requested field is emitted, with `Not stated` when evidence is absent |
| JSON output | Complete: `structured_bids.json` and `/api/export` |
| Search/review interface | Complete: responsive dashboard and `/api/bids` |
| Repeatable extraction | Complete: `python app.py --extract` |
| Text processing method | Complete: labeled-field extraction, regular expressions, HTML parsing, and PDF text extraction |
| Public cloud deployment | Deployment-ready, but a public URL still requires running the Docker image on a host such as Render, Railway, Azure, or AWS |