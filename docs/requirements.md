# Requirements and tools

Start by checking the laptop, then separate the tools that belong on the host from
those that belong inside the test VM. This guide covers the prototype's single-job
profile: **4 vCPUs, 8 GiB RAM and an 80-GiB virtual disk**.

## 1. Check the hardware

| Resource | Planning baseline | Recommended headroom |
| --- | --- | --- |
| CPU | x86-64 Intel/AMD, 4 logical CPUs, VT-x/AMD-V enabled in firmware | 4+ physical cores or 8+ logical CPUs |
| RAM | 16 GB installed, with **10 GiB available at VM launch** | 24–32 GB |
| Disk | SSD with **120 GiB free** where VM images will live | 150 GiB+ free |
| Power | AC connected | Reliable charger, healthy battery and clear ventilation |
| Network | Stable internet and DNS | Ethernet, or Wi-Fi that connects before desktop login |

These are sizing estimates for the profile, not benchmarked minimums. The tested
host had 16 logical CPUs and roughly 30 GiB usable RAM. Larger applications can need
more memory, disk or execution time.

### Memory: available is different from installed

The controller checks Linux's `MemAvailable` and requires at least 10 GiB before
starting the 8-GiB guest. A 16-GB laptop with many desktop applications open may not
have enough left. A dedicated or lightly loaded host is a better fit. An 8-GB host
would need a smaller, separately tested guest configuration.

### Disk: budget for growth

A thin disk starts small but grows as the job installs packages and writes data.
The host needs room for that growth, the baseline image and any retained failure
disks. Plan around **free space**, not the SSD's advertised capacity.

The controller's 20-GiB free-space threshold is a runtime stop condition. It is not
the installation requirement and does not reserve space against other host workloads.
A 128-GB SSD will generally not provide the 120 GiB free required by this estimate
once an operating system is installed.

### Time limits

The prototype allows 5 minutes for guest boot, 30 minutes for the QA workflow and
45 minutes for the controller's job deadline. A slower laptop or larger suite may
need a different, measured configuration.

## 2. Use a compatible host

The tested setup uses Ubuntu Linux with KVM, libvirt, systemd, nftables and AppArmor.
The guest is the official x86-64 Ubuntu 24.04 LTS cloud image. Windows/macOS hosts,
ARM devices and other distributions have not been validated.

Repurposing a Windows laptop with Linux is an option, but back up existing data
before changing its operating system. There is no WSL or Hyper-V setup supplied here.

## 3. Host tools

The host manages the VM; it does not run the application's service containers.

| Tool | Job |
| --- | --- |
| QEMU/KVM | Run a hardware-accelerated VM |
| libvirt and `virsh` | Manage domains, networks and guest power state |
| `virt-install` | Create the initial guest definition |
| `qemu-img` | Create and inspect qcow2 baselines and writable overlays |
| `genisoimage` | Build the cloud-init seed image |
| nftables (`nft`) | Apply guest network restrictions |
| systemd, `journalctl`, `systemd-inhibit` | Run services, inspect logs and prevent sleep during operation |
| Python 3 | Run the controller and provisioning helpers |
| OpenSSH | Administer the machine and guest using verified host keys |
| `curl`, CA certificates, `gpgv`, Ubuntu signing keys, `sha256sum`, `tar` | Download, verify and unpack images and runner releases |

Ubuntu's virtualization packages commonly include `qemu-kvm`, `qemu-utils`,
`libvirt-daemon-system`, `libvirt-clients` and `virtinst`. Package installation can
start services or create networks, so check existing VMs and firewall rules before
changing a machine that already hosts other workloads.

## 4. Guest tools

Inside the VM, the prototype uses:

- **cloud-init** for initial configuration.
- **QEMU guest agent** for the host management channel.
- **Docker** for temporary databases and service containers.
- **Git and the official GitHub Actions runner** to receive and execute a job.
- **The application's runtime and dependencies**, installed by its workflow.

The tested Actions runner was version 2.337.0. Action versions must be compatible
with the installed runner. The prototype pins Node24-compatible checkout and Python
setup actions; their JavaScript runtime is provided by the runner, independently
of the language used by the application.

Python is installed under `/opt/hostedtoolcache`, selected through
`AGENT_TOOLSDIRECTORY`. This location lets the tested shared Python build start even
when a child process clears its environment. The [case study](evidence.md#python-could-not-start-in-a-clean-environment)
explains the failure that led to this choice.

## 5. Application test tools

Pick tools that match your stack rather than installing every tool in this table.

| Purpose | Examples |
| --- | --- |
| Unit tests | pytest or the language's native test framework |
| Web/database integration tests | Django's test runner with PostgreSQL and Redis |
| Browser journeys | Playwright and Chromium |
| Known dependency vulnerabilities | pip-audit for Python; an equivalent scanner for other ecosystems |
| Secret detection | A dedicated secret scanner in a separate check |
| Local service orchestration | Docker Compose, if the application uses it |
| Load testing | k6 or another load generator against an isolated target |
| Adaptive browser exploration (planned) | Jev through TypeSafe, with Playwright executing permitted actions |

The completed prototype run exercised the Python library/web suites and pip-audit.
Chromium was installed, but full browser-journey coverage and load testing are still
open work. GitHub service containers did not require Docker Compose. No paid scanner
or AI service is required for the basic setup.

### Additional requirements for the planned Jev layer

Jev-guided exploration would add a TypeSafe API client, authenticated direct service
access, an explicit inference budget and a Playwright adapter with permitted actions
and fixtures. Credentials would be supplied only to an enabled test run, not stored
in the guest baseline. An unavailable service would stop the Jev stage rather than
switch to another provider.

Jev inference runs through TypeSafe's service, so this design adds no local GPU
requirement. Page summaries sent for evaluation would contain only minimized
synthetic test data. The adapter and its cost/coverage measurements are still to be
implemented; see [Jev-guided QA](jev-qa.md).

## 6. Accounts and access

You need administrator access to the laptop and permission to register a runner
with your GitHub repository. The GitHub web interface is enough to start manual
jobs; GitHub CLI (`gh`) is useful but optional.

Runner labels route work to the right machine. Repository permissions and reviewed
workflow/code revisions establish who can use it. Credentials and network boundaries
are covered in the [security guide](security.md).

## Read-only readiness checks

These commands inspect the machine without changing it:

```sh
uname -m
getconf _NPROCESSORS_ONLN
grep -E 'MemTotal:|MemAvailable:' /proc/meminfo
ls -l /dev/kvm
```

Expect an x86-64 architecture, enough CPUs and memory, and an available KVM device.
Device presence alone does not confirm that your account or libvirt service can use it.

Once libvirt is installed and its storage directory exists:

```sh
df -h /var/lib/libvirt/images
virsh --connect qemu:///system list --all
```

Substitute your chosen image-storage directory if different. A permission error may
require correcting group or service access; making `/dev/kvm` world-writable is not
a suitable fix.

Finally, verify network connection, guest startup and sleep behaviour on the actual
laptop. Service stop/start worked in the prototype; a physical host reboot remains
on the validation checklist.

## Further reading

- [Ubuntu virtualization](https://documentation.ubuntu.com/server/how-to/virtualisation/)
- [libvirt documentation](https://libvirt.org/docs.html)
- [GitHub self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [actions/setup-python](https://github.com/actions/setup-python)
- [Playwright](https://playwright.dev/python/)
- [pip-audit](https://github.com/pypa/pip-audit)
