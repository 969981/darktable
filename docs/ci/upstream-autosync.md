# darktable upstream sync and automatic build releases

## Sources and branches
- Official upstream: `darktable-org/darktable`, branch `master`.
- Fork: `969981/darktable`. Automation is installed on the fork's default `darktable-1.1.x` branch to make scheduled GitHub Actions and workflow dispatch available.
- The fork's `master` branch is safe-fast-forwarded to official `master` only. The historical default branch and other custom branches are left untouched. Diverged `master` causes a deliberate failure; no force pushes.
- `darktable-autosync-state` points at the last **dispatched** SHA; it does not prove a successful build or publication.

## Schedule and behavior
- `sync-upstream.yml` runs every 3 hours at minute 37 UTC, on manual dispatch, and at installation (push to the sync YAML in the default branch).
- It fetches only Git history and no blobs for official/fork `master`, checks fast-forwardability, and pushes new commits when found.
- A new SHA launches `build-release.yml` on the default branch with that exact 40-character source SHA. This workflow clones `master` and checks out the requested SHA before pulling required submodules.
- Official `nightly.yml` is the base for build-release: Linux x86_64/ARM64 AppImage, Windows x64/ARM64 EXE installers, and macOS ARM64 DMG on the official matrix.
- GitHub Release is created as `autosync-<SHA12>` only if all three build jobs succeed. Each release includes relevant files and checksums/update metadata produced by upstream scripts. This is an *unofficial prerelease*, not a signed official stable build.

## Operational cautions
- Linux x86_64 AppImage typically needs glibc 2.35+, Linux ARM64 glibc 2.39+, Windows x64 at least Windows 10, Windows ARM64 at least Windows 11, and the current macOS ARM64 DMG at least macOS 15. Check upstream release notes for changes.
- darktable nightly builds can change database schema. Keep a separate test library and back up databases before trying a development release.
- Builds require substantial GitHub Actions minutes and storage, especially two Windows environments and AppImage packaging, and might fail when runner images or dependencies change.
- A SHA is marked after successful *dispatch*. A failed build is not automatically rescheduled for the same SHA. Re-run the failed workflow jobs or manually dispatch `build-release.yml` on `darktable-1.1.x` with the same SHA.
- Repo must allow Actions GITHUB_TOKEN write to contents and actions. GitHub schedules are best-effort.
- No automatic build runs when SHA stays unchanged. A new SHA will start new builds without overwriting existing tags.

## Links
- Actions: https://github.com/969981/darktable/actions
- Releases: https://github.com/969981/darktable/releases
- Official nightly reference: https://github.com/darktable-org/darktable/blob/master/.github/workflows/nightly.yml
