# Evidence and limitations

## What was actually checked

Reference validation date: **25-09-2026**.

The working reference is private. This public-facing draft shares aggregate,
non-sensitive observations—not its source, registration state, logs or run URLs.
Until a standalone demo is supplied, readers cannot reproduce these application
results from this directory.

| Observation | Result / scope |
| --- | --- |
| Library tests | 278 passed; one warning reported |
| Web tests | 1,118 passed |
| Python dependency audit | No known vulnerabilities reported at run time |
| Node runtime | Pinned GitHub helper actions explicitly declared Node24; old Node20 deprecation warning absent |
| Clean Python subprocess | Passed without inheriting the parent's loader environment |
| Runner hooks | Actual job-started and job-completed hooks passed |
| Completed-job cleanup | Writable disk discarded; next guest had no previous workspace or started-job marker |
| Baseline protection | Root-owned, non-writable baseline remained protected while the guest ran |
| Local lifecycle checks | 11 controller tests and two hook-contract tests passed |
| Workflow contracts | Manual-only, self-hosted-only routing and runtime configuration checked |

The reference used a 4-vCPU, 8-GiB guest on an x86-64 laptop with 16 logical CPUs and
roughly 30 GiB usable RAM. This does **not** establish performance on the smallest
machine described in the requirements.

## Failure cases that improved the design

### Hook filenames

The first real job exposed that executable hooks also needed a supported script
suffix. Both filenames were corrected to `.sh`, and a contract check caught the
original failure. The uncertain guest state was preserved for review rather than
silently reused.

**Lesson:** a successful synthetic lifecycle does not establish compatibility with
the real GitHub runner's hook dispatcher.

### Clean Python subprocess

A later run reached the web suite but one test could not start Python: the dynamic
loader could not find `libpython3.14.so.1.0` after the test cleared the environment.
The downloaded interpreter was under a relocated tool-cache path.

The exact SHA-verified Python distribution reproduced that failure at the relocated
path and passed at `/opt/hostedtoolcache`, in offline disposable containers. The
workflow selected the canonical cache and added an early clean-environment check.
Application assertions were not weakened to obtain a pass.

**Lesson:** test the actual interpreter/runtime boundary, not just imports in a
parent process with a helpful inherited environment.

### Node action declarations

Earlier pinned action versions declared Node20 and were being forced onto Node24.
Their warning was separate from the Python test failure. Updating to verified
Node24 action commits removed the warning; it was not a replacement for fixing
Python's loader problem.

**Lesson:** pinning protects reproducibility, but old pins still need deliberate
maintenance. Distinguish warnings from the step that actually failed.

## Still unverified or outside scope

- Independent installation from this public draft: code/demo not packaged yet.
- Physical host reboot and readiness without a desktop login: configured, not verified.
- The smallest proposed laptop profile and other host operating systems.
- Complete browser-journey coverage: installing Chromium is not proof of it.
- Exploratory/AI-driven testing, a load-test programme or a security audit.
- High availability, multi-tenant hostile-code execution or production-scale capacity.
- Production deployment, rollback, disaster recovery or public launch readiness.

A green CI suite is useful evidence for a defined revision and set of assertions.
It must not be presented as proof that the entire product is correct or ready to launch.
