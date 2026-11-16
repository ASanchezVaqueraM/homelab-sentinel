# Logbook – Homelab-Sentinel (COM6016M)

Working log for the dissertation. Each entry follows the template:
**What I did / Results / Problems / Decisions / AI use / Next step.**

---

## 2026-11-09 — Initial review of the project and the brief
- **What I did:** Reviewed the repository and the submitted proposal. Read the COM6016M assessment brief (7,500 words, deadline 26/05/2027; ethics form due 04/12/2026).
- **Results:** Identified inconsistencies in the proposal: the hypothesis is not measurable, "anomaly detection" is undefined, and requesting a 16 GB Raspberry Pi 5 contradicts the "resource-constrained" framing. Also possible flaws in the scripts: injection in telegram-notify, broken login alert for non-root users, and an incorrect Fail2ban backend.
- **Decisions:** Keep the topic and reframe it as an evaluation with metrics and a comparison baseline (Wazuh), subject to the supervisor's feedback. Tagged the pre-dissertation state of the repository as `v0-baseline` (commit `e61722b`).
- **AI use:** Used Claude to review the repository, the proposal and the brief, and to organise a plan. The conversations are saved in OneDrive (Dissertation/AI-evidence). The flaws listed above are still to be verified by me on the Pi.
- **Next step:** Email Alec Wells (allocated supervisor) to request a first meeting.

## 2026-11-10 — Supervisor and technician feedback
- **What I did:** Received feedback from Alec Wells, my supervisor, and Norbert Dajnowski. Reply to both.
- **Results:** Alec suggested comparing my system with a traditional approach through a pen test, using FAR, TAR, FRR and TRR. Norbert offered a Jetson Orin Nano (8 GB) instead of the Raspberry Pi 5.
- **Problems:** Not yet clear what "traditional approach" refers to.
- **Decisions:** Accept the Jetson. Pending: rule-based only or hybrid (rules + ML).
- **AI use:** Used Claude to map Alec's biometric metrics to IDS terms and to find candidate studies. I still need to read them myself.
- **Next step:** Read Garalov & El-hajj, merge the logbook PR.

## 2026-11-11 — Reading papers
- **What I did:** Read 10 papers about IDS on Raspberry Pi and Jetson, and about mistakes when evaluating ML.
- **Results:** No paper compares Suricata and ML on the same device with live attacks. Some papers have numbers that don't add up.
- **Decisions:** Add a simple baseline besides Suricata. Check the content of each attack flow when labelling it.
- **AI use:** Claude helped me summarise the papers. I still need to check the key numbers myself.
- **Next step:** Download and fill in the ethics form.

## 2026-11-11 — Supervisor confirmation and Jetson loan
- **What I did:** Read the replies from Alec Wells and Norbert Dajnowski and replied to both. Filled in a draft of the ethics application
- **Results:** Alec confirmed that the baseline should be a rule-based IDS (Suricata or Snort). Norbert will borrow me a Jetson Orin Nano with a 64 GB microSD card, a display cable and a switch. I need to sign a loan agreement.
- **Decisions:** Use Suricata (or Snort) as the main baseline, plus a simple packet-rate baseline.
- **AI use:** Claude helped me read the emails, compare JetPack versions and draft the reply.
- **Next step:** Collect the Jetson and start the ethics form (due 4 December).

## 2026-11-12 — Jetson collected and first boot attempt (written on 2026-11-16)
- **What I did:** Collected the Jetson Orin Nano and an unmanaged switch from Norbert. Asked him to order a USB 3.0 to Gigabit Ethernet adapter (TP-Link UE300), arriving next week. Tried to boot the Jetson.
- **Results:** The switch has no port mirroring, so I will run the Jetson as an inline bridge with the USB adapter as the second port. The Jetson came without an OS. The green LED turns on, but there is no image on DP-to-DP or DP-to-HDMI, even though the monitor works with other devices. Windows shows an "Unknown USB device" when connected by USB-C.
- **Problems:** The Jetson may be in recovery mode or have old firmware that needs updating.
- **Decisions:** Flash JetPack 5.1.3 on the microSD to update the firmware, then JetPack 6.2.x. If it still doesn't boot, ask Norbert to flash it with SDK Manager.
- **AI use:** Claude helped me compare USB adapters and troubleshoot the boot.
- **Next step:** Flash JetPack 5.1.3 and try a headless boot over USB-C.

## 2026-11-16 — Jetson Orin Nano: OS installation, upgrade and remote access

### What I did
- Diagnosed the earlier boot failure: the QSPI firmware was already at 36.4.3 (JetPack 6.2), so the
  JetPack 5.1.3 "bridge" SD image could not boot (L4T 35.x kernel on 36.x firmware) and hung after
  `L4TLauncher: Attempting Direct Boot`.
- NVIDIA provides no SD card image for JetPack 6.2.2. Flashed the JetPack 6.2.1 image (L4T 36.4.4)
  and upgraded to JetPack 6.2.2 (L4T 36.5) via APT (`r36.4` → `r36.5` in
  `nvidia-l4t-apt-source.list`, `apt dist-upgrade`).
- Completed the initial configuration (oem-config) headless over the USB-C serial console
  (115200 baud), as no keyboard or mouse was available.
- Applied a minimal host firewall (`ufw`: default deny incoming; SSH and DHCP allowed only on the
  USB network interface `l4tbr0`).
- Configured key-based SSH (ed25519 with passphrase) and disabled password and root login via
  `/etc/ssh/sshd_config.d/10-hardening.conf`.

### Results
- `/etc/nv_tegra_release`: R36 REVISION 4.4 → **R36 REVISION 5.2** (JetPack 6.2.2).
- Bootloader firmware (`nvbootctrl dump-slots-info`): **36.5.2**, capsule status 0, both A/B slots normal.
- Ubuntu 22.04.5 LTS, kernel `5.15.199-tegra`; default power mode 25 W (mode 1).
- `sshd -T` confirms `passwordauthentication no`, `permitrootlogin no`; password login attempt
  rejected with `Permission denied (publickey)`.
- Evidence: `evidence/jetson-setup/` (command outputs, session log, screenshots).

### Problems
1. Wrong bridge step (JetPack 5.1.3) caused a boot hang; root cause was not checking the firmware
   version first.
2. `ufw` and `tcpdump` are not included in the L4T image; a temporary `iptables` rule set protected
   the device on the shared residence Wi-Fi until `ufw` could be installed.
3. SSH over USB failed for four separate reasons, isolated with `tcpdump` on the Jetson and
   `Get-NetAdapter`/`Find-NetRoute` on Windows:
   - `ufw` blocked DHCP (UDP 67) on `l4tbr0`, so the host could not renew its lease;
   - Windows bound two adapters (RNDIS and NCM) to the same bridge;
   - an active VPN kill-switch silently dropped LAN traffic on the host;
   - the USB adapter fell back to APIPA (169.254.x.x), so traffic to 192.168.55.1 was routed to the
     residence network instead. Fixed with a static IP (192.168.55.100/24, no gateway).
4. Without network access or an RTC battery, the system clock boots at a stale date, which breaks
   APT/TLS until NTP synchronises.

### Decisions
- JetPack 6.2.x (Ubuntu 22.04) as the testbed OS; exact version recorded for reproducibility.
- Conffile prompts during upgrade: kept `nv-oem-config-post.sh` deleted (removed by design after
  first boot); accepted NVIDIA's version of the APT source list.
- Least privilege on the IDS host: SSH reachable only from the USB management link, key-only.
- **Open methodological decision:** the isolated lab has no NTP source and the board has no RTC
  battery. A local `chrony` server is proposed so that all lab hosts share one time reference;
  otherwise time-to-alert and pcap/alert correlation are unreliable.

### AI use
Claude (Anthropic) was used to plan the installation path, interpret command outputs and diagnose
the network faults. Recommendations were checked against NVIDIA's JetPack 6.2.2 and Jetson Linux
36.5 documentation and verified on the device; all results above are from real command output.
One AI suggestion (an overly narrow `ufw` rule) caused the DHCP failure and was corrected.
Transcript: `AI-evidence/2026-11-16_claude-jetson-setup-transcript.pdf`.

### Next step
- Disable Wi-Fi before moving the Jetson into the isolated lab (CIS Ubuntu 22.04, 3.1.2).
- Decide and fix the power mode for performance measurements.
- Complete and submit the ethics form to the supervisor (deadline 4 Dec).
- Build the `br0` inline bridge once the USB Ethernet adapter arrives.
