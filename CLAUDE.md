# Development Guidelines

This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Core Deployment Rules

1. K8S Manifest File Organization
   - Keep related resources in a single file separated by `---` delimiters (e.g., Deployment + Service + Ingress together)
   - Organize manifests under `templates/` in per-student directories (e.g., `templates/nem2p/`)
   - Use subdirectories for specialized deployments (e.g., `hpa-testing/`)

2. K8S Resource Naming Conventions
   - Deployments: `{app-name}-deployment`
   - Services: `{app-name}-service`
   - Ingresses: `{app-name}-ingress`
   - HPAs: `{app-name}-hpa`
   - Jobs: `papermill-job-{student-id}`
   - File names should reflect the primary app (e.g., `fastapi.yaml`, `accord.yaml`)

3. K8S Labels and Selectors
   - All workloads must have an `app: {name}` label
   - Service selectors must match deployment `matchLabels`
   - Keep label usage consistent and minimal

4. K8S Resource Requests and Limits
   - All Deployments and Jobs MUST define both `requests` and `limits`
   - Deployment defaults: `cpu: 100m/250m`, `memory: 128Mi/256Mi` (requests/limits)
   - Job defaults: `cpu: 1/1`, `memory: 1024Mi/1024Mi` (requests/limits)
   - Jobs use a 1:1 request-to-limit ratio; Deployments use ~1:2 ratio

5. K8S Networking and Ingress
   - Use `ingressClassName: nginx` for all Ingress resources
   - Use `Prefix` path type for all ingress rules
   - Domain pattern: `{app-name}.uvasds.sh`
   - Internal services use `ClusterIP` type
   - Only the ingress controller uses `LoadBalancer` type
   - AWS LB annotations go on the ingress controller Service, not on individual Ingresses

6. K8S Job Configuration
   - Jobs must set `restartPolicy: Never`
   - Jobs must set `ttlSecondsAfterFinished: 86400` (24-hour auto-cleanup)
   - Parameterize jobs via `env` variables, not command-line args

7. K8S Namespaces
   - Application workloads go in `default` namespace
   - ArgoCD resources go in `argocd` namespace
   - Ingress controller resources go in `ingress-nginx` namespace

8. K8S Image References
   - Prefer `ghcr.io/uvasds-systems/` as the primary container registry
   - Use specific image tags (e.g., `:1.3`, `:1.117`) for production workloads
   - Avoid `:latest` tags except for development/testing
   - Be sure amd64 architecture is supported for images.

9. ArgoCD and GitOps
   - This repo is a Helm chart deployed via ArgoCD with automated sync
   - ArgoCD is configured with `prune: true` and `selfHeal: true`
   - Changes merged to `main` are automatically deployed
   - The Helm chart uses raw manifests in `templates/` (no `.Values` templating currently)

## Core Development Rules

1. Package Management
   - ONLY use pipenv for creating environment and package installation, NEVER pip or uv
   - Installation: `pipenv install`
   - Activation: `pipenv shell`
   - FORBIDDEN: `uv pip install`, `@latest` syntax

2. Code Quality
   - Type hints required for all code
   - Public APIs must have docstrings
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 88 chars maximum

3. Testing Requirements
   - Framework: `pytest`
   - Async testing: use anyio, not asyncio
   - Coverage: test edge cases and errors
   - New features require tests
   - Bug fixes require regression tests

4. Code Style
    - PEP 8 naming (snake_case for functions/variables)
    - Class names in PascalCase
    - Constants in UPPER_SNAKE_CASE
    - Document with docstrings
    - Use f-strings for formatting

- For commits fixing bugs or adding features based on user reports add:
  ```bash
  git commit --trailer "Reported-by:<name>"
  ```
  Where `<name>` is the name of the user.

- For commits related to a Github issue, add
  ```bash
  git commit --trailer "Github-Issue:#<number>"
  ```
- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions
- **Less Code = Less Debt**: Minimize code footprint

## Coding Best Practices

- **Early Returns**: Use to avoid nested conditions
- **Descriptive Names**: Use clear variable/function names (prefix handlers with "handle")
- **Constants Over Functions**: Use constants where possible
- **DRY Code**: Don't repeat yourself
- **Functional Style**: Prefer functional, immutable approaches when not verbose
- **Minimal Changes**: Only modify code related to the task at hand
- **Function Ordering**: Define composing functions before their components
- **TODO Comments**: Mark issues in existing code with "TODO:" prefix
- **Simplicity**: Prioritize simplicity and readability over clever solutions
- **Build Iteratively** Start with minimal functionality and verify it works before adding complexity
- **Run Tests**: Test your code frequently with realistic inputs and validate outputs
- **Build Test Environments**: Create testing environments for components that are difficult to validate directly
- **Functional Code**: Use functional and stateless approaches where they improve clarity
- **Clean logic**: Keep core logic clean and push implementation details to the edges
- **File Organsiation**: Balance file organization with simplicity - use an appropriate number of files for the project scale

## System Architecture

This repository is a Helm chart (`k8s-state/`) that defines the desired state of an EKS cluster. ArgoCD watches the `main` branch and automatically syncs changes to the cluster. Student workloads are organized in per-student subdirectories under `templates/`.

## Core Components

- `Chart.yaml`: Helm chart metadata (type: application, version: 0.1.0)
- `values.yaml`: Helm values (placeholder, not currently used)
- `templates/`: All K8S manifest files organized by student directory
- `templates/nem2p/argo-app.yaml`: ArgoCD Application definition
- `templates/nem2p/fastapi.yaml`: FastAPI deployment + service + ingress
- `templates/nem2p/nginx-ingress-service.yaml`: Nginx ingress controller LoadBalancer
- `templates/nem2p/hpa-testing/`: HPA experimentation manifests
- `templates/*/job.yaml`: Per-student Papermill job definitions

## Pull Requests

- Create a detailed message of what changed. Focus on the high level description of
  the problem it tries to solve, and how it is solved. Don't go into the specifics of the
  code unless it adds clarity.

- Always add `ArthurClune` as reviewer.

- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Python Tools

## Code Formatting

1. Ruff
   - Format: `uv run ruff format .`
   - Check: `uv run ruff check .`
   - Fix: `uv run ruff check . --fix`
   - Critical issues:
     - Line length (88 chars)
     - Import sorting (I001)
     - Unused imports
   - Line wrapping:
     - Strings: use parentheses
     - Function calls: multi-line with proper indent
     - Imports: split into multiple lines

2. Type Checking
   - Tool: `uv run pyright`
   - Requirements:
     - Explicit None checks for Optional
     - Type narrowing for strings
     - Version warnings can be ignored if checks pass

3. Pre-commit
   - Config: `.pre-commit-config.yaml`
   - Runs: on git commit
   - Tools: Prettier (YAML/JSON), Ruff (Python)
   - Ruff updates:
     - Check PyPI versions
     - Update config rev
     - Commit config first

## Error Resolution

1. CI Failures
   - Fix order:
     1. Formatting
     2. Type errors
     3. Linting
   - Type errors:
     - Get full line context
     - Check Optional types
     - Add type narrowing
     - Verify function signatures

2. Common Issues
   - Line length:
     - Break strings with parentheses
     - Multi-line function calls
     - Split imports
   - Types:
     - Add None checks
     - Narrow string types
     - Match existing patterns

3. Best Practices
   - Check git status before commits
   - Run formatters before type checks
   - Keep changes minimal
   - Follow existing patterns
   - Document public APIs
   - Test thoroughly

## Rules about Claude Code Behavior

- Do not pander or flatter me.
- Do not cite references that don't exist. SHow me citations.