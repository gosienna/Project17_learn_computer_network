# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This is a study project for learning computer networking by building things. So far there is no source code, build system, tests, or git repository. Update this file once code and tooling exist.

## Learning workflow

Teaching here follows the project's `instructor` skill (`.claude/skills/instructor/SKILL.md`). Read it before teaching. The main rules:

- The learner writes all application code. Claude gives one small, observable step at a time (one file, under 10 lines), as pseudocode only, and doesn't write runnable code.
- `framework/` holds the plan and the step checklist (`framework/level-1-bites.md` and later level files). The checkboxes there are the progress record.
- `reference/` holds learning material.
- Each finished step is committed on the learner's `learn/<slug>` branch.

## Contents

- `reference/Computer Networks (Andrew S. Tanenbaum.pdf` is the course textbook: 4th edition, 674 PDF pages, about 31 MB.
- `reference/README.md` maps each framework level to book sections by PDF page number.

## Working with the textbook PDF

There's no Homebrew on this machine. The user wants tools isolated, so don't install anything system-wide.

- **Page images, including figures:** poppler is installed locally in this folder's pixi environment (`pixi.toml`, env in `.pixi/`, which git ignores). `pdftoppm` isn't on PATH, so the Read tool can't open the PDF directly. Render pages to PNG in the scratchpad, then Read the PNG:
  `PIXI_NO_TELEMETRY=1 ~/.pixi/bin/pixi run pdftoppm -f <page> -l <page> -r 80 -png "reference/Computer Networks (Andrew S. Tanenbaum.pdf" <scratchpad>/p`
- **Text, for searching:** use PyMuPDF through uv in a throwaway environment: `~/.local/bin/uv run --no-project --with pymupdf python -I <script> ...`, with the script kept in the scratchpad. `fitz.open(path)[n-1].get_text()` gives PDF page `n`, and `doc.get_toc()` gives the outline. Figures come out blank in text mode.
- The 4th edition's Manchester convention (1 = high→low) is the reverse of IEEE 802.3. The framework follows the book.
