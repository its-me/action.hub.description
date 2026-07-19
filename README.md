# Docker Hub repository description

A GitHub Action that updates a Docker Hub repository's short and full
description from a markdown file.

It replaces the `update-hub` job pattern duplicated across this account's
image repositories (e.g.
[its-me/image-workflow](https://github.com/its-me/image-workflow)'s
`update-descriptions.yaml`) with a single reusable step.

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
      - uses: its-me/action.hub.description@v0
        with:
          username: ${{ secrets.HUB_USERNAME }}
          password: ${{ secrets.HUB_PASSWORD }}
          repository: workflow
```

## Inputs

| Name         | Description                                                                                                                                          | Default   |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| `username`   | Docker Hub username.                                                                                                                                     | _(none)_  |
| `password`   | Docker Hub password. Used to obtain a login token — a personal access token does not work against this endpoint.                                        | _(none)_  |
| `repository` | Docker Hub repository name, without the username (e.g. `workflow`).                                                                                      | _(none)_  |
| `file`       | Path to a markdown file. Its first line must be an HTML comment holding the short description (e.g. `<!-- short description -->`); the rest of the file becomes the full description. | `HUB.md`  |

## License

This project is licensed under the MIT License.

See [LICENSE](LICENSE) for the full text.
