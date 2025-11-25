# AI Code Review

Reusable GitHub Actions workflow for AI-powered code review with auto-merge support.

## Features

- Multi-AI provider support (Gemini, OpenAI, Claude)
- Multi-language project support (any codebase)
- Security vulnerability scanning with Trivy
- Configurable approval threshold
- Auto-merge for approved PRs
- Smart diff handling (auto-approve docs-only changes)
- Multi-language review output (Chinese/English)
- Detailed review comments on PR

## Quick Start

### 1. Add Secret

In your repository, go to **Settings > Secrets and variables > Actions** and add:

- `GEMINI_API_KEY`: Your Google Gemini API key (or `OPENAI_API_KEY` / `ANTHROPIC_API_KEY`)

### 2. Create Workflow

Create `.github/workflows/code-review.yml` in your repository:

```yaml
name: Code Review

on:
  pull_request:
    branches: [main]

jobs:
  review:
    uses: your-org/CodeReview/.github/workflows/ai-code-review.yml@main
    secrets:
      AI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

That's it! PRs to main will now be automatically reviewed.

## Configuration

All inputs are optional:

| Input | Description | Default |
|-------|-------------|---------|
| `ai-provider` | AI provider (gemini, openai, claude) | `gemini` |
| `ai-model` | AI model name (empty for default) | `` |
| `min-score` | Minimum score for approval (0-10) | `7` |
| `max-files` | Max files for auto-merge | `20` |
| `max-lines` | Max lines for auto-merge | `500` |
| `auto-merge` | Enable auto-merge | `true` |
| `merge-method` | Merge method (merge/squash/rebase) | `squash` |
| `language` | Review language (zh/en) | `zh` |
| `run-security-scan` | Run Trivy security scan | `true` |
| `exclude-patterns` | Files to exclude from review | `*.md,*.txt,*.json,*.lock,*.yaml,*.yml` |
| `custom-prompt` | Additional review instructions | `` |

### Default Models

| Provider | Default Model |
|----------|---------------|
| gemini | gemini-2.0-flash |
| openai | gpt-4o |
| claude | claude-sonnet-4-20250514 |

### Full Example

```yaml
name: Code Review

on:
  pull_request:
    branches: [main, develop]

jobs:
  review:
    uses: your-org/CodeReview/.github/workflows/ai-code-review.yml@main
    secrets:
      AI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    with:
      ai-provider: 'openai'
      ai-model: 'gpt-4o'
      min-score: 8              # Stricter approval
      max-files: 10             # Smaller PRs only
      max-lines: 300
      auto-merge: false         # Review only, no auto-merge
      language: 'en'            # English comments
      run-security-scan: true
      exclude-patterns: '*.md,*.txt,docs/*'
      custom-prompt: 'Focus on error handling and edge cases'
```

## Outputs

The workflow provides these outputs for downstream jobs:

| Output | Description |
|--------|-------------|
| `score` | AI review score (0-10) |
| `approved` | Whether PR passed review |
| `merged` | Whether PR was auto-merged |

### Using Outputs

```yaml
jobs:
  review:
    uses: your-org/CodeReview/.github/workflows/ai-code-review.yml@main
    secrets:
      AI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

  notify:
    needs: review
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Score: ${{ needs.review.outputs.score }}"
          echo "Approved: ${{ needs.review.outputs.approved }}"
```

## Review Criteria

The AI evaluates PRs on 5 dimensions (0-10 each):

1. **Code Quality** - Readability, complexity, naming
2. **Security** - Vulnerabilities, input validation
3. **Performance** - Algorithm efficiency, resource usage
4. **Testing** - Test coverage
5. **Best Practices** - Following conventions

## Special Behaviors

### Empty Diff Handling
PRs with only documentation/config changes (matching `exclude-patterns`) are auto-approved with score 10.

### Sensitive File Filtering
Files matching these patterns are automatically excluded from AI review:
- `.env*`
- `*secret*`
- `*.pem`, `*.key`

### Comment Updates
When a PR is updated, the existing AI review comment is updated instead of creating a new one.

## Requirements

- GitHub Actions enabled
- AI API key (Gemini/OpenAI/Anthropic)
- Repository permissions: `contents: write`, `pull-requests: write`

## License

MIT
