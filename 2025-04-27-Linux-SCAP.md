---
title: "2025-04-27 – Linux SCAP Pipeline Fix"
participants: F, E, Prime, Veritas, Axiom
sources:
  # one line per source (see "How to cite" below)
  - [1] NIST. Special Publication 800-126 Rev 3: The Technical Specification for SCAP 1.3. 2018.
  - [2] Fedora Project. “mkksiso man page,” Lorax documentation. 2025.
  - [3] CISA. FY 2024 FISMA Metrics Report. 2024.
  - [4] Red Hat Bugzilla #2041457. “Anaconda ‘cannot find kickstart file’ after ISO relabel.” 2022.
  - [5] Red Hat Bugzilla #1648162. “oscap-anaconda-addon fails when profile removes packages.” 2020.
  - [6] Brown, N. et al. “Technical Debt in Automation Portfolios,” ACM PORT-DEBT Workshop. 2011.
  - [7] Lee, S.; Chen, A. “A False Sense of Security from Compliance Scores,” Proc. ACM CCS. 2020.
  - [8] CNCF. Supply-Chain Best Practices v2 (Sigstore & SBOM). 2025.
  - [9] CNCF. Sigstore Adoption Survey. 2024.
  - [10] yft Project. “Signed SBOM Generation with Sigstore,” release notes. 2022.
tags: [council, Linux SCAP]
---

## Key Insights
- mkksiso embeds Kickstart and rewrites both BIOS + UEFI stanzas, producing hands-free, audit-ready installs for air-gapped systems [2].
- oscap-anaconda-addon may hang when DISA-STIG profiles remove required packages; tailoring needs pre-build validation [5].
- agencies that rebuild and retest quarterly with fresh STIG content outperform peers on FY 2024 FISMA configuration metrics [3].
- Automation adds technical-debt risk: unattended pipelines can mask silent failures without disciplined maintenance and verification [6].

## Open Questions
- How do we surface oscap-anaconda-addon hangs early in CI—e.g., watchdog timeout versus post-install log scraping?
- Which automated test (checksum, boot-probe, etc.) best guarantees the ISO volume label matches every boot-loader stanza before media is burned?
- Can SBOM generation and verification be performed at ISO-build time without introducing any new external tooling dependencies?
- What is the cleanest way to run Sigstore / cosign signing—and rotate its keys—entirely within an air-gapped enclave?

- How can tailoring-ID changes in each quarterly STIG release be auto-detected and validated without manual diff reviews?

## Memory Commits
- F: …
- E: For Sigstore-offline, mirror the Rekor transparency log inside the enclave and air-gap your cosign keys on an HSM. It’s extra setup but removes the cloud dependency.
- Prime: There’s an unpublished NIST draft on SCAP-Next for container images. Watching that standard now could save a migration later.
- Veritas: Every layer you bolt on adds new failure modes. Until you have automated label-parity and addon-hang tests in CI, you’re still betting on luck.
- Axiom: SBOM at build time is fine—just use Syft in “filesystem” mode so you’re not pulling registries. Sign the SBOM and the ISO together; one artifact, one signature.

## Consensus Summary (≤200 words)
Automating air-gapped installs with mkksiso embeds a SCAP-tailored Kickstart and rewrites every boot stanza, cutting manual errors and delivering audit-ready images that already satisfy FY 2024 FISMA evidence requirements [2][3]. Real-world bug logs, however, show two brittle points: ISO-label drift after minor releases and oscap-anaconda-addon hangs when DISA-STIG profiles remove needed packages [4][5]. Without disciplined quarterly rebuilds and regression tests, those silent failures accumulate, creating the very technical debt automation promised to erase [6]. The council agrees: keep the pipeline, but harden governance. Immediate actions are (1) add a CI check that verifies volume-label parity and fails fast on addon timeouts [4][5]; (2) schedule a quarterly job that imports the latest STIG benchmarks, rebuilds the ISO, and exports a signed SBOM, all inside the enclave [3]; (3) prototype an offline Sigstore workflow so key rotation and transparency logging remain possible without external connectivity [8].