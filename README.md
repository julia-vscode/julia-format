# julia-format

> [!WARNING]
> This action is under active development and its interface may change.

A GitHub Action that runs a check-only formatting pass over a Julia repository
with [`juliaformat`](https://github.com/julia-vscode/JuliaFormatApp.jl)
(`juliaformat --check --diff .`): it never modifies the repository, fails when
any file is not formatted, and prints the diff in the job log.

The action installs Julia (via juliaup) itself, so it has no prerequisites
beyond a checkout. The exact versions of JuliaFormatApp and all of its
dependencies are pinned by the committed `Manifest.toml`, so every run uses
the same, known-good versions.

Formatting is configured with a `JuliaFormat.toml` file in the formatted
repository — see the
[JuliaFormatApp documentation](https://github.com/julia-vscode/JuliaFormatApp.jl#configuration).

## Usage

```yaml
jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: julia-vscode/julia-format@v1
        # Opt-in: only run when the repository has a JuliaFormat.toml
        if: ${{ hashFiles('**/JuliaFormat.toml', '**/juliaformat.toml') != '' }}
```

## Updating pinned dependencies

```
julia --project=. -e 'using Pkg; Pkg.update()'
```

and commit the changed `Manifest.toml`.
