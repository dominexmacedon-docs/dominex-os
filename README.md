# Dominex OS — Releases and Distribution

This repository is the **distribution and explanation layer** for Dominex OS. Development, source code, image construction, command tests, and QEMU boot tests live in [Dominex-Core](https://github.com/dominexmacedon-docs/Dominex-Core).

## Automated publishing

After a Dominex-Core build completes its image and command checks, its workflow sends a `dominex-core-image-ready` event here. This repository's `Publish Dominex OS Release` workflow then:

1. Downloads the exact build artifact ID emitted by Dominex-Core.
2. Verifies the artifact belongs to the reported workflow run and checks its checksum.
3. Converts the QCOW2 disk to a raw `.img.gz` image.
4. Publishes the raw image, QCOW2 image, checksums, and release notes as a GitHub Release.

Releases are tagged `build-RUN_ID`, so each build has a unique tag. The release workflow can also be run manually with an artifact ID and its originating run ID.

## One-time GitHub configuration

Create a fine-grained personal access token and add it as a repository Actions secret named `DOMINEX_OS_TOKEN` in **Dominex-Core** and `DOMINEX_CORE_TOKEN` in **dominex-os**. Grant the token access to both repositories with:

- **Dominex-Core:** Actions — Read
- **dominex-os:** Contents — Read and write

The token is used only to read the completed source artifact and dispatch the publishing event. Never commit the token into either repository. The publishing workflow is deliberately event-driven; it does not poll or continuously rebuild.

## What the downloads mean

- `Dominex-Core-amd64.img.gz`: gzip-compressed raw disk image, suitable for writing to a disk after decompression. **Writing an image erases the target disk.**
- `Dominex-Core-amd64.qcow2`: virtual disk image for QEMU/KVM-compatible platforms.
- `SHA256SUMS`: SHA-256 checksums for both published images.
- `RELEASE_NOTES.md`: build provenance, validation scope, and current limitations.

## Current platform scope

The image is amd64/x86_64 and based on Ubuntu 24.04 userspace plus the Linux generic kernel. The current image builder configures a legacy-BIOS boot disk; do not assume UEFI-only machines or every cloud import format are supported until those boot paths are separately built and tested. The guest uses cloud-init and SSH public-key authentication. It is headless and does not include a desktop environment.

A successful release means the source workflow's build and static command tests passed. Run the separate **Test Dominex Core Boot** workflow in Dominex-Core to verify the exact artifact actually boots under QEMU and completes its in-guest self-test before treating it as a validated boot release.

## Development

Open [Dominex-Core](https://github.com/dominexmacedon-docs/Dominex-Core) for source, build scripts, command implementation, and tests. This repository is for downloadable results, release history, and user-facing explanations.
