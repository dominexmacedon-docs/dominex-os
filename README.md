# Dominex OS — Releases and Distribution

This repository is the **distribution and explanation layer** for Dominex OS. Development, source code, image construction, command tests, and the primary QEMU boot gate live in [Dominex-Core](https://github.com/dominexmacedon-docs/Dominex-Core). This repository publishes the resulting downloads and explains how to use them.

## Fully automated publishing

After a Dominex-Core build completes its compile checks, image build, QEMU boot, and in-guest command self-test, its workflow sends a `dominex-core-image-ready` event here. The **Publish Dominex OS Release** workflow then:

1. Confirms the supplied source run is a successful **Build Dominex Cloud Image** run.
2. Downloads the exact artifact ID created by that run and verifies the artifact name, source run, archive, and SHA-256 checksum.
3. Converts the QCOW2 disk to a gzip-compressed raw `.img.gz` image.
4. Generates and verifies checksums for the published images.
5. Writes release notes containing source commit, build-run link, artifact ID, validation scope, and platform limitations.
6. Creates a GitHub Release tagged `build-RUN_ID`, attaching the raw image, QCOW2 image, checksum manifest, and release notes.

The **Test Dominex Core Boot** workflow also runs automatically after each successful build and independently boots the uploaded artifact under QEMU. The release itself is gated by the source workflow's boot and in-guest self-test, so a failed source boot does not dispatch a release.

You can also run the publishing workflow manually by providing a successful Dominex-Core build run ID and its artifact ID. Manual publishing still verifies that the artifact belongs to the specified successful build.

## One-time GitHub configuration

Create a fine-grained personal access token with access to both repositories and store the same token as two repository Actions secrets:

- `DOMINEX_OS_TOKEN` in **Dominex-Core**.
- `DOMINEX_CORE_TOKEN` in **dominex-os**.

Required token permissions:

- **Dominex-Core:** Actions — Read.
- **dominex-os:** Contents — Read and write.

The token lets Dominex-Core dispatch the event and lets this repository read and validate the source artifact. The release workflow uses its repository-scoped `GITHUB_TOKEN` to create the release. Never commit the token into either repository.

## Release assets

- **`Dominex-Core-amd64.img.gz`** — gzip-compressed raw disk image. Decompress it before writing to a physical disk or importing it into a platform that accepts raw images. **Writing an image to a disk erases the target disk's existing contents.**
- **`Dominex-Core-amd64.qcow2`** — QCOW2 virtual disk for compatible QEMU/KVM-based platforms.
- **`SHA256SUMS`** — SHA-256 checksums for both published images.
- **`RELEASE_NOTES.md`** — build provenance, validation scope, and current limitations.

Check a downloaded asset against the checksum file before using it. The raw image is not a universal installer: cloud platforms have different import requirements, and the current image targets amd64/x86_64 with a legacy-BIOS boot layout.

## Current platform scope

The image is a headless Ubuntu 24.04-derived system with the generic Linux kernel and cloud-init. It does not include a desktop environment. The current disk layout targets legacy BIOS boot; UEFI-only machines and provider-specific image import formats are not guaranteed. Cloud-init behavior depends on the platform's metadata and user-data support.

Ubuntu's package servers are used when building the base image, and the image retains Ubuntu APT sources for package updates. Publishing the image through this repository does not by itself make the OS independent of Canonical's repositories. Custom Dominex utilities can be distributed through their own versioned releases and integrated into the build after their repositories and assets are defined.

## Development

Open [Dominex-Core](https://github.com/dominexmacedon-docs/Dominex-Core) for source, build scripts, command implementation, and tests. This repository is for downloadable results, release history, and user-facing explanations.
