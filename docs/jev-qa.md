# Jev-guided browser QA

Scripted tests are good at repeating known checks. An exploratory layer can help
choose which permitted interaction to try next when the page or journey changes.
The planned addition to Laptop QA Lab uses **Jev from TypeSafe** for that narrow
decision, while code keeps control of the browser and the test verdict.

**Status: design only.** This adapter is not implemented in the public repository,
and the passing prototype CI run did not use Jev for browser exploration. The
existing test totals are not evidence of this capability.

## What Jev contributes

TypeSafe accepts application state and typed questions. For example, a **Choice**
question returns one option from a defined set, together with probabilities and
confidence. Jev supplies a judgment that the harness can use; it does not execute
the chosen function or independently operate the browser.

For this design, the state would contain:

- The current test goal and synthetic fixture values.
- Relevant page text and accessible control labels.
- Identifiers for currently available, permitted actions.
- Recent actions, observed results and coverage already collected.

Jev takes text/structured textual state, not screenshots. Playwright could retain
screenshots for human diagnosis, but the model would receive a deliberately selected
text summary rather than an unrestricted HTML dump.

## Example: checking a task form

Suppose the goal is to check that a task with an empty title is rejected. A scripted
test already defines the expected result: a validation message appears and no task
is created.

The explorer would gather the visible controls and offer a small set of options,
such as opening the task form, submitting a predefined empty-title fixture,
inspecting the resulting message, or stopping. Only actions permitted by the test
scenario would be offered.

Jev could help select the option that best advances that goal on the current page.
The harness would then verify that the action still exists, execute it with
Playwright, and run the assertions. If a task was created unexpectedly, the test
would fail—even if a model judgment suggested the journey looked successful.

Form values come from fixtures. The design does not ask Jev to invent credentials,
generate arbitrary JavaScript or write a new test oracle during the run.

## Proposed control loop

1. **Observe:** collect a fresh, filtered page state and the current test goal.
2. **Constrain:** build candidate actions from reviewed handlers and allowed test
   origins. Include a stop outcome when no useful safe action is available.
3. **Budget:** check the step, time, request and cost limits before reserving a call.
4. **Judge:** send the state and a narrow Choice question directly to TypeSafe.
5. **Validate:** check the returned action identifier, uncertainty policy and page
   freshness. A stale or uncertain result pauses exploration rather than expanding
   its permissions or switching providers.
6. **Execute:** let Playwright perform the permitted interaction with fixture inputs.
7. **Assert and record:** capture the observed outcome, run deterministic checks and
   update coverage before considering another bounded step.

Typed output makes the interface predictable; it does not guarantee that a judgment
is correct. Confidence thresholds need evaluation against the demo's outcomes,
not a universal number copied from an example. Page content is test data, not an
instruction source for changing permissions or running commands.

## Controls the harness would own

| Control | Intended behaviour |
| --- | --- |
| Test environment | Synthetic accounts and data; approved local/staging targets only |
| Browser permissions | Reviewed action handlers and allowed navigation/request origins |
| Test verdict | Assertions determine pass/fail; Jev judgments remain separate observations |
| Execution limits | Finite browser steps, wall-clock duration and inference requests |
| Cost accounting | Persist reservations and usage across restarts; unresolved calls keep their reservation until reconciled |
| Service failure | Stop and report the requested Jev stage as unavailable/incomplete, not passed; no automatic provider fallback or blind retry |
| Credentials | Inject only for an explicitly enabled run; never bake them into a baseline or include them in page state |
| Reports | Separate assertion failures, model uncertainty, infrastructure errors and incomplete exploration |

The laptop would run the browser and application; Jev inference would use TypeSafe's
service. This does not require hosting the model or adding a GPU to the laptop,
but it adds network, privacy and API-cost considerations to the otherwise non-AI
workflow. Only minimized synthetic test state should leave the guest for inference.

## How to validate the addition

A useful pilot needs a small demo with known good and deliberately broken journeys.
Compare a fixed Playwright path and the Jev-guided path against the same fixtures,
then record:

- Which expected failures each approach detects or misses.
- Distinct pages, controls and branches exercised, including repeated-action loops.
- Assertion outcomes separately from model choices and uncertainty.
- End-to-end duration, inference latency, request counts and usage/cost.
- Behaviour at budget limits, stale page state and service failure.
- Whether an interrupted run can be reviewed without repeating side effects.

A deliberately failing assertion must stay failed regardless of the model's answer.
These measurements would establish whether Jev adds useful exploration—not merely
whether an API call succeeds. Exploratory navigation is also separate from load
testing; generating browser clicks is not a throughput benchmark.

## TypeSafe references

- [Jev and System One](https://docs.typesafe.ai/introduction)
- [Building with TypeSafe](https://docs.typesafe.ai/concepts/how-to-build-with-system-one)
- [State and supported input](https://docs.typesafe.ai/concepts/state)
- [Choice questions](https://docs.typesafe.ai/primitives/choice)
- [Typed function selection](https://docs.typesafe.ai/cookbooks/function_calling)
