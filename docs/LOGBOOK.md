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
