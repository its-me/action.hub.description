# Docker Hub repository description

A GitHub Action that updates a Docker Hub repository's short and full
description from a markdown file.

## Why this exists

- It's a plain composite action — a handful of `curl`/`jq` calls —
  with no Docker daemon required to start it, so it runs on minimal
  runners, including this account's `ubuntu-slim` runner.
- The entire implementation is the ~30 lines visible directly in
  `action.yaml`. Nothing to build, nothing bundled to trust blindly.
- The short description comes from the first line of
  `description-file`, so there's one file to maintain instead of a
  file plus a duplicated string in the workflow YAML.

## Usage

```yaml
name: Update descriptions

on:
  push:
    branches: [main]
    paths:
      - HUB.md
  workflow_dispatch:

jobs:
  update-hub:
    runs-on: ubuntu-slim
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v7
      - uses: its-me/action.hub.description@v1
        with:
          username: ${{ secrets.HUB_USERNAME }}
          password: ${{ secrets.HUB_PASSWORD }}
          repository: workflow
```

## Inputs

| Name               | Description                                                                                                                                          | Default   |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| `username`         | Docker Hub username.                                                                                                                                     | _(none)_  |
| `password`         | Docker Hub password. Used to obtain a login token — a personal access token does not work against this endpoint.                                        | _(none)_  |
| `repository`       | Docker Hub repository name, without the username (e.g. `workflow`).                                                                                      | _(none)_  |
| `description-file` | Path to a markdown file. Its first line must be an HTML comment holding the short description (e.g. `<!-- short description -->`); the rest of the file becomes the full description. | `HUB.md`  |
| `url-completion`   | Rewrite relative markdown links/images in the full description to absolute GitHub URLs, since Docker Hub renders the description outside the repository's context. Covers `![alt](path)` and `[label](path)` references (with or without a title), `[![alt](img)](path)` badge-links, and `[ref]: path` link reference definitions. In-page anchors (`#section`) are deliberately left as-is. | `false`   |
| `image-extensions` | Comma-separated file extensions treated as images when `url-completion` is enabled.                                                                     | `bmp,gif,jpg,jpeg,png,svg,webp` |

## Description file format

`description-file` (default `HUB.md`) is a markdown file structured as:

```
<!-- short description -->
Full description in markdown, starting from the second line
and continuing for the rest of the file.
```

The first line must be an HTML comment holding the short description;
everything from the second line onward becomes the full description.
Leading/trailing whitespace around `<!--`/`-->` and inside the comment
is trimmed, and both LF and CRLF line endings are accepted. If the
first line doesn't match this format at all, a `::warning::`
annotation is emitted and the short description is sent empty rather
than failing silently.

## Length limit

The full description is truncated to Docker Hub's 25,000-byte limit in
a Unicode-safe way — truncation always lands on a whole character,
never mid-codepoint — and then backs off to the last complete line, so
a cut never lands mid-way through a markdown link or image reference
(relevant since `url-completion` can expand a short relative path into
a much longer absolute URL). If the cut leaves an unclosed ` ``` ` code
fence open, a closing fence is appended so the rest of the page doesn't
render as code. When truncation happens, a `::warning::` annotation is
emitted in the job log.

## Retries

Both the login request and the description update request retry up to
3 times (5-second delay) on transient failures — timeouts, connection
refused, and HTTP 408/429/5xx — via `curl --retry`. Permanent failures
(e.g. bad credentials) are not retried.

## License

This project is licensed under the MIT License.

See [LICENSE](LICENSE) for the full text.
