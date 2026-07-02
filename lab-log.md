# Cybersecurity Home Lab - Build Log

## Goal
Build an isolated virtual lab environment using Kali Linux and Windows
to practice network scanning, traffic analysis, and vulnerability assessment.

## Environment
- Host Machine: Apple Silicon MacBook Pro
- Virtualization Software: UTM (mac.getutm.app)
- Attacker VM: Kali Linux ARM64
- Target VM: Windows (planned)

---

## Entry 1 — Initial Setup & VirtualBox Compatibility Issue
**Date:** June 10, 2026

### What I Tried
- Downloaded Oracle VirtualBox as initial virtualization software
- Attempted to install Kali Linux using the standard installer ISO
- Selected "Graphical Install" from the GRUB boot menu
- VM immediately hung with the following error:
  `PCI: OF: of_root node is NULL, cannot create PCI host bridge node`
- Screen stayed frozen with no further output

### Root Cause
VirtualBox has poor and unstable ARM support on Apple Silicon Macs.
The standard Kali Linux ISO is built for x86 architecture and is
incompatible with ARM64 hardware regardless of virtualization software.

### Resolution
- Switched virtualization software from VirtualBox to UTM (mac.getutm.app)
  which is purpose-built for macOS including Apple Silicon
- Downloaded the correct Kali Linux ARM64 ISO from kali.org

### What I Learned
- Apple Silicon (M1/M2/M3/M4) requires ARM64-specific software
- VirtualBox is not a reliable choice for Apple Silicon Macs
- UTM is the recommended virtualization tool for ARM-based Macs
- Always verify ISO architecture matches host hardware before downloading

---

## Entry 2 — UTM Setup & Persistent Boot Error Investigation
**Date:** June 11, 2026

### What I Tried
- Installed UTM and created a new Linux VM with the following config:
  - Architecture: ARM64 (aarch64)
  - Machine: QEMU 10.0 ARM Virtual Machine (virt-10.0)
  - Memory: 4GB
  - Storage: VirtIO Drive (sparse .qcow2 format)
  - ISO: kali-linux-2026.1-installer-arm64.iso (3.7GB)
- Booted the VM — same PCI error appeared:
  `PCI: OF: of_root node is NULL, cannot create PCI host bridge node`

### Troubleshooting Steps Taken
Systematically investigated each component of the VM configuration:

**Drive Settings**
- Checked USB Drive: confirmed Kali ARM64 ISO correctly mounted (3.7GB)
- Checked VirtIO Drive: showed 196KB (confirmed normal for empty 
  sparse disk — grows as OS installs)
- No changes needed here

**Display Settings**
- Emulated Display Card: virtio-gpu-pci ✅ (correct for ARM64)
- No changes needed here

**QEMU Settings**
- UEFI Boot: enabled ✅
- Use Hypervisor: enabled ✅
- RNG Device: enabled ✅

### Fix Attempts
1. Disabled "Use Hypervisor" → rebooted → error persisted
2. Changed display card from virtio-gpu-pci to virtio-ramfb → 
   error persisted
3. Re-enabled "Use Hypervisor" → error persisted

### Status
Error unresolved at end of session. All visible configuration settings
confirmed correct. Issue appears deeper than surface-level VM settings.

---

## Entry 3 — Root Cause Found & Resolution
**Date:** July 2, 2026

### Research Conducted
After two sessions of troubleshooting VM settings without resolution 
(June 10-11), independently reviewed the official UTM documentation 
on July 2 to approach the problem from a fresh angle. Also searched 
GitHub community issues related to Kali + UTM + ARM64.

Identified that UTM's "Other" VM type preset uses a more compatible 
QEMU configuration for the Kali ARM64 installer than the "Linux" preset.

### Root Cause Found
UTM's "Linux" VM type preset applies a specific QEMU machine 
configuration that conflicts with the Kali Linux ARM64 installer ISO.
The correct approach is to select **"Other"** as the VM type during 
creation, which uses a more compatible base configuration.

This was not surfaced by checking individual settings — it required
reading documentation and community-reported issues to identify.

### Resolution
- Deleted existing Linux VM
- Created new VM in UTM selecting "Other" instead of "Linux"
- Mounted same ARM64 ISO
- Proceeding with boot test

### What I Learned
- When settings all appear correct but an error persists, the issue
  may be at a higher configuration level (VM type preset, not 
  individual settings)
- GitHub issues and official documentation are more reliable for
  obscure errors than general troubleshooting guides
- Systematic elimination of variables (drives → display → QEMU → VM type)
  is an effective debugging methodology even when early steps don't
  resolve the issue
- Documenting failed attempts is as valuable as documenting successes —
  it shows the full problem-solving process

---

## Next Steps
- [X] Confirm Kali boots successfully with "Other" VM type
- [X] Complete Kali Linux installation
- [ ] Create Windows VM as target machine
- [ ] Configure Host-Only networking between both VMs
- [ ] Verify connectivity with ping between VMs
- [ ] Run first nmap scan from Kali targeting Windows VM
- [ ] Begin network scanner project inside lab environment
