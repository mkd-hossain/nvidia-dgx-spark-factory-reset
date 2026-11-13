# Dell FCM1253 Ethernet recovery: observed trial and troubleshooting

These notes add Dell findings from 2026-11-13. The tested ASUS workflow, media,
defaults and instructions remain separate and unchanged. Dell recovery is a
community Ethernet procedure using Dell media; Dell documents USB recovery.

## What was actually observed

- Windows host with a dedicated direct Ethernet cable, negotiated at 1 Gbps.
- Exact Dell Pro Max with GB10 FCM1253 identity and a 4 TB NVMe SSD verified
  before boot. The Worldwide OTA3 ISO matched its pinned full SHA-256.
- Original kernel and initrd loaded through ARM64 PXE; the full ISO loaded
  into RAM over HTTP. Operator checks confirmed read-only media, the Dell OEM
  package and hidden NVMe devices before enabling installation.
- The first Dell partitioned/formatted the approved disk, installed packages
  and reached the Dell first-time welcome screen. Initial account setup and
  post-setup GPU checks were left for later. Host cleanup remains required.
- A second Dell passed the same shell checks. After installation was enabled,
  the operator confirmed successful installation. Its final setup screen and
  post-setup GPU checks were not independently inspected.

This is evidence for the Worldwide terminal path on these systems, not a
qualification of US media, all firmware versions, macOS/Linux hosts, or the
automatic desktop observer. No vendor image or device credentials are published.

## Switching to another Dell

Revoke installation in the old session before swapping cables. Reuse the
verified ISO, but create a new session with the new target MAC, network plan,
inventory and SSD serial. Never copy the old diagnostic or erase receipts.
Stop the old server before starting the new one; only one DHCP/TFTP/HTTP server
may own the recovery ports. Check the listening ports and new server readiness
before asking the operator to reboot.

An Ethernet link being up does not imply the target has an IPv4 address.
Docker addresses are not the direct Ethernet address. Existing scoped IPv6
link-local discovery and authenticated SSH inspection worked on the second Dell.
Verify the SSH identity and enter passwords only in the local masked prompt.
Passwords must not be written into scripts, logs or chat.

## Windows service startup

Internet Connection Sharing (SharedAccess) reclaimed UDP 67 during the trial.
Check the port owner before starting recovery. With operator-approved network
changes, stop conflicting sharing and verify the dedicated adapter still owns
the planned server address. Stopping ICS can remove that address. Restore the
address only on the selected isolated Ethernet adapter; preserve the original
network/sharing configuration for cleanup. A running SharedAccess service alone
does not prove there is a conflict: inspect its active bindings.

In Windows PowerShell, native Python logging on stderr combined with
`$ErrorActionPreference='Stop'` and merged redirection caused a launcher to stop
on an ordinary INFO message. A separate Python process with separate stdout and
stderr log files avoided this problem. Process creation is not readiness:
wait for successful DHCP, TFTP and HTTP binding and the target-specific serving
message. Do not use the one-off machine-specific troubleshooting scripts as
portable launchers.

## Boot menu and Secure Boot

For this model, Dell documents **F7** for the one-time boot menu and **Delete**
for BIOS Setup. Tap repeatedly during restart; a directly connected wired USB
keyboard can help when a wireless receiver misses the startup window.

- [Dell: one-time boot menu](https://www.dell.com/support/manuals/en-ed/dell-pro-max-fcm1253-micro/dell_pro_max_with_gb10_fcm1253_om/One-time-boot-menu?guid=guid-2734c8d7-80ad-4f53-9887-9a2c0e238925&lang=en-us)
- [Dell: entering BIOS Setup](https://www.dell.com/support/manuals/en-ca/dell-pro-max-fcm1253-micro/dell_pro_max_with_gb10_fcm1253_om/entering-bios-setup-program?guid=guid-2cf5a0b0-3747-417a-83d6-04a09584dcce&lang=en-us)

The second Dell reported an invalid-signature Secure Boot violation after TFTP
completed the same bootloader bytes used by the first Dell. It later reached
GRUB and diagnostics, but the intervening firmware change was not recorded.
Do not claim that F7 fixed signature validation or that Secure Boot was disabled.
Inspect the actual firmware policy and boot chain before recommending changes;
do not clear enrolled keys. Record any approved setting change for restoration.
No SSD erase occurs merely from a firmware bootloader rejection.

## Diagnostic shell and installation progress

`break=casper-bottom` intentionally stops at `(initramfs)`. Run:

```sh
cat /proc/cmdline
mount | grep /cdrom
ls /cdrom/oemdata/debs/dell*
ls /dev/nvme*
```

Check for diagnostic kernel arguments, `ro` on `/cdrom`, the Dell wallpaper
package, and no NVMe device nodes. Do not treat the shell or overlay fallback
messages alone as an installation failure. Do not exit the shell to install:
record the diagnostic checks, enable the session-bound cancel-first install menu
only after erase authorization, and boot that entry separately. Use the verified
device's current firmware boot entry; do not assume another device's entry number.

The GRUB message announcing erasure precedes actual disk writes. Partitioning and
formatting messages show that writes have started. During late package setup,
the console displayed one long apt command while drivers, CUDA and desktop
packages were installed. Less than 15 minutes without a new console line was
not evidence of a hang; the first Dell subsequently reached welcome setup.
Ping responsiveness alone cannot prove installer progress. If progress is in
doubt, inspect installer logs/processes before restarting or retrying. Keep both
machines powered, the host awake and Ethernet connected throughout installation.

After confirmed first boot, revoke the erase menu and stop recovery services,
then restore the journaled Windows networking/sharing and remove temporary
firewall rules. Factory reset creates new SSH identity/account setup; do not
silently accept a changed host key.
