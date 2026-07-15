# companion-satellite-armbian

A ready-to-flash Armbian image with [Bitfocus Companion Satellite](https://github.com/bitfocus/companion-satellite) pre-installed, targeting 200+ single-board computers (Orange Pi, Banana Pi, Rock Pi, Khadas, NanoPi, and more).

Connect USB control surfaces (Elgato Stream Deck, Loupedeck, Xencelabs Quick Keys, etc.) to an SBC and have them appear in [Bitfocus Companion](https://bitfocus.io/companion) over the network — even across subnets and VPNs.

> See also: [companion-armbian](https://github.com/grace-bible/companion-armbian) (Raspberry Pi–focused variant)

---

## How it works

```
GitHub Actions
  └─ Compile Armbian (your board)
       └─ Packer provisions image
            └─ install.sh sets up Node.js + Companion Satellite
                 └─ Compressed .img.gz → download & flash
```

No local tooling required. Everything builds in CI.

---

## Quick Start

### 1 — Build an image for your board

1. **Fork this repository** (your own fork is needed so you can trigger workflows)
2. In your fork, go to **Actions → Build armbian + build companion satellite**
3. Click **Run workflow** and fill in:
   - **Board** — select your SBC from the dropdown (200+ options)
   - **Companion Version** — leave as `latest` to auto-detect, or type a specific tag like `v2.6.0` (see [releases](https://github.com/bitfocus/companion-satellite/releases))
4. Wait ~30–45 minutes for the build to complete
5. Open the workflow run → **Summary** tab → download `Armbian_firmware.zip`

### 2 — Flash the image

Extract the `.img.gz` from the zip and write it to your SD card or eMMC:

- **GUI:** [Balena Etcher](https://etcher.balena.io/) (supports `.img.gz` directly)
- **CLI:**
  ```bash
  gunzip -c yourboard-companion-satellite-v2.6.0.img.gz | sudo dd of=/dev/sdX bs=4M status=progress
  ```
  Replace `/dev/sdX` with your card's device path.

### 3 — Boot & configure

1. Insert the card, power on, and find the device's IP (check your router or use `arp-scan`)
2. Open the web UI: **`http://<device-ip>:9999`**
3. Enter your Companion server IP/hostname and save
4. Surfaces connected via USB will appear in Companion automatically

#### SSH / serial access (if needed)

Default credentials are set by Armbian first-boot. SSH may be disabled by default — connect via serial console to set a password on first boot.

#### Manual config file

On the device, edit `/home/satellite/satellite-config.json` or use the helper:

```bash
sudo satellite-edit-config
```

Useful on-device commands:

| Command | Description |
|---|---|
| `satellite-update` | Interactively update Companion Satellite |
| `satellite-edit-config` | Edit the config file |
| `satellite-help` | List available commands |
| `satellite-license` | Show license info |

---

## Auto-build your board on every Companion release

When Companion Satellite publishes a new release, [Renovate](https://docs.renovatebot.com/) opens a PR to update the submodule. When that PR merges, the **release workflow** automatically builds images and publishes a GitHub Release.

By default it builds: `orangepizero`, `orangepizero2`, `orangepizero2w`, `orangepizero3`.

**To build your board(s) automatically on every release:**

1. Fork this repository
2. Edit **`.github/release-boards.json`** and add your board name(s):
   ```json
   {
     "armbian-board": [
       "rockpi-s",
       "rock-5b",
       "nanopi-r5s"
     ]
   }
   ```
3. Commit and push — your boards will be built on every future Companion Satellite release

Board names match the `BOARD=` argument accepted by the [Armbian build system](https://github.com/armbian/build). The full list of supported boards is in the workflow dropdown.

---

## Supported boards

The build supports every board in the [Armbian build system](https://github.com/armbian/build) (current branch, Ubuntu Noble minimal). The workflow dropdown includes 200+ boards. Not every board has been tested — if yours works, feel free to open a PR noting it.

**Boards built on every release** (edit `.github/release-boards.json` in your fork to change):

| Board name | Product page |
|---|---|
| `rockpi-s` | [Rock Pi S — Allnet China](https://shop.allnetchina.cn/products/rock-pi-s) |

---

## Local build (advanced)

Builds are designed to run in GitHub Actions, but you can build locally on a Linux host with `packer`, `qemu-user-static`, and `qemu-system-aarch64`:

1. Build a vanilla Armbian image for your board using the [Armbian build system](https://github.com/armbian/build)
2. Run Packer against the resulting `.img`:
   ```bash
   packer init companion-satellite.pkr.hcl
   sudo packer build \
     -var "url=path/to/armbian.img" \
     -var "build=v2.6.0" \
     companion-satellite.pkr.hcl
   ```

---

## License

MIT — see [LICENSE](LICENSE)
