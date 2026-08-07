# julia-format

> [!WARNING]
> This action is under active development and its interface may change.

A GitHub Action that runs
[`juliaformat`](https://github.com/julia-vscode/JuliaFormatApp.jl) over a
Julia repository. In the default `check` mode it never modifies the
repository, fails when any file is not formatted, and prints the diff in the
job log; in `write` mode it reformats the files in place (useful together with
an auto-commit step).

The action installs Julia (via juliaup) itself, so it has no prerequisites
beyond a checkout. The exact versions of JuliaFormatApp and all of its
dependencies are pinned by the committed `Manifest.toml`, so every run uses
the same, known-good versions.

Formatting is configured with a `JuliaFormat.toml` file in the formatted
repository — see the
[JuliaFormatApp documentation](https://github.com/julia-vscode/JuliaFormatApp.jl#configuration).
Note that without a config file `juliaformat` still formats everything with
its defaults; set `require-config: true` to make the action a no-op unless
the repository has opted in with a config file.

## Usage

```yaml
jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: julia-vscode/julia-format@v1
        with:
          # Opt-in: only run when the repository has a JuliaFormat.toml
          require-config: true
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `path` | `.` | Path to format, relative to the workspace. |
| `mode` | `check` | `check` fails when any file is not formatted and prints the diff; `write` reformats the files in place. |
| `require-config` | `false` | When `true`, the action is a no-op unless the repository contains a `JuliaFormat.toml` (or `juliaformat.toml`) file. |

## Outputs

| Output | Description |
| --- | --- |
| `formatted` | `true` when all files were formatted (or the action was skipped because `require-config` found no config file), `false` otherwise. |

## Updating pinned dependencies

```
julia --project=. -e 'using Pkg; Pkg.update()'
```

and commit the changed `Manifest.toml`.
