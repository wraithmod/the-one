# Windows 11 Network Install Runbook

Purpose: install Windows 11 onto a new PC with an empty M.2 drive, using the existing `windows11.iso` served from another computer on the same LAN. The target PC will be wired. The serving computer may be on Wi-Fi, as long as Wi-Fi and Ethernet clients are on the same broadcast domain and the AP does not isolate clients.

## Outcome

- PXE boot the new PC from the network.
- Install Windows 11 from the existing ISO.
- Prefer a local account over a Microsoft account.
- Disable or minimize Copilot, OneDrive, Widgets/news/MSN, advertising ID, location services, and consumer suggestions.
- Keep the deployment files and notes versioned in this repository.

## Assumptions

- The target PC supports UEFI PXE boot.
- The Windows ISO is legitimate and already downloaded.
- The serving computer can run a temporary PXE service.
- Router DHCP remains authoritative unless explicitly changed.
- Secure Boot may need to be temporarily disabled if the PXE loader does not boot cleanly.
- Windows 11 Pro gives more policy control than Home. Home can still be hardened with registry and PowerShell, but some Group Policy tooling is absent.

## Recommended Tooling

Use iVentoy for the network install path.

Why:

- TFTP by itself is too limited for serving a full Windows ISO efficiently.
- PXE normally uses TFTP only for early boot files, then switches to a better transfer path.
- iVentoy is built for booting ISO files over the network.
- iVentoy can attach Windows unattended XML without rebuilding the ISO.

Keep Rufus as the USB fallback. Rufus is excellent for creating customized USB Windows install media, including Windows 11 account/OOBE bypass options, but it is not the primary tool for serving an ISO over the LAN.

Avoid Balena Etcher for this workflow. It writes images but does not provide the Windows customization controls needed here.

References:

- iVentoy: https://www.iventoy.com/en/
- iVentoy auto install/unattend support: https://www.iventoy.com/en/doc_autoinstall.html
- Microsoft OOBE automation: https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/automate-oobe
- Microsoft `HideOnlineAccountScreens`: https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hideonlineaccountscreens
- Microsoft `UserAccounts`: https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-useraccounts
- Microsoft Windows Widgets overview: https://learn.microsoft.com/en-us/windows/apps/design/widgets/

## Host Preparation

On the computer that will host the ISO:

1. Install or extract iVentoy.
2. Place `windows11.iso` in the iVentoy ISO directory.
3. Put this repository somewhere easy to update, for example:

   ```bash
   git clone <repo-url> the-one
   ```

4. Copy or generate the Windows unattended XML into this repo, preferably:

   ```text
   scripts/windows11/autounattend.xml
   scripts/windows11/post-install-privacy.ps1
   scripts/windows11/SetupComplete.cmd
   ```

5. In the iVentoy web UI, map the Windows ISO to the unattended XML file.
6. Start iVentoy.

Network checks:

- Host and target are on the same VLAN/subnet.
- AP client isolation is disabled.
- Wired clients can receive PXE broadcasts.
- Firewall on the host allows iVentoy.
- If the router already provides PXE options, avoid conflicting DHCP/PXE settings.

## Target Firmware Setup

On the new PC:

1. Enter firmware setup.
2. Confirm the M.2 drive is detected.
3. Enable UEFI network boot/PXE for the wired NIC.
4. Put wired PXE before the empty M.2 drive in boot order, or use the one-time boot menu.
5. If PXE fails before showing the iVentoy menu, temporarily disable Secure Boot and retry.
6. Boot from wired PXE.

## Unattended Windows Setup Strategy

Use Windows unattended XML for setup and OOBE behavior.

Minimum goals:

- Create a local admin account.
- Hide online-account screens where supported.
- Hide wireless setup during OOBE.
- Set locale, keyboard, and timezone.
- Avoid storing a real long-term password in the repo.

Important security note:

- Unattend XML can expose account passwords if committed.
- Prefer a temporary local password that is changed after first boot.
- Do not commit personal passwords, Microsoft account credentials, Wi-Fi passwords, license keys, or secrets.

Suggested account default:

- Username: `owner`
- Group: `Administrators`
- Temporary password: set outside git or replace before use.

Suggested OOBE controls:

- `HideOnlineAccountScreens=true`
- `HideWirelessSetupInOOBE=true`
- `HideOEMRegistrationScreen=true`
- `NetworkLocation=Other` or `Work`, depending on preference
- `ProtectYourPC=3` if choosing the least express-style OOBE posture

Windows 11 Home note:

- Microsoft changes OOBE behavior over time.
- If Home ignores the local-account path while online, disconnect the network during OOBE or use Rufus USB as fallback.
- Windows 11 Pro is the cleaner target for policy-driven local account setup.

## Post-Install Privacy And Debloat Strategy

Use a controlled PowerShell script, not a broad third-party debloat script.

Disable or reduce:

- Copilot access via policy/registry where supported.
- OneDrive auto-start; optionally uninstall OneDrive.
- Widgets taskbar entry and widget/feed surface.
- News/MSN surfaces where controlled by Widgets, Edge, or consumer content settings.
- Advertising ID.
- Tailored experiences and suggested content.
- Location services.
- Consumer feature provisioning.
- Unwanted provisioned AppX packages by explicit list only.

Do not disable blindly:

- Windows Update.
- Microsoft Defender.
- Driver update mechanisms.
- Store framework packages required by other apps.
- Networking, firewall, cryptographic, identity, or servicing components.

Conservative removable-app candidates:

- Clipchamp
- Microsoft News
- Microsoft People
- Microsoft Solitaire Collection
- Microsoft Teams consumer/chat package if present
- Microsoft To Do if unwanted
- Spotify or other third-party promotional packages if present
- Xbox apps if this machine will not use Game Pass, Xbox services, or related features

Keep unless intentionally removed:

- Microsoft Store
- App Installer
- Windows Terminal
- Photos, Paint, Notepad, Calculator unless you have replacements
- HEIF/AV1/Web media extensions if media compatibility matters

## Validation Checklist

After install:

- Local account login works.
- No Microsoft account is attached.
- Windows is activated or ready for activation.
- Windows Update runs successfully.
- Device Manager has no unknown critical devices.
- OneDrive is not running or is uninstalled, depending on selected profile.
- Copilot icon and access path are disabled where policy supports it.
- Widgets/news/taskbar feed is hidden or disabled.
- Advertising ID is off.
- Location services are off.
- No unwanted provisioned apps return after reboot.
- Restore point or system image is created after the clean baseline is confirmed.

Useful Windows validation commands:

```powershell
whoami
Get-LocalUser
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
Get-AppxPackage -AllUsers | Sort-Object Name | Select-Object Name, PackageFullName
Get-AppxProvisionedPackage -Online | Sort-Object DisplayName | Select-Object DisplayName, PackageName
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo" -ErrorAction SilentlyContinue
Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsCopilot" -ErrorAction SilentlyContinue
```

## Fallback Path: Rufus USB

Use Rufus if PXE is blocked by firmware, AP isolation, DHCP behavior, or Secure Boot.

1. Create a Windows 11 USB installer from `windows11.iso`.
2. Use Rufus Windows User Experience options to reduce Windows 11 setup friction.
3. Select local account / privacy options where available.
4. Install from USB.
5. Run the same post-install privacy script from this repository.

## Repo Sync Plan

This repository should hold only repeatable deployment material:

```text
plans/
  windows11-network-install-runbook.md
scripts/
  windows11/
    autounattend.xml
    post-install-privacy.ps1
    SetupComplete.cmd
```

Recommended `.gitignore` entries:

```gitignore
.venv/
*.iso
*.wim
*.esd
*.swm
*.vhd
*.vhdx
*.log
secrets/
local/
```

After a remote is configured:

```bash
git pull --rebase
git status
git add AGENTS.md model-defs plans scripts .gitignore
git commit -m "docs: add windows 11 network install runbook"
git push
```

On the image-hosting computer:

```bash
git clone <repo-url> the-one
cd the-one
git pull --rebase
```

If no remote exists yet, create one on GitHub, Gitea, Forgejo, GitLab, or another reachable Git host, then run:

```bash
git remote add origin <repo-url>
git branch -M main
git push -u origin main
```

## Open Decisions

- Host OS: Linux, Windows, or macOS.
- Windows edition: Home, Pro, or multi-edition ISO.
- Local account name and temporary password handling.
- Whether OneDrive should be fully uninstalled or only disabled.
- Conservative or aggressive app removal profile.
- Whether Secure Boot must remain enabled.
- Git remote provider and repository URL.

