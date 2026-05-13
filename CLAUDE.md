# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static HTML documentation project containing Chinese translations of Claude Code guides. The project is designed to work **offline** with local image assets.

## Files

- `the-shortform-guide-zh.html` - Quick reference guide (速查指南)
- `the-longform-guide-zh.html` - Comprehensive tutorial (详细指南)
- `the-security-guide-zh.html` - Security guide (安全指南)
- `images/` - Local image assets organized by guide:
  - `shortform/` - Quick reference images
  - `longform/` - Tutorial images
  - `security/` - Security guide images

## Architecture

Each HTML file is self-contained with embedded CSS styles. All images are referenced via relative paths to the local `images/` directory, enabling offline viewing without external dependencies.

## Git Branches

- `main` - Primary branch
- `master-offline` - Current working branch with offline image support
