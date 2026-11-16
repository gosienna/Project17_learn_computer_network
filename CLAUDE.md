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

- `reference/Computer Networks (Andrew S. Tanenbaum.pdf` is the course textbook (about 31 MB).

## Working with the textbook PDF

- The Read tool needs `pdftoppm` from poppler to render PDF pages, and poppler isn't installed on this machine yet (`brew install poppler`).
- The PDF is large, so read it in small page ranges with the Read tool's `pages` parameter. Don't try to load the whole file.
