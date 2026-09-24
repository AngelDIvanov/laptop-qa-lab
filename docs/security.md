# Security model

This page describes the reference design's trust assumptions and limits. The public
repository currently contains documentation, not an installer that applies these
controls to your machine.

## Trust assumptions

The runner is intended for code and workflows reviewed by the laptop owner. It is
not a service for executing arbitrary public pull requests or untrusted forks.
Manual dispatch, explicit code revisions and reviewed action pins limit what runs;
runner labels only route jobs and are not an authorization boundary.

Host administrators are trusted. Jobs can have administrative privileges inside
the guest to install dependencies and run service containers. A malicious job could
therefore access guest credentials or make network requests before its disk is
discarded. This design is not an audited hostile-code sandbox.

## Host and guest isolation

Application code, Docker, test databases and browsers run inside a dedicated VM.
The reference does not share the host's:

- Filesystems, personal home or physical devices.
- Docker socket or SSH agent.
- Production credentials.

A fresh writable disk prevents ordinary job state from carrying into the next run.
A protected baseline supplies the starting environment. These controls reduce
exposure and improve repeatability; they do not eliminate hypervisor vulnerabilities
or replace host patching and access control.

## Credentials and test data

Use synthetic accounts and disposable test data rather than production records.
Keep application secrets out of source control, test output and exported artifacts.

The runner has a persistent, repository-scoped identity. An enrolled baseline and
its derived disks can contain registration credentials, so they must be protected
like other secrets—not distributed as reusable VM images. Each installation needs
its own registration. Recreating the guest does not require storing a long-lived
administrative GitHub token on the host in this reference design.

Test execution is local, but coordination and job logs are hosted by GitHub. This
is not an air-gapped system or a guarantee that application data stays on the laptop.

## Network boundaries

The reference restricts guest access to the host, LAN, private address ranges and
cloud metadata endpoints, with explicit management and DNS exceptions. It also
blocks selected production addresses. Public HTTP(S) remains available for GitHub
and dependency downloads.

This is **not a comprehensive outbound allowlist**. A reachable public endpoint may
still receive requests from a job. Subnet overlap, IPv6 and interaction with an
existing host firewall need review for each installation. Production isolation also
depends on withholding credentials and using safe test configuration.

## Failure handling

After confirmed job completion and guest shutdown, the writable disk is discarded.
A failed test can still complete this lifecycle normally; its failed result remains
in GitHub.

Interrupted or uncertain execution retains the disk and stops for operator review
instead of automatically redispatching the job. This avoids blindly repeating
state-changing operations. Retained disks and diagnostic output can contain sensitive
data and need the same protection as the active environment.

Controller status is lifecycle telemetry, not proof that tests passed or security
attestation against a malicious guest. A green CI result is also not automatic
approval to deploy.

## Known runner limitation

Disposable disks are separate from GitHub's `--ephemeral` registration model. The
reference reuses a repository-scoped registration and runs the listener with
`--once`. That option carries an upstream deprecation warning; a supported replacement
needs evaluation before a runnable release. A persistent multi-job guest is not an
equivalent fallback because it would change the isolation and cleanup model.
