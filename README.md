# homebrew-tools

 Homebrew tap for tools by [@desaianand1](https://github.com/desaianand1).

## Usage

     brew tap desaianand1/tools
     brew install <formula>

 `brew tap` clones this repo locally. When you run `brew install <formula>`, Homebrew
 looks in `Formula/` for a matching `.rb` file.

## Available formulae

 | Formula | Description |
 |---|---|
 | `shrink` | Local-first CLI to minimize, convert, and batch-process files |

## How formulae get here

 Formula files are **auto-generated and pushed by GoReleaser** during the release
 pipeline of each tool's private source repo. Manual edits to files in `Formula/` will be
 overwritten on the next release.

 To change a formula, edit the `brews` section in the source repo's `.goreleaser.yaml`
 and let the next release propagate the change.

### Release flow (using shrink as example)

 1. semantic-release creates a version tag in the private `shrink` repo.
 2. GoReleaser builds binaries and generates `Formula/shrink.rb`.
 3. GoReleaser pushes `Formula/shrink.rb` to this repo using the
    `HOMEBREW_TAP_GITHUB_TOKEN` secret (a fine-grained PAT scoped to this repo with
    `Contents: Read and write`).
 4. Pre-release tags (e.g. `v1.0.0-beta.1`) do NOT update the formula, so `brew upgrade`
    never pulls a beta by accident. This is controlled by `skip_upload: auto` in
    GoReleaser config.

 Binary download URLs in the formula point to
 [shrink-releases](https://github.com/desaianand1/shrink-releases) (a public mirror
 repo), not the private source repo.

## Authentication & tokens

 This repo is written to by GoReleaser using a fine-grained PAT stored as
 `HOMEBREW_TAP_GITHUB_TOKEN` in the private source repo's secrets. The token needs:

- **Repository access**: This repo (`homebrew-tools`) only
- **Permissions**: `Contents: Read and write`
- **Created at**: GitHub → Settings → Developer settings → Personal access tokens →
   Fine-grained tokens

 If the token expires, GoReleaser will build and release successfully but the formula push
 will fail. The GitHub Release will exist in `shrink-releases` but `brew upgrade` won't
 see the new version until the token is renewed and a re-release or manual formula push is
 done.

## Adding a new tool

 1. Add a `brews` section to the new tool's `.goreleaser.yaml` pointing to this repo.
 2. Ensure `HOMEBREW_TAP_GITHUB_TOKEN` (or a new token) has access to this repo.
 3. Set `directory: Formula` and configure install/test/caveats blocks.
 4. On the next release, GoReleaser will create `Formula/<tool>.rb` automatically.
 5. Users install with: `brew install desaianand1/tools/<tool>`.

## Naming convention

 The repo is named `homebrew-tools` (not `homebrew-shrink`) so it can serve as a shared
 tap for multiple tools. Homebrew requires tap repos to start with `homebrew-`. The tap
 name users type is the suffix: `desaianand1/tools`.
