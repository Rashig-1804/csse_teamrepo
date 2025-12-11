This repository is a Jekyll-based learning site with helper scripts for converting Jupyter notebooks and DOCX files into blog posts. The file below gives targeted guidance for AI coding agents to be immediately productive.

Key architecture
- **Site engine:** Jekyll (Ruby) — primary build and serve via `bundle exec jekyll build` / `jekyll serve` (see `Makefile`).
- **Notebook pipeline:** Python scripts in `scripts/` convert `_notebooks/**/*.ipynb` -> `_posts/*_IPYNB_2_.md` using `nbconvert` and `nbformat` (`scripts/convert_notebooks.py`).
- **Front matter convention:** The first cell of each notebook, when it starts with `---`, is treated as YAML front matter and removed from the exported notebook content (see `convert_notebooks.py::extract_front_matter`).
- **Mermaid diagrams:** Markdown `~~~mermaid` blocks are converted to PNG via `mmdc`. Generated images are written to `assets/mermaid/` using a content hash.
- **Progress UI:** `scripts/progress_bar.py` wraps the third-party `progress` package (`progress.bar.ChargingBar`) to show conversion progress.

Developer workflows & important commands
- Install Python deps used by conversion and utilities:

  ```bash
  python3 -m pip install --user -r requirements.txt
  # or (recommended) create a virtualenv and then:
  python3 -m venv .venv
  source .venv/bin/activate
  pip install -r requirements.txt
  ```

- Install mermaid-cli (used by `convert_notebooks.py`):

  ```bash
  npm i -g @mermaid-js/mermaid-cli
  # ensures `mmdc` is on PATH
  ```

- Run the project (local dev server):

  ```bash
  make serve    # starts Jekyll server and watches _notebooks and _docx
  make convert  # convert notebooks and docx into _posts
  ```

Why you saw `ModuleNotFoundError: No module named 'progress'`
- The Python package `progress` is declared in `requirements.txt`, but the environment used by `make` wasn't running with those packages installed. Installing the requirements (see commands above) fixes the error.
- Quick fix example:

  ```bash
  python3 -m pip install --user -r requirements.txt
  make convert   # or make serve
  ```

Project-specific patterns to follow
- Notebook exports: exported markdown filenames are created by replacing `*.ipynb` -> `*_IPYNB_2_.md` and placed under `_posts`. Do not edit generated files directly — edit the source notebook in `_notebooks/`.
- The convert script removes the first notebook cell after reading front matter; that cell must contain only YAML front matter if present.
- JavaScript cells inside notebooks are indicated with `%%js` in `python` code blocks; the converter replaces those with fenced `javascript` blocks (`fix_js_code_blocks` in `convert_notebooks.py`).
- Mermaids: the converter expects markdown mermaid blocks starting with `~~~mermaid` (not fenced backticks); it converts them to images and rewrites the markdown to reference the generated PNG.
- Concurrency: the conversion uses `concurrent.futures.ProcessPoolExecutor` and CPU cores. Avoid editing long-running conversion code without testing on a few notebooks first.

Integration points and external dependencies
- `requirements.txt` (Python): `nbconvert`, `nbformat`, `pyyaml`, `progress`, etc. Ensure these are installed in the Python environment used by make.
- `mmdc` (mermaid-cli): required for diagram rendering.
- Ruby/Jekyll/Gems: `bundle install` / `bundle exec jekyll serve` (Makefile wraps these). The build depends on the `Gemfile` selected by theme targets (e.g., `use-minima`).

Files to inspect when making changes
- `Makefile` — controls serve/build/watch and the `convert` target.
- `scripts/convert_notebooks.py` — conversion logic, front matter extraction, mermaid handling, and progress reporting.
- `scripts/progress_bar.py` — thin wrapper around `progress.bar.ChargingBar` (dependency: `progress`).
- `requirements.txt` — Python dependencies.
- `_notebooks/` and `_posts/` — source notebooks and generated posts.

If you plan to modify conversion behavior
- Test locally on a small notebook before running the full conversion (the code uses all CPUs by default).
- Keep `extract_front_matter` and `process_mermaid_cells` behavior in mind when changing notebook structure.

If anything in this file is unclear or you want additional examples (e.g., exact test commands or a small test notebook), tell me which area to expand.
