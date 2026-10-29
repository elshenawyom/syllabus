
# UgS Syllabus

The syllabus document for the Mathematical Undergraduate Seminar (UgS) at Constructor University. Each edition is self-contained: one `syllabus.tex` that renders through `ugssyllabus.sty`, which layers on top of `smjarticle.sty`, which pulls in the shared `smj/` style modules.

## Layout

```
syllabus/
├── main.tex          ← the edition's content (only file that changes per edition)
├── main.pdf          ← latest compilation of main.tex
├── ugssyllabus.sty       ← syllabus style (run-in sections, unnumbered ToC,
│                            \smjsyllabus metadata, smjsyllabusabstract wrapper)
├── smjarticle.sty        ← article foundation (title block, abstract, header)
├── smj/                  ← shared style modules (fonts, colors, layout, theorems, …)
├── assets/               ← images used in the document (exmp-*-toc.png)
├── copies/               ← final PDFs (version of record)
├── .gitignore
└── build artefacts       ← ignored
```

`syllabus.tex` is the only file that changes between editions. The `.sty` files and `smj/` are infrastructure.

## Building

```powershell
latexmk -pdf syllabus.tex
```

Two passes run automatically. No `.bib`, no biber, no `.bbl`.

When the PDF is final, move it into `copies/`:

```powershell
Move-Item syllabus.pdf copies/syllabus.pdf
```

`copies/syllabus.pdf` is the version of record for the current edition.

## The interface

Two commands, one abstract environment, run-in sections. Do not invent new ones.

| Command / environment                                      | Effect                                                                                |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `\smjsyllabus{title=…, author=…, date=…}`             | Sets metadata and renders the title page. Must be called inside `\begin{document}`. |
| `\begin{smjsyllabusabstract}…\end{smjsyllabusabstract}` | Styled abstract block + `\tableofcontents` + `\newpage` on `\end`.              |
| `\section{Structure.}`                                   | Run-in, unnumbered. The leading word usually ends with a period.                      |
| `\subsection{1. Pick a topic.}`                          | Run-in, unnumbered. Numbers are written into the title text, not the counter.         |

Everything else is standard LaTeX. `\begin{example}…\end{example}` is a definition-style theorem environment. `\href{…}{…}` for external links. `\footnote{…}` for asides. `\label{…}` / `\ref{…}` for cross-references.

## Writing an edition

Edit `syllabus.tex`. Do not edit `ugssyllabus.sty`, `smjarticle.sty`, or anything in `smj/` unless you are fixing a bug in the shared style.

The document structure:

```latex
\documentclass[12pt]{article}
\usepackage{ugssyllabus}
\begin{document}
\pagenumbering{gobble}
\smjsyllabus{
  title  = {The \UgS Syllabus},
  author = {Omar Elshinawy},
  date   = {Spring 2025},
}
\begin{smjsyllabusabstract}
…
\end{smjsyllabusabstract}
\pagenumbering{arabic}

\section{Structure.} …
\subsection{1. Pick a topic.} …
\subsection{2. Prepare an article.} …
\subsection{3. Present a talk.} …

\section{Deliverables.} …
\subsection{1. Abstract,} \label{deliverable-1} …
\subsection{2. Report Draft,} \label{deliverable-2} …

\section{Closing Words.} …
\end{document}
```

The `\pagenumbering{gobble}` / `\pagenumbering{arabic}` pair suppresses page numbers on the title page and front matter, then restarts at 1 for the body.

## Editions and branches

One git branch per edition. See *Branch strategy* below.

## Reference material

`assets/` contains images used in the document. Add new assets there and reference them with a relative path: `\includegraphics{assets/my-figure.png}`.

## Ignoring build artefacts

`.gitignore` covers everything `latexmk` produces:

```
*.fdb_latexmk
*.fls
*.log
*.out
*.toc
*.aux
*.synctex.gz
*.bbl
*.blg
```

`syllabus.pdf` is not committed. The version of record lives in `copies/` under a stable name.

## Branch strategy

One branch per edition.

```
main          ← repo metadata (this README, LICENSE, CI, tooling)
spring2025    ← edition branch (live)
fall2025      ← edition branch
spring2026    ← edition branch
```

`main` holds only metadata. Each edition lives on its own branch and is self-contained — its own `syllabus.tex`, `ugssyllabus.sty`, `smjarticle.sty`, `smj/`, `assets/`, `copies/`.

### Setup (one-time)

```powershell
# main: minimal
git checkout --orphan main
git rm -rf .
# write README.md, LICENSE, .gitignore
git add . && git commit -m "Initialize repository"

# spring2025: the edition
git checkout --orphan spring2025
git checkout main -- .gitignore
# add syllabus.tex, ugssyllabus.sty, smjarticle.sty, smj/, assets/, copies/
git add . && git commit -m "Spring 2025 edition"

git push -u origin main spring2025
```

### Contributor workflow

1. Fork the repo.
2. `git checkout spring2025`.
3. Edit `syllabus.tex` (or, if fixing a style bug, `ugssyllabus.sty` and note it in the PR).
4. `latexmk -pdf syllabus.tex` — verify it builds.
5. Update `copies/syllabus.pdf` with the rebuilt version.
6. Open PR against `spring2025`.

Reviewer approves, merges, tags:

```powershell
git checkout spring2025
git pull
git tag -a spring2025-v1.1 -m "Spring 2025, revision 1.1"
git push --tags
```

### Propagating shared style fixes

`smjarticle.sty` and `smj/` are copied into every edition branch. A fix on one edition does not automatically reach the others. Two options:

- **Manual cherry-pick.** Fix once, `git checkout fall2025 && git cherry-pick <sha>` on each branch. Tedious but explicit.
- **Migrate to folders.** Move shared files to `main/common/` and each `syllabus.tex` reads from `../common/`. Diffs then propagate through `main`.

Cherry-pick is fine for a syllabus that's written once and touched a few times. Migrate to folders only if the shared style starts changing monthly.

### Edition lifecycle

| State              | Access                               | Trigger                          |
| ------------------ | ------------------------------------ | -------------------------------- |
| **Live**     | PRs welcome                          | Edition is current or upcoming   |
| **Frozen**   | No PRs, tag `<branch>-final`       | Semester ended, no further edits |
| **Archived** | Read-only, tag `<branch>-archived` | Kept for reference               |

Freeze at semester end so the record is stable.

### Naming

- Branches: `spring2025`, `fall2025`, `spring2026`. Lowercase, season + year.
- Tags: `<branch>-v<major>.<minor>` for release points, `<branch>-final` at freeze, `<branch>-archived` at archive.
- Commits on edition branches: prefix with scope when useful (`syllabus:`, `style:`, `assets:`). Not mandatory.
