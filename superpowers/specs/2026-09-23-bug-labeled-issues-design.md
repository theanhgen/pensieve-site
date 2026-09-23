# Bug-labeled issue remediation design

## Scope

Fix the nine currently open issues labeled `bug` in `theanhgen/pensieve`:

- #156 deployment targets versus advertised iOS 17 support
- #154 release stats omit Share and Siri notes
- #153 fading live notes lose readable contrast
- #152 release stats use the CloudKit-mirrored store despite the local-only promise
- #128 StoreKit purchase reads entitlements too early
- #118 Siri and Shortcuts use English for Czech and Vietnamese users
- #112 monetisation tests do not exercise StoreKit's state machine
- #87 the no-clawback test is disconnected from StoreKit and restore is untested
- #81 CloudKit record conflicts may null an already-committed local delete

Unlabeled hypotheses #138, #137, and #136 are explicitly out of scope.

## Work decomposition

Work is split by subsystem rather than by issue to avoid contradictory changes:

1. **StoreKit:** #87, #112, and #128. Make entitlement state transaction-driven,
   cover purchase and restore through `SKTestSession`, and preserve explicit
   error reporting.
2. **Persistence and CloudKit:** #152 and #81. Keep release statistics in the
   intended local store and make merge/delete semantics explicit and tested.
3. **Platform and localization:** #156 and #118. Align all relevant targets
   with the supported deployment floor and provide localized Siri/Shortcut
   user-facing strings using existing localization conventions.
4. **Stats and accessibility:** #154 and #153. Aggregate notes from every
   supported capture origin and keep fading content above the established
   contrast threshold until expiry.

Agents may modify only their assigned subsystem and its directly related tests.
The parent agent integrates the branches and resolves cross-cutting conflicts.

## Validation

Each subsystem adds or updates focused regression tests. After integration:

1. Run all targeted tests for the changed subsystems.
2. Run the complete `scripts/test.sh` suite.
3. Have a dedicated reviewer agent inspect the complete diff and test results for
   regressions, missing issue coverage, error handling, and target consistency.
4. Resolve reviewer findings and rerun affected tests.

No test failures will be bypassed with `--skip-tests`.

## Release

After review and a clean worktree:

1. Commit the integrated fixes with the required Copilot co-author trailer.
2. Push the branch to `origin`.
3. Run `./upload.sh` with a concise changelog and without `--skip-tests`.

The upload script remains the source of truth for build numbering, archive
checks, signing, TestFlight upload, and `tf-build-*` tagging. Credential,
toolchain, signing, or App Store Connect failures are reported as blockers and
are not represented as a successful upload.
