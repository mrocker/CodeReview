# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **reusable GitHub Actions workflow** for AI-powered code review. It's designed to be called from other repositories via `workflow_call`, not run directly.

## Architecture

The workflow (`ai-code-review.yml`) consists of 3 jobs:

1. **security-scan** - Uses Trivy for multi-language vulnerability scanning (dependencies, code, IaC)
2. **ai-code-review** - Uses AI (Gemini/OpenAI/Claude) to analyze PR diffs and score changes on 5 dimensions
3. **auto-approve-and-merge** - Conditionally approves and merges PRs that pass AI review

Key design decisions:
- Multi-AI provider support via `ai-provider` input (gemini, openai, claude)
- Language-agnostic: no language-specific linting, Trivy handles all languages
- AI review uses inline Python embedded in the workflow YAML
- Supports Chinese (zh) and English (en) review output
- Smart diff handling: auto-approves docs-only PRs, excludes sensitive files
- Updates existing PR comments instead of creating duplicates
- API calls include retry logic with exponential backoff

## Input/Output Reference

**Key Inputs:**
- `ai-provider`: gemini | openai | claude
- `ai-model`: Custom model name (empty = provider default)
- `exclude-patterns`: Comma-separated glob patterns to skip
- `custom-prompt`: Additional review instructions

**Secrets:**
- `AI_API_KEY`: API key for the selected provider

## Testing Changes

This repository has no automated tests. To test workflow changes:

1. Create a test repository that calls this workflow
2. Open a PR in the test repository to trigger the workflow
3. Check the workflow run logs in GitHub Actions
