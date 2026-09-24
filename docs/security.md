# Security and privacy boundaries

## Publish the lab, not the private application

The public package must contain deliberately selected, reviewed source and synthetic
examples. Do not copy a working operational directory wholesale and then rely on
`.gitignore` to make it safe.

Never publish:

- Private application source, fixtures or customer/candidate data.
- Tokens, `.env` values, private SSH keys or runner registration files.
- VM baseline disks, overlays, snapshots, seed images or backups.
- Generated cloud-init `user-data`/`meta-data`, inventory or SSH known-host files.
- Private repository/run links, machine identities, real IP addresses or raw job logs.

A baseline containing an enrolled runner is **credential-bearing**, even when its
filesystem permissions are restrictive. It must not become a downloadable demo image.
Build a new public example from source and require the owner to register their own
runner. GitHub coordination necessarily involves remote repository data and logs;
"tests execute on your hardware" does not mean "nothing leaves your laptop".

The included `.gitignore` is only a backstop. It neither removes tracked files nor
scrubs Git history. Before publication, review the complete tracked-file list and
history, run a secret check, inspect any screenshots/artifacts, and obtain approval.
No licence has been chosen for this draft yet.

## Trusted code only

Do not run arbitrary pull requests, forks or other people's code on this runner.
Start with manual workflows, reviewed action pins and explicit trusted revisions.
A repository's runner labels select a machine; they do not establish trust.

A job can have administrative power **inside the guest**. Discarding its disk later
does not prevent malicious code from stealing guest-accessible credentials or making
network requests during that job. VM isolation reduces exposure; it is not a proof
against hypervisor bugs or an audited hostile-code sandbox.

The reference design deliberately excludes:

- Host filesystem and device mounts, personal home and SSH-agent forwarding.
- The host Docker socket.
- Production credentials and unbounded paid-service access.
- Automatic redispatch after uncertain execution.

## Network and identity limits

The reference blocks guest access to host/LAN/private/metadata destinations, with
explicit management/DNS exceptions, and blocks known production addresses. Public
HTTP(S) remains available for GitHub and dependency downloads. This is **not** a
comprehensive internet allowlist or a guarantee that every production endpoint is
unreachable. Review overlapping subnets, IPv6 and firewall integration on each host.

The runner uses a persistent repository-scoped identity with **disposable disks**.
It is not GitHub's `--ephemeral` registration model. The tested reference uses
`--once`, which carries an upstream deprecation warning. Revalidate support and
plan a supported replacement before presenting it as a long-lived turnkey product;
do not add a silent persistent-runner fallback.

Owner/root administrators are trusted. Protect and rotate the runner's own
credentials through supported mechanisms; do not place a long-lived administrative
GitHub token on the host simply to recreate the guest.

## Lifecycle status is not release authority

The controller's state tracks whether it is safe to manage a disk. It does not prove
that application assertions passed and is not security attestation against a
malicious guest. GitHub reports the test verdict; the owner decides whether to release.

After uncertain execution, preserve state and investigate without automatically
replaying paid or state-changing operations. Review any evidence privately and
publish only deliberately sanitized examples.
