# How to Set Up ai-job-search-india Locally

Give this file to Claude Code (or follow it manually) to set up the
[ai-job-search-india](https://github.com/upendra25312/ai-job-search-india) repo
on a new machine. Instruction to Claude: work through the sections in order,
checking what's already installed before installing anything, and ask the
user before doing anything destructive (force pushes, deleting remotes, etc.).

**Before running the setup interview (section 3), Claude MUST ask the user the
two questions in section 2.5 and wait for answers.** Do not populate any profile
files until both are resolved.

## 0. Clone the repo (skip if already cloned)

```bash
git clone https://github.com/upendra25312/ai-job-search-india.git
cd ai-job-search-india
```

## 1. Check prerequisites

Check each of these before installing — don't reinstall something that's
already present:

```bash
claude --version        # Claude Code CLI
py --version || python --version || python3 --version   # Python 3.10+
bun --version            # Bun
lualatex --version       # LaTeX (MiKTeX/TeX Live)
pdftotext -v             # poppler (optional, for ATS check in /apply)
```

### Install Claude Code (if missing)

```bash
npm install -g @anthropic-ai/claude-code
```

Requires an Anthropic API key or a Claude Pro/Team subscription.

### Install Python 3.10+ (if missing)

Only needed for the salary lookup tool. Install from https://python.org or
your OS package manager.

### Install Bun (if missing)

Runs the TypeScript job-portal search CLIs.

- macOS/Linux:
  ```bash
  curl -fsSL https://bun.sh/install | bash
  ```
- Windows PowerShell:
  ```powershell
  powershell -ExecutionPolicy Bypass -c "irm https://bun.sh/install.ps1 | iex"
  ```
  or `winget install Oven-sh.Bun`. Restart the terminal afterward.

### Install LaTeX (if missing)

Needed to compile the generated CV/cover letter `.tex` files to PDF.

- **Windows:** [MiKTeX](https://miktex.org/download) (Basic installer is fine)
- **macOS:** [MacTeX](https://tug.org/mactex/) (or TinyTeX for a lighter install)
- **Linux:** `sudo apt install texlive-full` or `sudo dnf install texlive-scheme-full`

The CV compiles with `lualatex` (pdflatex often fails on modern MiKTeX with
`fontawesome5` font-expansion errors). The cover letter compiles with
`xelatex` because `cover.cls` requires `fontspec` for its custom fonts.

**Windows Basic MiKTeX only** — turn off the GUI install-prompt so
non-interactive terminals (including Claude Code) don't block on it:

```powershell
initexmf --admin --set-config-value=[MPM]AutoInstall=1
initexmf --set-config-value=[MPM]AutoInstall=1
```

Run the first line from an elevated PowerShell only if MiKTeX was installed
for all users; the second covers a per-user install. Running both is harmless.

Optional — pre-install the template packages instead of relying on
on-the-fly installs:

```powershell
mpm --admin --install=moderncv --install=fontawesome5 --install=fontawesome6 --install=academicons --install=import --install=luatexbase --install=pgf --install=titlesec --install=textpos --install=xltxtra --install=xunicode --install=cite --install=realscripts --install=needspace
```

(macOS/Linux equivalent via `tlmgr install <same package list>`.)

### Install poppler / pdftotext (optional)

`/apply` uses this for an ATS parseability check on the compiled CV PDF. If
missing, `/apply` just skips that check with a warning.

- macOS: `brew install poppler`
- Debian/Ubuntu: `sudo apt install poppler-utils`
- Windows: `choco install poppler`

## 2. Install job search CLI dependencies

Run from the repository root (requires Bun):

- PowerShell:
  ```powershell
  $tools = @("jobbank-search", "jobdanmark-search", "jobindex-search", "jobnet-search", "linkedin-search", "freehire-search")
  foreach ($tool in $tools) {
    Push-Location ".agents/skills/$tool/cli"
    bun install
    Pop-Location
  }
  ```
- Bash/zsh/Git Bash:
  ```bash
  for tool in jobbank-search jobdanmark-search jobindex-search jobnet-search linkedin-search freehire-search; do
    (cd .agents/skills/$tool/cli && bun install)
  done
  ```

`linkedin-search` and `freehire-search` have zero runtime dependencies and
run with plain `bun`; installing for them only pulls TypeScript dev types.

If you're outside Denmark, generate an equivalent search skill for a local
job board with `/add-portal` inside Claude Code — it scaffolds the same CLI
structure for any public portal.

## 2.5 Information Claude must collect before setup

**Instruction to Claude:** before running `/setup` (section 3), ask the user both
questions below and wait for answers. Do not read documents into, or write, any
profile file until both are resolved.

### a. Where should the personalized data be stored?

`/setup` writes personal data — name, contact details, employment history,
education, salary expectations — into **tracked** files. Anything committed and
pushed to a public repo is visible to everyone. Ask the user which they want:

- **Make this repo private** — `gh repo edit <owner/repo> --visibility private`.
  Only works if the repo is **not a fork** (GitHub blocks making a fork private).
- **New private repo** (recommended when the current `origin` is a public fork) —
  create a fresh private repository, set it as `origin`, and keep the public
  template as a separate remote named `template` or `upstream`. Full steps in
  section 7.
- **Local only** — keep every profile commit local and never push it.

If `origin` is already a **private, non-fork** repository, say so and move on —
no change needed.

### b. Which career documents can the user provide?

Ask the user to drop whatever they have into `documents/` **before** setup runs
(see `documents/README.md` for the full spec). Accepted: text-based `.pdf`,
`.tex`, `.md`, `.txt` — **not** `.docx`, and **not** scanned images.

| Folder | What goes there | Priority |
| ------ | --------------- | -------- |
| `documents/cv/` | Most complete master CV (not a role-tailored variant) | Strongly recommended |
| `documents/linkedin/` | LinkedIn profile export — Profile → More → Save to PDF | Recommended |
| `documents/diplomas/` | Degree certificates / transcripts | Optional |
| `documents/references/` | Reference letters | Optional |

Wait until the user confirms the files are in place, then use `/setup` Path A. If
the user has no documents at all, fall back to Path C (interview mode) and ask the
profile questions conversationally.

## 3. Run the setup interview

Start Claude Code in the repo root:

```bash
claude
```

Then inside Claude Code, run:

```
/setup
```

This offers three paths (all produce the same result — fully populated
profile files):

- **Path A (documents folder):** add CV, LinkedIn export, diplomas,
  references, or past applications under `documents/`; Claude reads and
  cross-references them.
- **Path B (single CV import):** share one CV/resume with `@file` or pasted
  text; Claude extracts it and asks follow-up questions.
- **Path C (interview mode):** answer structured questions section by
  section.

Files populated: `CLAUDE.md`, `01-candidate-profile.md`,
`02-behavioral-profile.md`, `04-job-evaluation.md`, `05-cv-templates.md`,
`07-interview-prep.md`, `cv/main_example.tex`, `search-queries.md`.

Re-run later to update just one section:

```
/setup --section skills
/setup --section experience
/setup --section search
```

## 4. Optional: salary benchmarking

If you have salary data (union data, salary survey, Glassdoor, personal
research):

- **Option A:** create `salary_data.json` manually in the repo root — see
  `tools/README_SALARY_TOOL.md` for the format.
- **Option B:** convert from Excel:
  ```bash
  pip install openpyxl
  python3 tools/convert_salary_excel.py path/to/salary-data.xlsx --source "My Salary Data 2025"
  ```

This creates `salary_data.json`, used by `/apply` for salary benchmarking.
If skipped, salary lookup is simply omitted.

## 5. Test the workflow

Inside Claude Code, with a job posting in hand:

```
/apply https://jobindex.dk/job/1234567
```

or paste the job description directly:

```
/apply [paste job posting text here]
```

Claude evaluates fit against the profile, asks whether to proceed, drafts a
tailored CV and cover letter, has a reviewer agent critique the drafts, then
revises and presents the final output.

## 6. Compile the generated documents

```bash
# Bash / zsh / Git Bash
cd cv && lualatex main_<company>_<role>.tex && cd ..
cd cover_letters && xelatex cover_<company>_<role>.tex && cd ..
```

```powershell
# PowerShell
Set-Location cv; lualatex main_<company>_<role>.tex; Set-Location ..
Set-Location cover_letters; xelatex cover_<company>_<role>.tex; Set-Location ..
```

To use a custom LaTeX template instead of the stock ones, run
`/add-template` inside Claude Code — it captures the template's compile
engine, fonts, style rules, and page limit, then wires it into `/apply`.

## 7. Make the repository private

**Important:** a fork of a public GitHub repo cannot be made private —
GitHub does not allow it. `/setup` writes personal data (CV details, career
profile) into tracked files, so pushing those commits to a public fork
publishes that data. Use a private repository instead, with the original
project added as `upstream` so updates can still be pulled in.

### Option A — fresh private repo (recommended)

1. Create a new empty **private** repository on GitHub (no README).
2. Check the current remote:
   ```bash
   git remote -v
   ```
3. If it points at the public fork/template, rename it to `upstream`:
   ```bash
   git remote rename origin upstream
   ```
4. Add the new private repo as `origin`:
   ```bash
   git remote add origin https://github.com/<your-username>/<your-private-repo>.git
   ```
5. Push:
   ```bash
   git push -u origin master   # or main
   ```

### Option B — a public fork already exists on GitHub

1. Create a new empty **private** repository on GitHub.
2. Add it as a second remote and push there instead of the fork:
   ```bash
   git remote add private https://github.com/<your-username>/<your-private-repo>.git
   git push -u private master
   ```
3. Push all future personalized commits to `private` only — never to the
   public fork.
4. Optional cleanup: once the private repo is confirmed working, delete the
   public fork to avoid accidentally pushing sensitive data to it later.

### Pulling upstream updates afterward

Prefer tagged releases over raw `master`:

```bash
git fetch upstream --tags
git merge v1.0.0    # substitute the latest release tag
```

Preview what changed before merging:

```bash
python3 tools/check_upstream_updates.py
```

Triage which upstream commits are worth reviewing:

```bash
python3 tools/upstream_triage.py --remote upstream
```

A merge conflict in a personalized file is expected, not a failure — it
means upstream changed methodology in a section you customized. Resolve by
keeping your data and adopting the methodology change around it.

Note: the genuinely sensitive files (application tracker, salary data,
`documents/`, application archives) are already gitignored and never enter
git, regardless of which remote is used.

## Troubleshooting

- **"salary_data.json not found"** — expected if salary benchmarking wasn't
  set up; `/apply` skips this step automatically.
- **Job search CLI tools not working** — confirm Bun is installed and
  `bun install` ran successfully in each CLI directory; the tools need
  network access.
- **LaTeX compilation errors** — CV uses `lualatex`, cover letter uses
  `xelatex`; confirm the TeX distribution includes the `moderncv` package.
- **Fonts not found in cover letter** — confirm
  `cover_letters/OpenFonts/fonts/` exists and contains the Lato and Raleway
  font files.
- **Stale `.claude/settings.local.json`** — shared permissions now live in
  `.claude/settings.json`. If a broader `settings.local.json` remains from
  an older clone, delete it:
  ```bash
  rm .claude/settings.local.json
  ```
