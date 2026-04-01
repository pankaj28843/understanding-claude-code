# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A research and documentation repository analyzing Claude Code's architecture as revealed by the March 31, 2026 npm source map leak. There is no application code, build system, or tests — only Markdown files.

## Structure

- `README.md` — The main document. A 1,300+ line comprehensive analysis covering architecture, security, memory, tools, unreleased features, community reactions, and legal implications. This is the primary deliverable.
- `ARCHITECTURE.md` — Condensed technical reference of Claude Code's subsystems, design principles, and key metrics.
- `SOURCES.md` — Index of all research sources: articles, HN threads, official docs, AutoDream analyses, media coverage.

## Working on This Repo

- All content is Markdown. Edits are prose/research, not code.
- The README is the canonical document. ARCHITECTURE.md and SOURCES.md are supplementary.
- When enriching content, check the podcast research at `~/Personal/Code/rss-audio-forge/tmp/podcast-research/understanding-claude-code/` (26 articles, 7 HN threads, 10 TechDocs) and podcast scripts at `~/Personal/Code/rss-audio-forge/tmp/podcast-scripts/understanding-claude-code/` (12 episodes with deep technical detail).
- Maintain the existing section structure and table of contents in README.md — update the TOC when adding/removing sections.
- Use tables, code blocks, and ASCII diagrams for technical content (matching existing style).
- Always include source links for claims. Cross-reference with SOURCES.md.
- The repo is public at github.com/pankaj28843/understanding-claude-code.
