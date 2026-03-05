---
page_meta:
  title: Coding Standards
  labels:
    - engineering
    - standards
deploy_config:
  ci_banner: true
---

# Coding Standards

These standards apply to all production code at Acme Corp. They exist to reduce cognitive overhead when switching between services, not to enforce a particular aesthetic.

## Status

| Standard | Status |
| --- | --- |
| Python style guide (PEP 8 + Black) | ::Accepted::green:: |
| TypeScript strict mode | ::Accepted::green:: |
| Go formatting (gofmt) | ::Accepted::green:: |
| API versioning policy | ::In Progress::blue:: |
| Database migration policy | ::Review::yellow:: |

---

## Python

- Formatter: **Black** (line length 88). No exceptions.
- Linter: **Ruff** — configured in `pyproject.toml`, checks enforced in CI.
- Type hints: required on all public functions and class methods.
- Docstrings: Google style on public APIs; internal helpers don't require them.

```python
def calculate_discount(price: float, rate: float) -> float:
    """Apply a percentage discount to a price.

    Args:
        price: Original price in GBP.
        rate: Discount rate as a decimal (e.g. 0.15 for 15%).

    Returns:
        Discounted price, rounded to 2 decimal places.
    """
    return round(price * (1 - rate), 2)
```

## TypeScript

- **Strict mode** enabled in all `tsconfig.json` files.
- `any` is banned — use `unknown` and narrow with type guards.
- Prefer `interface` over `type` for object shapes; use `type` for unions and aliases.
- No `console.log` in committed code — use the structured logger from `@acme/logger`.

## Go

- `gofmt` and `golangci-lint` run in CI. PR will not merge if either fails.
- Error wrapping: use `fmt.Errorf("context: %w", err)` — never discard errors.
- Avoid `init()` functions unless absolutely necessary.

---

## Pull Request Guidelines

> [!info]
> PRs are the primary quality gate at Acme. These aren't bureaucracy — they're how we catch mistakes before they reach production.

### Size

Keep PRs small and focused. A PR that touches more than 400 lines of non-generated code needs a written justification in the description.

| Size | Lines changed | Expectation |
| --- | --- | --- |
| Small | < 100 | Merge same day |
| Medium | 100–400 | Reviewed within 1 business day |
| Large | 400+ | Requires justification; consider splitting |

### Reviews

- At least **1 approval** required before merging.
- The author is responsible for resolving all comments before merging — don't merge with unresolved threads.
- Reviewers should respond within **1 business day**.

### Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(orders): add bulk cancellation endpoint
fix(auth): handle expired token on refresh
chore(deps): bump requests to 2.32.0
```

---

## CI Requirements

All of the following must pass before a PR can merge:

:::expand CI checks detail
- **Lint**: Ruff (Python), ESLint (TypeScript), golangci-lint (Go)
- **Tests**: Unit tests with ≥ 80% coverage on changed files
- **Type check**: mypy (Python), tsc (TypeScript)
- **Security**: Snyk dependency scan — critical/high findings block merge
- **Docs**: ccfm plan check (informational, non-blocking for feature PRs)
:::

> [!warning]
> Do not use `--no-verify` to bypass pre-commit hooks. If a hook is blocking you, investigate the underlying issue rather than skipping the check.
