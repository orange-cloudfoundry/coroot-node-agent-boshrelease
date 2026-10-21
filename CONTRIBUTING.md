# Contributing

## Commits

All commits MUST be SSH-signed (`git config gpg.format ssh`), see the
paas-templates CONTRIBUTING guide. Conventional-commit messages (`feat:`, `fix:`, ...).

## Bumping the agent version

1. Edit the `tag:` in `vendir.yml` (or wait for the Renovate PR).
2. `./vendor.sh && ./addblob.sh` (requires devbox: vendir, bosh, yq).
3. Verify the checksum printed in `config/blobs.yml` against the sha256 shown on
   the upstream GitHub release page for the pinned asset.
4. Validate locally: `bosh create-release --name coroot-node-agent --version 0.0.0-dev+<x> --tarball ./dev.tgz --force`.
5. PR and merge — CI produces the final release tagged with the upstream version.

## Never commit

- `src/` and `blobs/` (vendir/blob caches, huge binaries)
- `config/private.yml` (blobstore credentials)
- any real API key / endpoint value in manifests (use `-v` / `--vars-store`)
