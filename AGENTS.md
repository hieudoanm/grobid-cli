# AGENTS.md

## Project Overview

grobid-cli converts PDF articles to Markdown via GROBID TEI XML. Built upon [GROBID](https://github.com/grobidOrg). The CLI is designed to make PDFs AI-friendly by producing clean, structured Markdown with metadata in frontmatter.

**Problem:** AI models struggle with raw PDFs (messy structure, headers/footers, citations).  
**Solution:** Use GROBID to extract structured content and convert to clean Markdown.

## Architecture

### Source Structure
```
src/grobid/
├── __init__.py      # Public API
├── main.py          # Entry point
├── cli.py           # CLI logic (convert_pdf_to_md)
├── config.py        # Configuration (ports, images, paths, NS, META_FIELDS)
├── utils.py         # Logging (timestamped, levels: fatal/error/warn/info/debug/trace) & helpers
├── paths.py         # Path resolution (PDF_DIR, TEI_DIR, MD_DIR)
├── grobid_client.py # Docker GROBID management & PDF→TEI via grobid-client-python
├── metadata.py      # Metadata extraction from TEI
└── render.py        # TEI → Markdown conversion (inline, blocks, tables, lists, figures)
```

### Pipeline

1. **Auto-start GROBID**: If not running, starts `grobid/grobid:0.9.1-crf` in Docker on port 8070
2. **PDF → TEI**: Uses `grobid-client-python` to call `processFulltextDocument` service
3. **TEI → Markdown**: Parses TEI XML with lxml, extracts metadata/title/authors/abstract/body/references, renders to Markdown with YAML frontmatter

### Single Entry Point

- Command: `grobid` (via `pyproject.toml` scripts: `grobid = "grobid.main:main"`)
- Behavior: Converts all PDFs in `pdf/` dir to TEI in `tei/` and Markdown in `md/`
- Resumable: Skips existing TEI and Markdown files
- Auto-creates directories

## Environment Variables

- `GROBID_DATA_DIR` or `ARTICLES_DATA_DIR`: Data directory (default: cwd)
- `GROBID_PORT`: GROBID port (default: 8070)
- `GROBID_IMAGE`: Docker image (default: `grobid/grobid:0.9.1-crf`)
- `GROBID_CONTAINER`: Container name (default: `grobid-cli`)

## Dependencies

- `lxml>=5.0` - XML parsing
- `grobid-client-python` (optional extra `[grobid]`) - PDF processing
- `pyinstaller` (dev) - Binary builds
- `pytest`, `ruff`, `black` (dev) - Testing/linting/formatting

## Build & Test

```bash
make lint    # ruff check src tests
make format  # black + ruff --fix
make test    # pytest with coverage
make build   # pyinstaller --onefile --name grobid src/grobid/main.py
```

## GitHub Actions

CI workflow (`.github/workflows/ci.yaml`) runs lint, test, build on push/PR to main/master. On push to main/master, deletes old "latest" release and publishes new binary to GitHub Releases tagged as "latest".

## Code Style

- Python 3.10+
- Type hints where appropriate
- Timestamped logging with levels (fatal, error, warn, info, debug, trace)
- Keep files under ~200 lines
- Minimal, focused implementation
