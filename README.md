# Talos Companion and Omni — Unraid templates

Installation templates for Talos Companion and Sidero Labs Omni.
This repository contains no Rust/React application source, credentials, or user backups.
The template, icon, and documentation use MIT. Talos Companion's source is also MIT:
https://github.com/holblin/talos-companion . Omni has its own upstream license and EULA;
the template's MIT license does not change those terms.

## Omni template

See [the Omni installation guide](omni-install.md) and
[example configuration](omni-config.example.yaml). The template pins the official
`ghcr.io/siderolabs/omni:v1.13.2` image. Prepare configuration, TLS, encryption keys,
identity and your own EULA acceptance before starting. Do not replace an existing Omni
container or reuse its database without following the upstream migration procedure.
This repository does not install, stop, or change your current Omni deployment.

## Manual template installation

Public template URLs:

- https://raw.githubusercontent.com/holblin/talos-companion-unraid/main/templates/talos-companion.xml
- https://raw.githubusercontent.com/holblin/talos-companion-unraid/main/templates/omni.xml

Copy the desired XML into `/boot/config/plugins/dockerMan/templates-user/` as
`my-talos-companion.xml` or `my-omni-siderolabs.xml`, then open Docker > Add Container.
Community Apps catalog inclusion is a separate review; a public repository alone does
not mean the templates already appear in its catalog.

## Image

Docker Hub: https://hub.docker.com/r/holblin/talos-companion

Image: `holblin/talos-companion:latest`. Linux amd64 only. The image has exactly one filesystem
layer containing a static Rust binary and compiled frontend files.
Version and `sha-<commit>` tags are available for published versions.

## Install on Unraid

For a fresh Omni plus Companion with LAN, Cloudflare DNS or Tailscale certificates,
follow the [illustrated home setup and migration guide](https://github.com/holblin/talos-companion/blob/main/docs/home-setup.md).
It includes existing-instance configuration checks, scoped repairs and owner password recovery.
The optional Omni managed configuration path must point only to the helper's dedicated
overlay directory, never Omni's database or encryption keys.

1. Create `/mnt/user/appdata/talos-companion` with ownership `99:100` and mode `700`.
2. Install this template through Community Apps after the listing is approved, or copy
   `templates/talos-companion.xml` to
   `/boot/config/plugins/dockerMan/templates-user/my-talos-companion.xml` on your server.
   Then open Docker > Add Container and choose talos-companion.
3. Set PUBLIC_URL to the exact address you will use, including scheme and port.
   Use HTTPS at your reverse proxy for deployment beyond a trusted local network.
4. Leave secret overrides empty to generate private persistent secrets automatically,
   or supply your own SETUP_TOKEN (24+ characters) and BACKUP_PASSPHRASE (16+ characters).
   Find the generated setup token in first-start container logs. At Review, save the shown
   backup passphrase outside this server, check the saved box, and re-enter it to confirm.
5. Start the container and open its WebUI. The wizard guides account creation or restore,
   backup scheduling, retention, and connection preparation. All backup settings can
   be updated later in Settings and persist in `/data/config.json`.
6. Open Identity to configure Omni's callback and copy the OIDC and S3 credentials.
   Expose the S3 receiver (container port 9000) to Omni, not to the public internet.
7. In Backups, upload Omni recovery keys/configuration and create your first recovery file.

The setup token, account password, and backup passphrase are different secrets.
The web listener is port 8080; the S3 receiver starts after initialization on port 9000.
The wizard can also encrypt a recovery copy of the passphrase to age or SSH public keys,
including supported keys imported from GitHub. Keep the matching private key yourself.
Decrypt the note locally with age, then use its recovered passphrase with the .tcb backup.

## Optional NFS / SMB destination

Mount the share on the Unraid host and map that existing mount to `/destinations/network`.
Only then set MOUNTED_BACKUP_DIR to `/destinations/network`. The container requires
no mount privileges or SMB/NFS utilities. Confirm the host mount is actually available;
an empty directory at the mountpoint does not prove the share is connected.

## Recovery

Restore on an uninitialized data directory using a version 2 `.tcb` recovery file and
its original passphrase. You can override the first administrator's credentials while
preserving identity and other users. Restore includes accounts, integration credentials,
vault files, received snapshots, and schedule/retention configuration. Future backups
use the replacement container's BACKUP_PASSPHRASE. Allow disk space for both the uploaded
encrypted file and extracted contents, and configure proxy limits for large uploads.

Companion does not collect Omni encryption keys or snapshot Omni's own database automatically.
Configure Omni to send supported S3 backups and explicitly upload its recovery keys/configuration.
Test recovery with your Omni deployment before relying on it.

## Support

Report problems at https://github.com/holblin/talos-companion-unraid/issues. Do not attach passwords, tokens, encryption keys,
or recovery files to issues. This project is independent of Sidero Labs and Talos Omni.
