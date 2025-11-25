# AI Code Review

Reusable GitHub Actions workflow for AI-powered code review with auto-merge support.

## Features

- AI code review using Google Gemini
- Configurable approval threshold
- Auto-merge for approved PRs
- Security vulnerability scanning (Bandit, Safety)
- Code quality checks (Black, Ruff)
- Multi-language support (Chinese/English)
- Detailed review comments on PR

## Quick Start

### 1. Add Secret

In your repository, go to **Settings > Secrets and variables > Actions** and add:

- `GEMINI_API_KEY`: Your Google Gemini API key

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
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

That's it! PRs to main will now be automatically reviewed.

## Configuration

All inputs are optional:

| Input | Description | Default |
|-------|-------------|---------|
| `python-version` | Python version for checks | `3.12` |
| `min-score` | Minimum score for approval (0-10) | `7` |
| `max-files` | Max files for auto-merge | `20` |
| `max-lines` | Max lines for auto-merge | `500` |
| `auto-merge` | Enable auto-merge | `true` |
| `merge-method` | Merge method (merge/squash/rebase) | `squash` |
| `language` | Review language (zh/en) | `zh` |
| `gemini-model` | Gemini model to use | `gemini-2.0-flash` |
| `run-security-scan` | Run security checks | `true` |
| `run-code-quality` | Run code quality checks | `true` |

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
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
    with:
      python-version: '3.11'
      min-score: 8              # Stricter approval
      max-files: 10             # Smaller PRs only
      max-lines: 300
      auto-merge: false         # Review only, no auto-merge
      language: 'en'            # English comments
      gemini-model: 'gemini-2.0-flash'
      run-security-scan: true
      run-code-quality: true
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
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

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

## Requirements

- GitHub Actions enabled
- Google Gemini API key
- Repository permissions: `contents: write`, `pull-requests: write`

## License

MIT
