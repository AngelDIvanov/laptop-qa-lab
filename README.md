# Laptop QA Lab

**Turn a spare laptop into a repeatable QA environment before paying for production hosting.**

Build your product locally. Check its behaviour in a clean environment. Fix problems
while they are cheap to reproduce. Then make an informed decision about cloud
hosting, a domain name and a public launch.

This SRE portfolio project documents a laptop-based GitHub Actions setup with a
separate KVM virtual machine and a fresh writable disk for each job. It demonstrates
reproducible infrastructure, failure handling, resource limits and operational
trade-offs—not a claim that automation can find every bug.

> **Status: public documentation preview—not an installable release.** The private
> reference implementation has completed a real CI run and disposable-VM cleanup.
> This repository does not yet include a general-purpose installer, runner
> implementation or runnable demo app. Those need independent packaging and testing.
> No workflows are shipped; GitHub Actions is disabled for this documentation preview.

## Why start locally?

You do not need a public domain or a rented cloud server to check most application
logic, database behaviour and browser journeys. A spare machine can run those
checks against synthetic data without exposing the product to the public internet.

A repeatable local QA process helps you:

- Catch regressions before putting a release in front of users.
- Test installation and migrations outside your development environment.
- Exercise critical user journeys and permission boundaries, if you write those tests.
- Check dependencies for known vulnerabilities.
- Learn what resources the test workload needs before choosing infrastructure.
- Collect useful failure evidence without treating a successful cleanup as a passed test.

This can defer spending on hosted compute. It does **not** eliminate electricity,
internet, storage or external-service costs. GitHub-hosted runner minutes are not
used by jobs executed on your own machine; other GitHub billing limits still apply.

**Local execution is not offline operation:** GitHub coordinates jobs and stores
results. The machine needs internet access for GitHub and approved dependencies.

## How the reference design works

1. Develop the application locally and commit a trusted revision to your GitHub repository.
2. Power on the QA laptop, connect AC power and wait for its runner to be Online.
3. Manually start the QA workflow. Select the workflow branch and the code revision separately.
4. A dedicated VM checks out that code and creates temporary services and test data.
5. The workflow runs the checks you configured; GitHub displays the results.
6. After confirmed job completion and guest shutdown, the controller discards that
   writable disk and creates a fresh waiting VM.

A completed job may contain **failed tests**. That result remains failed in GitHub,
even though the VM can be cleaned up normally. An interrupted or uncertain lifecycle
preserves the disk and stops for operator review; it does not automatically replay
the job. When the laptop is off, jobs wait instead of using a hosted-runner fallback.

This design runs one job at a time. It is not a highly available runner fleet.

## What laptop do I need?

For the reference **4-vCPU / 8-GiB guest / 80-GiB thin-disk** profile:

| Component | Practical starting point | Preferable |
| --- | --- | --- |
| CPU | x86-64 Intel/AMD, 4 logical CPUs, VT-x/AMD-V enabled | 4+ physical cores; 8+ logical CPUs |
| RAM | 16 GB installed, **at least 10 GiB `MemAvailable` before VM launch** | 24–32 GB, with little else running |
| Disk | SSD with **120 GiB free** on the VM storage filesystem | 150 GiB+ free; room for retained failure disks |
| OS | Ubuntu Linux host with working KVM/libvirt; Ubuntu 24.04 LTS guest | A maintained, patched installation |
| Power/network | AC power, sound cooling, stable internet | Wired Ethernet; Wi-Fi with system-wide autoconnect |

These are **planning requirements, not a measured minimum-hardware benchmark**.
The reference run used a substantially larger, roughly 30-GiB-usable-RAM laptop.
An 8-GB machine is not suitable for this guest profile. The application may need more
resources; passing a hardware checklist does not guarantee it fits the job timeout.

See [Requirements and tools](docs/requirements.md) for the enforced limits,
installation prerequisites and checks. Windows hosts, macOS, ARM and smaller
profiles are not validated by this reference.

## Tools involved

- **Host:** QEMU/KVM, libvirt, virt-install, qemu-img, nftables, systemd, Python 3,
  SSH and image/download verification tools.
- **Guest:** Ubuntu cloud image, cloud-init, QEMU guest agent, Docker, Git and the
  official GitHub Actions runner. The host's Docker socket is never shared.
- **Application QA:** the project's language/runtime and test runner; optional
  Playwright/Chromium browser journeys and ecosystem-specific dependency checks.
- **Administration:** a GitHub repository and permission to register its runner;
  the GitHub UI is sufficient for dispatch. GitHub CLI is optional.

Node24 runs the GitHub helper actions in the reference workflow. It is not a
requirement to rewrite a Python application in Node.js or install Node on the host.
No AI subscription or paid security scanner is required.

## Evidence and boundaries

The private reference run passed **278 library tests, 1,118 web tests and a Python
dependency audit**. Actual job hooks, disk disposal and a fresh guest without the
previous workspace were also checked. Only aggregate, non-sensitive results are
included here; private application code, run links and machine identities are not.

Read [Evidence and limitations](docs/evidence.md). The counts are reference results,
not tests you can currently reproduce from this documentation-only draft.

A clean VM improves repeatability. It does not create missing test coverage, prove
security or guarantee production performance. Real browser coverage must contain
assertions about your product—not just install a browser or click around.

**Trusted code only.** This is not an audited sandbox for arbitrary public pull
requests. Read [Security and privacy](docs/security.md) before attaching a runner
to any repository.

## From laptop to online product

A useful progression is:

**Build locally → repeatable QA → fix failures → plan production → stage → launch.**

Once the product works locally and its important checks pass, consider hosting,
a domain, TLS, backups, observability and deployment automation. Buying these does
not make the product ready; local tests also cannot replace testing those real
production integrations. Purchasing a domain early is optional, not a QA prerequisite.

[Local-first QA guide](docs/local-first-qa.md) describes that progression and the
checks that still need a staging or production-like environment.

## Design trade-offs

- **One job at a time:** predictable resource use rather than a highly available runner fleet.
- **Fresh environments:** less state leakage, at the cost of downloading and installing dependencies again.
- **Stop on uncertainty:** interrupted jobs need operator review rather than an automatic retry.
- **Trusted workloads:** VM isolation limits exposure but is not a hostile-code security guarantee.
- **Explicit evidence:** CI results describe the checks that ran, not complete product or release readiness.

### Roadmap

- [x] Document requirements, reference results and security boundaries.
- [ ] Package configurable runner/provisioning code with lifecycle tests.
- [ ] Add a synthetic demo app, manual workflow and asserted browser journeys.
- [ ] Add an architecture diagram and a clean-machine operator runbook.
- [ ] Validate installation on a second machine, physical reboot and interrupted-job recovery.
- [ ] Select a licence for reuse.

## Licence

No licence grant is currently included. This is a documentation preview, not an
open-source software release.
