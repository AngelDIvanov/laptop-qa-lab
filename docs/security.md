# Security model

The prototype is a single-owner QA worker for reviewed application code. Its main
boundaries are the host, the disposable guest and GitHub. This page explains what
those boundaries protect and where trust is still required.

## Who can run code?

The laptop owner controls the repository, workflow and revision selected for a
manual run. Runner labels direct the job to a machine; they do not authorize the
person or code using it.

The design is not intended for arbitrary public pull requests or untrusted forks.
Host administrators are trusted, and a job can have administrator privileges inside
the guest to install dependencies. A malicious job could read guest-accessible
credentials or send network requests before cleanup. This is not an audited
hostile-code sandbox.

## What does the VM isolate?

Application code, Docker, databases and browsers run inside the guest. The prototype
does not mount the host's directories or physical devices, share its Docker socket,
or forward its SSH agent. Production credentials are not supplied to the guest.

The baseline starts read-only; each job writes to a separate overlay. Replacing the
overlay removes ordinary files and configuration left by the previous job. It does
not undo external side effects or eliminate hypervisor vulnerabilities. The host
still needs patching and appropriate access controls.

## Where are credentials and data stored?

The runner registration belongs to one repository and persists across guest resets.
An enrolled baseline, its overlays and backups may contain registration credentials.
Treat them as secret-bearing storage rather than distributable VM images. Each new
installation needs its own registration.

The prototype does not need a long-lived administrative GitHub token on the host
to recreate a guest. Test applications use synthetic records and disposable services;
secrets should stay out of fixtures, logs and exported artifacts.

GitHub stores the repository and job results. Running tests on your laptop therefore
does not make the system air-gapped or keep every piece of data off remote services.

## What network access remains?

The prototype restricts access from the guest to the host, LAN, private address
ranges and cloud metadata endpoints, with explicit management and DNS exceptions.
Selected production destinations are also blocked. Public HTTP(S) is available for
GitHub and dependency downloads.

This is not a comprehensive outbound allowlist. A public production endpoint could
still be reachable; withholding its credentials and using test-specific configuration
remain important. Subnet overlap, IPv6 and existing firewall rules must be considered
when applying the design to another machine.

## What happens after a failure?

| Outcome | Disk handling |
| --- | --- |
| Tests pass and the job completes normally | Discard the overlay after guest shutdown |
| Tests fail but the job completes normally | Keep the failed result in GitHub; discard the overlay after shutdown |
| Execution is interrupted or completion is uncertain | Retain the disk and stop for investigation; no automatic redispatch |

Retained disks and diagnostic output can contain sensitive data. Controller state
is lifecycle telemetry, not security attestation against a malicious guest or proof
that application tests passed. Deployment remains a separate operation.

## Runner lifecycle dependency

Disposable disks are different from GitHub's `--ephemeral` registration model. The
prototype keeps its repository-scoped registration and starts the listener with
`--once`. That option carries an upstream deprecation warning and needs a supported
replacement before a runnable release. Keeping one guest alive for multiple jobs
would change the cleanup model, rather than being an equivalent fallback.
