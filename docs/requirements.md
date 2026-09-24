# Requirements and tools

These requirements describe the private reference design. They are not an installer
or a promise that every laptop meeting the numbers can test every application.

## Hardware and capacity

The reference guest has **4 vCPUs, 8 GiB RAM and an 80-GiB thin-provisioned disk**.
Only one job runs at a time.

| Resource | Practical starting requirement | Why |
| --- | --- | --- |
| CPU | x86-64 Intel/AMD, 4 logical CPUs, hardware virtualization enabled in firmware | KVM runs the guest; the host still needs CPU time. Prefer 8+ logical CPUs. |
| Memory | 16 GB installed; at least **10 GiB `MemAvailable` at each guest launch** | The controller enforces available memory, not the number printed on the laptop. The guest consumes 8 GiB. |
| Storage | SSD; plan for **120 GiB free** on the filesystem holding the VM | Allows for the guest disk's possible growth, baseline, provisioning files and headroom. Prefer 150 GiB+. |
| Power | AC connected whenever a new guest starts | The reference controller refuses a battery-only launch. |
| Networking | Stable internet and working DNS | GitHub coordination, images and dependencies are remote. No public inbound port forwarding is required. |
| Cooling | Working fan, clear ventilation and a sound battery/charger | Sustained tests load the CPU; a damaged or overheating laptop is not suitable. |

**Memory:** a 16-GB laptop with a busy desktop may fail the 10-GiB availability
check. Close other workloads or use a lighter dedicated host. Do not blindly lower
the guard to force an 8-GB laptop through. A smaller VM profile would need its own
capacity tests and limits.

**Storage:** the reference controller checks for at least **20 GiB free** before
launch and periodically during a job. That is an emergency guard, not a sensible
installation requirement or an atomic reservation. The 80-GiB virtual disk does
not reserve 80 GiB on the host immediately. Concurrent host workloads can still
exhaust space; retained failure disks also accumulate. Check the actual VM-storage
filesystem, not just total advertised SSD capacity. A "128-GB SSD" is not equivalent
to 120 GiB free after installing the OS.

**Timing:** the reference configuration allows 30 minutes for the QA workflow,
45 minutes for the controller's job deadline and 5 minutes for guest boot. These
are timeout policies, not measured capacity guarantees for old hardware.

## Operating systems

- Reference host: Ubuntu Linux with KVM, libvirt, systemd, nftables and AppArmor.
- Reference guest: official Ubuntu 24.04 LTS (Noble), x86-64 cloud image.
- GitHub helper actions: verified Node24-compatible versions; use a runner version
  meeting their documented minimum. The tested reference runner was 2.337.0.
- Windows/macOS hosts, ARM machines and other distributions are not validated here.

A laptop already running Windows may be repurposed with Linux, but changing the OS
is a separate owner decision. Back up existing data first. This draft does not
format disks, recommend erasing another OS or claim Hyper-V/WSL compatibility.

## Required host tools

| Tool | Purpose |
| --- | --- |
| QEMU/KVM | Hardware-accelerated virtual machine execution |
| libvirt daemon and `virsh` | VM/network definitions, power state and lifecycle control |
| `virt-install` | Initial guest definition and provisioning |
| `qemu-img` | Baseline and per-job qcow2 disk operations |
| `genisoimage` | Non-secret cloud-init seed image in the reference design |
| `nft` / nftables | Dedicated guest network restrictions |
| systemd, `systemctl`, `journalctl`, `systemd-inhibit` | Services, logs and sleep/lid inhibition while active |
| Python 3 | Controller and provisioning helpers |
| OpenSSH | Owner-controlled administration; use host-key verification |
| `curl`, CA certificates, `gpgv`, official Ubuntu signing keys, `sha256sum`, `tar` | Download verification and trusted image/runner preparation |

Package names vary by distribution. Typical Ubuntu virtualization packages include
`qemu-kvm`, `qemu-utils`, `libvirt-daemon-system`, `libvirt-clients` and `virtinst`.
Installing virtualization/network packages can create services and networks: review
that on a machine with existing VMs. Do not paste an unreviewed root install script
or replace an existing firewall to make a demo work.

The host does **not** need to run the application's Docker containers. Docker and
application dependencies belong inside the dedicated guest.

## Required guest tools

- Official Ubuntu cloud image with cloud-init for initial setup.
- QEMU guest agent for the host's management channel.
- Docker for disposable databases and other test services.
- Official GitHub Actions runner, Git and its documented native dependencies.
- The application's runtime, package manager and test dependencies.
- Optional browser binaries and their OS dependencies for actual browser tests.

The GitHub runner supplies the runtime used for JavaScript actions. A Node24
checkout/setup action and an application's Node/Python runtime are separate things.
Keep action commits pinned **and** check their declared runtime; pinning an old
commit alone does not keep it supported.

For Linux Python packages from `actions/setup-python`, the reference uses a
runner-writable `/opt/hostedtoolcache` through `AGENT_TOOLSDIRECTORY`. Its clean
subprocess check catches missing shared-library resolution before the long suite
starts. See [the verified failure](evidence.md).

## Application tools: choose, do not install everything

| Check | Example tools | Reference status |
| --- | --- | --- |
| Unit/library tests | pytest, or your language's test runner | Python library suite passed |
| Web/database tests | Django test runner with temporary PostgreSQL and Redis | Web suite and migrations passed |
| Browser journeys | Playwright plus Chromium | Browser installed; complete user-journey coverage not established |
| Known dependency vulnerabilities | pip-audit; use an appropriate equivalent for other ecosystems | Python audit passed |
| Secret detection | A separately reviewed, pinned secret-scanning tool | Separate manual entrypoint exists; not part of the successful QA run |
| Local service orchestration | Docker Compose, if your application uses it | Optional; GitHub service containers did not require Compose |
| Load testing | A bounded tool such as k6 against an isolated target | Not implemented/validated by this reference |

External services should be stubbed or use explicit, bounded test accounts. Never
point test migrations, browser flows or load tests at production by default. Paid
scanners and AI are optional, not prerequisites.

## Accounts and administration

You need a GitHub repository, runner-registration permission and administrator
access to your own laptop. Use a dedicated label and trusted refs. Repository labels
route jobs; they are not an access-control system.

The GitHub web UI is enough to dispatch a manual workflow. GitHub CLI (`gh`) is an
optional administration tool, not a reason to install a long-lived administrative
token on the laptop. Keep credentials out of cloud-init, command history, screenshots
and source control. The sealed runner baseline itself is credential-bearing and private.

## Read-only checks before installation

These inspect local resources; they do not install tools or change permissions:

```sh
uname -m
getconf _NPROCESSORS_ONLN
grep -E 'MemTotal:|MemAvailable:' /proc/meminfo
ls -l /dev/kvm
```

After choosing an existing VM-storage directory and installing libvirt, also check:

```sh
df -h /var/lib/libvirt/images
virsh --connect qemu:///system list --all
```

Use your actual storage location. A permission error accessing KVM/libvirt may mean
an account or service configuration problem, not an incompatible CPU. Review the
appropriate group/service permissions; do not make `/dev/kvm` world-writable.

Boot-time Wi-Fi, guest startup and sleep inhibition must be checked on the actual
laptop. The reference's services were exercised, but its **physical host reboot
check is still pending**.

## Upstream documentation

- [Ubuntu virtualization](https://documentation.ubuntu.com/server/how-to/virtualisation/)
- [libvirt documentation](https://libvirt.org/docs.html)
- [GitHub self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [actions/setup-python](https://github.com/actions/setup-python)
- [Playwright](https://playwright.dev/python/)
- [pip-audit](https://github.com/pypa/pip-audit)
