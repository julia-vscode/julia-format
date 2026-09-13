# julia-format

> [!WARNING]
> This action is under active development and its interface may change.

A GitHub Action that runs
[`juliaformat`](https://github.com/julia-vscode/FormatApp.jl) over a
Julia repository. In the default `check` mode it never modifies the
repository, fails when any file is not formatted, and prints the diff in the
job log; in `write` mode it reformats the files in place (useful together with
an auto-commit step).

The action installs Julia (via juliaup) itself, so it has no prerequisites
beyond a checkout. The exact versions of FormatApp and all of its
dependencies are pinned by the committed `Manifest.toml`, so every run uses
the same, known-good versions.

Formatting is configured with a `JuliaFormat.toml` file in the formatted
repository — see the
[FormatApp documentation](https://github.com/julia-vscode/FormatApp.jl#configuration).
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
| `path` | `.` | Paths to format, relative to the workspace; separate multiple paths with spaces. |
| `mode` | `check` | `check` fails when any file is not formatted and prints the diff; `list` fails likewise but only names the files; `write` reformats the files in place. |
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

## Caching

The action caches its own toolkit — FormatApp and the tree its `Manifest.toml`
pins — in a depot of its own under `RUNNER_TEMP`, keyed on the runner OS and
architecture, the Julia version and a hash of that manifest. Nothing
run-specific enters the key, so the entry is written once and then only read,
rather than re-saved on every run and duplicated for every pull request the way
a depot cached with `julia-actions/cache` is.

The toolkit is also precompiled against a portable CPU target. Julia's default,
`native`, compiles package images for whichever machine precompiled them while
recording only the literal string `native` in the cache path — so a depot moved
between two runners with different CPUs looks valid, is rejected on load, and
recompiles. GitHub's runner fleet is mixed enough for that to happen regularly;
see [julia-actions/cache#114](https://github.com/julia-actions/cache/issues/114).
A platform whose Julia does not accept the target gets a warning and Julia's
default instead, keeping the behaviour it had before.

None of this needs configuration, and nothing in your workflow should point at
that depot.
