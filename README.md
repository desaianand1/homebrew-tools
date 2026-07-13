# homebrew-tools

Homebrew tap for tools by [@desaianand1](https://github.com/desaianand1).

## Usage

    brew tap desaianand1/tools
    brew install --cask shrink

Or install directly (auto-taps):

    brew install --cask desaianand1/tools/shrink

## Available casks

| Cask | Description |
|---|---|
| `shrink` | Local-first CLI to minimize, convert, and batch-process files |

## How casks get here

Cask files are **auto-generated and pushed by GoReleaser** during the release
pipeline of each tool's private source repo. Manual edits to files in `Casks/` will be
overwritten on the next release.

To change a cask, edit the `homebrew_casks` section in the source repo's
`.goreleaser.yaml` and let the next release propagate the change.

### Release flow (using shrink as example)

1. semantic-release creates a version tag in the private `shrink` repo.
2. GoReleaser builds binaries and **signs macOS binaries** with Apple Developer ID
   via [rcodesign](https://github.com/indygreg/apple-platform-rs) (cross-platform
   Apple code signing, runs on Linux CI).
3. GoReleaser archives the signed binaries and generates `Casks/shrink.rb`.
4. **macOS binaries are notarized** with Apple's notary service, so Gatekeeper
   trusts them on end-user machines.
5. GoReleaser pushes `Casks/shrink.rb` to this repo using the
   `HOMEBREW_TAP_GITHUB_TOKEN` secret (a fine-grained PAT scoped to this repo with
   `Contents: Read and write`).
6. Pre-release tags (e.g. `v1.0.0-beta.1`) do NOT update the cask, so `brew upgrade`
   never pulls a beta by accident. This is controlled by `skip_upload: auto` in
   GoReleaser config.

Binary download URLs in the cask point to
[shrink-releases](https://github.com/desaianand1/shrink-releases) (a public mirror
repo), not the private source repo.

## Code signing & notarization

All macOS binaries are signed with an Apple Developer ID Application certificate and
notarized with Apple's notary service. This means:

- No Gatekeeper warnings ("Apple could not verify...")
- No need to right-click > Open or run `xattr -d com.apple.quarantine`
- Works via `brew install`, `install.sh`, or direct download

Signing and notarization run on a Linux CI runner using
[rcodesign](https://github.com/indygreg/apple-platform-rs) — a cross-platform
reimplementation of Apple's code signing tools.

## Design decision: Why Cask, not Formula (2026-07-13)

This tap distributes CLI tools as **Homebrew Casks**, not Formulae. This was a
deliberate and researched decision, documented here for posterity.

### The short answer

Two independent reasons converge on Cask:

1. **Homebrew's own guidance** directs closed-source, binary-only software to Casks
2. **GoReleaser** (our release toolchain) removed Formula support in v2.16 — Cask
   is the only supported Homebrew integration

### Homebrew's guidance

> "If you want to add software that is either **closed source** or a GUI-only
> program, you will want to follow the guide for Casks."
> — [Adding Software to Homebrew](https://docs.brew.sh/Adding-Software-to-Homebrew)

> "**We don't like binary formulae**: Our policy is that formulae in the core tap
> (homebrew/core) must be open-source with a Debian Free Software Guidelines
> license and either built from source or producing cross-platform binaries
> [...] **Binary-only formulae should go in homebrew/cask.**"
> — [Acceptable Formulae](https://docs.brew.sh/Acceptable-Formulae)

shrink is proprietary and distributes pre-built binaries — both criteria point to
Cask.

**Note:** These are editorial policies for `homebrew/core` and `homebrew/cask`
(the official repos), not hard technical requirements. Custom taps *can* use
Formulae for binary distribution. However, since GoReleaser no longer supports
Formula generation, and signing+notarization eliminates the quarantine concern
that was the primary advantage of Formulae, Cask is the pragmatic choice.

### GoReleaser's architecture

> "Historically, GoReleaser would generate *hackyish* formulas that would install
> the pre-compiled binaries. [...] Homebrew Casks should be used instead."
> — [GoReleaser Deprecation Notices](https://goreleaser.com/deprecations/)

The `brews` config was soft-deprecated in v2.10 and **removed in v2.16**.
`homebrew_casks` is now the only supported Homebrew integration. Since we use
GoReleaser as our release toolchain, this is a hard constraint.

### Why Cask without signing was a problem

Homebrew Casks preserve the `com.apple.quarantine` extended attribute on
downloaded binaries. Without an Apple Developer ID signature and notarization
ticket, macOS Gatekeeper blocks execution with "Apple could not verify..."

The Acceptable Casks doc confirms this is a rejection criterion for the official
`homebrew/cask` repo:

> "**App fails with GateKeeper enabled** on Homebrew supported macOS versions and
> platforms (e.g. unsigned apps will not launch on Apple Silicon Macs)."
> — [Acceptable Casks: Rejected Casks](https://docs.brew.sh/Acceptable-Casks#rejected-casks)

**Our solution:** All macOS binaries are now signed with a Developer ID
Application certificate and notarized with Apple's notary service, making the
Cask approach fully correct.

### Why not maintain a Formula manually?

Formulae strip `com.apple.quarantine` on install, which would side-step the
Gatekeeper issue without signing. We considered this but rejected it because:

1. GoReleaser can't generate Formulae — we'd need a custom CI script to maintain
   the Formula file, duplicating what GoReleaser already handles for Casks
2. Signing + notarization is needed anyway for `install.sh` (curl pipe) users
3. With signing in place, the quarantine-stripping benefit of Formula is redundant
4. Cask aligns with Homebrew's guidance for our distribution model

## Authentication & tokens

This repo is written to by GoReleaser using a fine-grained PAT stored as
`HOMEBREW_TAP_GITHUB_TOKEN` in the private source repo's secrets. The token needs:

- **Repository access**: This repo (`homebrew-tools`) only
- **Permissions**: `Contents: Read and write`
- **Created at**: GitHub > Settings > Developer settings > Personal access tokens >
  Fine-grained tokens

If the token expires, GoReleaser will build and release successfully but the cask push
will fail. The GitHub Release will exist in `shrink-releases` but `brew upgrade` won't
see the new version until the token is renewed and a re-release or manual cask push is
done.

## Adding a new tool

1. Add a `homebrew_casks` section to the new tool's `.goreleaser.yaml` pointing to
   this repo.
2. Ensure `HOMEBREW_TAP_GITHUB_TOKEN` (or a new token) has access to this repo.
3. Set `directory: Casks` and configure `binaries`/`dependencies`/`caveats` blocks.
4. On the next release, GoReleaser will create `Casks/<tool>.rb` automatically.
5. Users install with: `brew install --cask desaianand1/tools/<tool>`.

## Naming convention

The repo is named `homebrew-tools` (not `homebrew-shrink`) so it can serve as a shared
tap for multiple tools. Homebrew requires tap repos to start with `homebrew-`. The tap
name users type is the suffix: `desaianand1/tools`.
