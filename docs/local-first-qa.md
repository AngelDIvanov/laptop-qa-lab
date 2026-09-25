# From a local app to a public launch

A product can be tested before it has a public URL. Start with the application on
`localhost`, make its checks repeatable, and use the spare laptop to run those
checks in a clean environment. Hosting and a domain can follow once the basics work.

The workflow below is a way to organize that process. The runner packaging itself
is still on the [project roadmap](../README.md#roadmap).

## Make a clean checkout usable

Start with one question: could someone set up the application using only the
repository and its instructions?

Document the runtime version, dependency installation, required services, database
migrations and start command. Provide sample configuration without credentials and
seed the database with synthetic records. A setup that depends on an undocumented
manual fix will be difficult to reproduce in CI.

## Test outcomes that matter to a user

Choose a few important journeys and turn them into assertions. For example, a small
task-tracking application might need these checks:

| Journey | Expected result |
| --- | --- |
| Create a task | The saved task appears after reloading the page |
| Submit an empty title | A validation error appears and no task is saved |
| Sign out | Protected pages are no longer accessible |
| Open another user's task | Access is denied |
| Save while a service is unavailable | A useful error appears without corrupting existing data |

Cover business rules with unit tests, database/service boundaries with integration
tests, and visible behaviour with browser tests. A browser script is useful when
it checks the result of an action, not just whether the click completed.

For email, payments and similar integrations, use mocks or the provider's sandbox.
That keeps routine tests repeatable without sending real messages or creating real
transactions.

## Move the checks into a clean job

Once the application has test commands, a manual GitHub workflow can run them on a
selected commit. In this design, the QA laptop receives the job inside a dedicated
VM and creates temporary databases there.

A useful order is:

1. Check out the selected revision and prepare its runtime.
2. Install dependencies and verify that the runtime starts correctly.
3. Create test services, apply migrations and load fixtures.
4. Run unit and integration tests, followed by the configured browser journeys.
5. Check dependencies and collect failure details.
6. Finish the job and replace the used VM disk.

Pin action commits and record runtime/dependency versions so a failure can be
reproduced. No public application endpoint or router port forwarding is needed:
the runner connects out to GitHub, where job results and logs are stored.

## Add exploration alongside fixed tests

A fixed browser script follows the path its author chose. The planned Jev layer
would help choose the next permitted check from the page that is actually visible,
using the current goal and prior observations. Playwright would perform the action;
existing assertions would still decide whether the expected result occurred.

For the task-form example, Jev might help select a useful route to exercise an
untested validation branch. It would not replace the rule that an empty title must
be rejected, invent new form data or override a failed assertion. Exploration would
have explicit step and API-cost limits.

This is a planned addition, not part of the completed CI run. The [Jev design](jev-qa.md)
explains how to test whether it adds coverage over fixed scripts.

## Use a failure to improve the setup

Find the first failed step and distinguish an application defect from a setup
problem. A missing library, unavailable database or incorrect runtime path may stop
the test before it reaches application code.

Reproduce the failure with the smallest useful check, fix its cause and keep that
check in the workflow when it will catch the problem earlier next time. The
[validation case study](evidence.md) shows this with a Python startup failure.

Successful cleanup does not change a failed test result. Likewise, an interrupted
job may have left work unfinished; investigate it before repeating operations with
external side effects.

## Decide what to host

With a working application and dependable checks, infrastructure decisions become
more concrete. You know which services the app requires, how it is configured and
how its database is initialized.

That is a useful point to compare hosting options, estimate costs and choose a
domain. Registering a domain earlier is fine too—it is simply not a prerequisite
for testing application behaviour.

Measure application resource use and expected traffic separately. A test suite's
runtime on one laptop does not tell you how many production users a server can handle.

## Test the deployment in staging

Local QA catches application and setup problems. Staging adds the infrastructure
that users will actually reach:

| Area | What to check |
| --- | --- |
| Public endpoint | DNS, TLS, reverse proxy behaviour and secure cookies |
| Configuration | Production settings, service permissions and secret delivery |
| Integrations | Email, OAuth callbacks and webhooks in safe test modes |
| Data | Migrations, backups and a demonstrated restore |
| Operations | Logs, metrics, alerts and a usable rollback procedure |
| Capacity | Representative load against an isolated target |

Keep deployment separate from the QA job. A green suite is valuable input to a
release decision; staging confirms the parts that only exist once the product is
hosted.

The progression is **build locally → test repeatably → fix failures → plan hosting
→ validate staging → launch**. The laptop supplies the feedback loop, rather than
becoming the production server.
