# Validation case study

On **25-09-2026**, a Python/Django application completed a manual GitHub Actions
workflow on the laptop runner. Its PostgreSQL and Redis services ran inside the
VM. The application is not included in this repository, so the results below
describe that installation rather than a bundled example you can run here.

## Test environment

| Component | Configuration |
| --- | --- |
| Host | Ubuntu Linux, x86-64, 16 logical CPUs, roughly 30 GiB usable RAM |
| Guest | Ubuntu 24.04 LTS, 4 vCPUs, 8 GiB RAM, 80-GiB thin disk |
| Execution | One manual GitHub Actions job at a time |
| Job storage | Writable overlay over a protected baseline |
| Application services | Temporary PostgreSQL and Redis containers |

This machine had more resources than the proposed entry-level laptop profile.
The results are not a benchmark for a 16-GB host.

## Results

| Check | Observed result |
| --- | --- |
| Library suite | 278 tests passed; one warning reported |
| Web suite | 1,118 tests passed |
| Python dependency audit | No known vulnerabilities reported at the time of the scan |
| Clean Python subprocess | Started successfully without the parent's environment |
| GitHub action runtime | Node24 declared by both pinned helper actions; Node20 warning absent |
| Job hooks | Start and completion hooks both succeeded |
| Guest reset | Used overlay discarded; fresh guest had no previous application workspace or started-job marker |
| Baseline permissions | Remained root-owned and non-writable while the guest ran |
| Controller and hook checks | 11 controller tests and two hook-contract tests passed |
| Workflow checks | Manual-only routing, self-hosted labels and runtime configuration passed validation |

This was a conventional CI run, not a Jev-guided exploration run. No Jev browser
coverage, latency or cost result is included in these totals. That layer is described
separately in the [planned Jev integration](jev-qa.md).

The successful run followed two setup fixes. Both were found by the real workflow,
not by changing application assertions to make the suite pass.

## Hook scripts were rejected

**Symptom:** the first job stopped before checkout because GitHub rejected the
configured hook paths.

**Cause:** the scripts were executable, but their filenames lacked a supported
script extension. The earlier synthetic lifecycle check had not exercised GitHub's
hook dispatcher.

**Fix:** rename the hooks with `.sh` suffixes and add a configuration check that
catches the original invalid filenames. The uncertain guest disk was retained for
inspection. A later real job confirmed both hooks executed successfully.

This exposed a gap between testing the controller's state transitions and testing
the external runner API it depends on.

## Python could not start in a clean environment

**Symptom:** a later run reached the web suite, but one fresh-process test failed
with exit code 127:

```text
error while loading shared libraries: libpython3.14.so.1.0: cannot open shared object file
```

**Cause:** Python had been installed beneath a relocated runner tool-cache path.
The parent process could find its shared library through the environment; the test's
clean child environment could not.

**Reproduction:** the exact SHA-verified Python 3.14.7 distribution failed at the
relocated path and started successfully under `/opt/hostedtoolcache`. Both checks
used disposable Ubuntu containers with networking disabled.

**Fix:** set `AGENT_TOOLSDIRECTORY` to the runner-writable canonical cache and add a
clean-environment Python startup check before dependency installation. The web test
remained unchanged and passed in the next complete workflow run.

The early check now catches this runtime failure without waiting for the full suite.

## A Node warning was a separate maintenance issue

The earlier action pins declared Node20 even though the runner executed them with
Node24. Checkout and Python setup succeeded, but GitHub emitted a deprecation warning.

Updating the pins to verified Node24 action releases removed that warning. The
Python shared-library fix was still needed: these were two different issues, not
two descriptions of the same failure.

## What remains to validate

| Next check | Current position |
| --- | --- |
| Installation from this repository | Runner code and standalone demo still to be packaged |
| Physical-host reboot | Service stop/start tested; unattended recovery after a real reboot still pending |
| Smaller hardware and other operating systems | Not tested |
| Browser journeys | Chromium installed; complete journey coverage not established |
| Jev-guided exploration | Planned; adapter, assertion controls and comparative coverage/cost evaluation pending |
| Load and security testing | Not completed |
| Production deployment and recovery | Outside this CI validation run |

The run demonstrates that this workload can complete and that its guest can be
replaced cleanly. Reproducing the setup on a second machine is the next portability
check.
