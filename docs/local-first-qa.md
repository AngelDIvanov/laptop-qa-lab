# Check the product before taking it online

A public URL is not required to start building confidence in a product. Develop
locally, automate the checks that matter, and run them in a clean environment before
committing to a production hosting plan.

This is a practical early-stage workflow—not a rule that infrastructure or a domain
must never be purchased early, and not a substitute for production-readiness work.

## 1. Get the application working locally

Write down how to start it from a clean checkout. Identify its runtime, services,
migrations and configuration. Use disposable databases and synthetic accounts.
Keep secrets and real customer data out of test fixtures and Git history.

You should be able to explain the application's critical journeys. For example:

- Can a user sign in and sign out?
- Can they create, read, change and remove the records they are allowed to manage?
- Are records belonging to another user inaccessible?
- Do invalid inputs and external-service failures produce useful errors?

These are suggested tests, not claims about coverage already supplied by this lab.

## 2. Turn expectations into executable checks

Use the test framework appropriate to your application:

- **Unit tests** for application rules and edge cases.
- **Integration tests** for database and service boundaries.
- **Browser tests** for important visible behaviour and navigation.
- **Dependency and secret checks** for known vulnerabilities and accidental exposure.

Browser automation must assert an outcome: a saved record appears, an unauthorized
request is rejected, or an expected error is visible. Opening Chromium or clicking
a button is not, by itself, proof the feature works.

External calls should be mocked or explicitly limited to safe test environments.
Do not repeatedly purchase, send real messages, run paid inference or mutate
production accounts to manufacture a green result.

## 3. Run a clean, repeatable QA job on the spare laptop

Commit a trusted revision and start the manual GitHub workflow. The application is
executed in a dedicated VM rather than in the laptop owner's development home.
Temporary service containers and synthetic data are created inside that VM.

A fresh disk helps expose dependencies on leftovers from earlier runs. Pin the
inputs you can: source revision, action commits, runtime versions and dependency
versions. Record unavoidable external dependencies instead of claiming perfect
reproducibility.

No public domain, public application endpoint or inbound router port is required.
GitHub is still the remote coordinator; results and logs live there. This design is
not an air-gapped or fully local CI service.

## 4. Read the result, not just the final log lines

- A failed test means the job failed, even if container cleanup succeeded.
- A passing suite means the selected checks passed against the selected revision.
- A dependency audit with no findings means no known issues were reported by that
  scan at that time—not that the application is vulnerability-free.
- Inspect the first relevant failure, fix its cause, and rerun deliberately.
- Preserve uncertain or interrupted runs for review; do not silently erase evidence
  and replay a workload that may have produced external side effects.

The job should not deploy anything merely because tests passed.

## 5. Decide whether to invest in hosting and a domain

Once the product works locally and the important checks are repeatable, you can
make a more informed decision about cloud hosting, a domain and release tooling.

Use what you learned to identify required services, storage and operational needs.
Laptop test duration is not a production-sizing benchmark. Application latency,
concurrency and resource use need separate, bounded measurements with a suitable
load generator and target.

The benefit is avoiding a situation where you are paying to keep a server online
while still discovering basic setup and application problems. It is not a promise
that local QA removes every reason to use cloud development or staging.

## 6. Add production-like staging before launch

Local results cannot validate infrastructure that has not been configured. Before
opening the product to users, separately check:

- DNS, TLS, reverse proxies and secure cookies.
- Production configuration, IAM and secret management.
- Real email, OAuth callbacks, webhooks and other integrations in their safe test modes.
- Database migration, backup **and restore** procedures.
- Deployment rollback and data compatibility.
- Logs, metrics, alerts and useful incident diagnostics.
- Performance under a representative, authorized load.
- Privacy, authorization, accessibility and security review appropriate to the product.

Cloud accounts, domains and production credentials belong to this separate phase.
They are not part of the laptop runner's default permissions. A release remains an
explicit decision rather than an automatic consequence of a green CI job.

## Cost and scope

You reuse compute you already own, but electricity, hardware wear, network access,
GitHub storage and external services may still cost money. The laptop is an on-demand
QA worker, not a highly available production server.

The intended progression is:

**Local product → useful tests → repeatable isolated QA → production planning → staging → deliberate launch.**
