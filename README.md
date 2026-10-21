# BOSH release for coroot-node-agent

Packages the [coroot-node-agent](https://github.com/coroot/coroot-node-agent) (Apache-2.0)
static binary as a BPM-managed BOSH job, for fleet-wide deployment as a BOSH
runtime-config add-on (eBPF metrics, network traces and continuous profiling
pushed to a self-hosted Coroot collector).

In the Orange PaaS, it is wired via `shared-operators/runtime-config/` in
paas-templates-private — see change `add-coroot-bosh-instrumentation`.

## Design notes

- One job (`coroot-node-agent`), one package (upstream static binary, no compilation).
- Runs as `root` under BPM with a least-privilege capability set (see `bpm.yml.erb` D3-SPIKE note).
- Agent HTTP listener defaults to `127.0.0.1:80` (upstream default `0.0.0.0:80` would collide with router VMs).
- API key is passed through the process environment, never in `cmdline`.
- WAL/spool under `/var/vcap/data/coroot-node-agent`, capped (default 200MB) so an unreachable collector cannot fill the disk.
- Each BPM job on a VM shows up as a distinct application in Coroot (cgroup grouping).

## Usage as an add-on (recommended)

See `shared-operators/runtime-config/coroot-node-agent-runtime-operators.yml`
in paas-templates-private. Minimal properties: `coroot.collector_endpoint`, `coroot.api_key`.

## Standalone smoke test

```bash
bosh upload-release # or use the dev tarball from CI artifacts
bosh deploy manifests/coroot-node-agent.yml \
  -v collector_endpoint=https://coroot-00-core-connectivity-k8s.example.internal \
  --vars-store ./dev-creds.yml
```

## Versioning

The bosh release version mirrors the upstream agent version pinned in `vendir.yml`
(e.g. bosh release `1.36.4` == agent `v1.36.4`), like the sibling k3s-packages release.

## Update flow

1. Renovate opens a PR bumping the tag in `vendir.yml` (patch versions are automerged).
2. The `create-final-bosh-release` workflow runs `vendor.sh` (vendir sync) +
   `addblob.sh` (bosh add-blob/upload-blobs), commits `config/blobs.yml` +
   `vendir.lock`, then creates the final bosh release tagged with the upstream
   version and attaches the release tarball to the GitHub release.

Locally:

```bash
./vendor.sh   # downloads the pinned asset into src/ (gitignored)
./addblob.sh  # refreshes config/blobs.yml (+ blobs/ when no blobstore creds)
bosh create-release --name coroot-node-agent --version 0.0.0-dev+local \
  --tarball ./coroot-node-agent-dev.tgz --force
```

## Blobstore bootstrap (TODO)

Final releases require the s3-compat GCS bucket `orange-coroot-node-agent-boshrelease`
and the `AWS_BOSH_ACCES_KEY_ID` / `AWS_BOSH_SECRET_ACCES_KEY` repo secrets
(same pattern as orange-k3s-wrapper-boshrelease). Until then only dev releases work.

## Upstream provenance

| | |
|---|---|
| Upstream | https://github.com/coroot/coroot-node-agent |
| Current pin | v1.36.4, asset `coroot-node-agent-amd64`, sha256 recorded in `config/blobs.yml` and `vendir.lock` |
| Upstream license | Apache-2.0 (BPF code GPLv2) |
