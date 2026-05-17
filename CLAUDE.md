# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static HTML documentation project containing Chinese translations of Claude Code guides. The project is designed to work **offline** with local image assets.

## HTML Guide Files

| File | Description | Source |
|------|-------------|--------|
| `the-shortform-guide-zh.html` | Quick reference guide (速查指南) | Original ECC project |
| `the-longform-guide-zh.html` | Comprehensive tutorial (详细指南) | Original ECC project |
| `the-security-guide-zh.html` | Security guide (安全指南) | Original ECC project |

## ECC Phase Guides (Markdown)

| File | Phase |
|------|-------|
| `ECC-GUIDE.md` | Main overview with all 10 phases |
| `ECC-GUIDE-requirements-exploration.md` | Phase 1: Requirements understanding |
| `ECC-GUIDE-architecture-design.md` | Phase 2: Architecture design |
| `ECC-GUIDE-development.md` | Phase 3: Development |
| `ECC-GUIDE-code-review.md` | Phase 4: Code review |

## Image Assets

All images are in `images/` directory, organized by guide:
- `images/shortform/` - Quick reference images (12 files)
- `images/longform/` - Tutorial images (9 files)
- `images/security/` - Security guide images (7 files)

## Architecture

Each HTML file is self-contained with embedded CSS styles. All images use relative paths to local `images/` directory for offline viewing.

## Git Branches

- `main` - Primary branch
- `master-offline` - Current working branch with offline image support
