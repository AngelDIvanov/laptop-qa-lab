# Laptop QA Lab

**Put a spare laptop to work testing your product before you pay for production hosting.**

Laptop QA Lab documents a GitHub Actions setup that runs tests inside a disposable
virtual machine on your own hardware. Each job starts with a clean environment,
uses temporary databases, and leaves its results in GitHub.

The aim is simple: build locally, get reliable feedback, and fix the basics before
adding a public domain and a production server.

> **Documentation preview.** These guides describe a working prototype. The runner
> code, installer and demo application are not yet included. No workflows are shipped,
> and GitHub Actions is disabled for this repository.

## Why use a spare laptop?

A local development environment can hide missing dependencies, old database state
and manual setup steps. Running the same checks in a fresh VM helps catch those
problems before someone else tries to use the application.

Reusing a laptop also lets you start without renting compute for your test jobs.
Electricity, internet, GitHub storage and any external services still have their own
costs. GitHub coordinates the work and stores logs, so this is local execution—not
an offline CI system.

The runner executes tests you supply. It does not generate complete test coverage
or decide whether your product is ready to launch.

## How a job runs

1. Commit a reviewed version of your application to your GitHub repository.
2. Connect the QA laptop to power and wait for its runner to show **Online**.
3. Start a manual workflow and select the code revision to test.
4. The VM checks out the code, prepares temporary services and runs your checks.
5. GitHub reports the outcome. After confirmed completion and shutdown, the used
   VM disk is discarded and a fresh guest waits for the next job.

If the laptop is off, the job queues rather than switching to hosted compute.
An interrupted or uncertain run stops for investigation instead of being replayed
automatically. One job runs at a time.

## Is my laptop suitable?

The prototype gives its guest **4 vCPUs, 8 GiB RAM and an 80-GiB thin disk**.
For that profile, use these planning requirements:

| Component | Starting point |
| --- | --- |
| Processor | x86-64 Intel/AMD, 4 logical CPUs, VT-x/AMD-V enabled |
| Memory | 16 GB installed; at least **10 GiB available before the VM starts** |
| Storage | SSD with **120 GiB free** on the VM-storage filesystem |
| System | Ubuntu Linux with KVM/libvirt; Ubuntu 24.04 LTS guest |
| Power and network | AC power, working cooling and stable internet |

A 24–32-GB machine with more CPU cores and 150 GiB+ free SSD space gives useful
headroom. These are sizing estimates, not a tested minimum-hardware certification:
the validation machine had roughly 30 GiB usable RAM. An 8-GB laptop cannot accommodate
this guest profile comfortably. Windows/macOS hosts and ARM hardware are untested.

## Read the guides

| Guide | What you will find |
| --- | --- |
| [Requirements and tools](docs/requirements.md) | Hardware checks, host software, guest dependencies and application test tools |
| [Local-first QA](docs/local-first-qa.md) | A practical path from a local app to repeatable tests, staging and launch |
| [Validation case study](docs/evidence.md) | Test results, failures encountered and the fixes that resolved them |
| [Security model](docs/security.md) | Trust assumptions, VM boundaries, credentials and network access |

## What has been tested?

One Python/Django application completed **278 library tests, 1,118 web tests and a
Python dependency audit**. The job hooks and disk cleanup also worked: the next
guest had no previous application workspace.

That application is not included here. The [case study](docs/evidence.md) separates
observed results from work still to be validated, including physical-host reboot
and installation on a second machine.

## Design choices

- **Disposable jobs:** fewer dependencies on leftovers, at the cost of reinstalling tools.
- **Manual execution:** the owner chooses when and which code to test.
- **Stop on uncertainty:** recovery favours investigation over repeating side effects.
- **Trusted code:** the VM is not an audited sandbox for arbitrary public pull requests.
- **Separate deployment:** test results inform a release; the runner does not deploy it.

Once local checks are dependable, the next step is to choose production hosting,
configure a domain and test the real infrastructure in staging. DNS, TLS, backups,
monitoring and third-party integrations need checks of their own.

## Roadmap

- [x] Document requirements, validation results and the security model.
- [ ] Package configurable runner/provisioning code with lifecycle tests.
- [ ] Add a demo application, manual workflow and browser journeys with assertions.
- [ ] Add an architecture diagram and a clean-machine setup/runbook.
- [ ] Validate a second machine, physical reboot and interrupted-job recovery.
- [ ] Select a licence for reuse.

## Licence

No licence grant is currently included.
