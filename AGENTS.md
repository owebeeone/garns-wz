Read and follow `AGENTS_GWZ.md` before doing any work in this workspace.

## Application UI dialogs

Avoid blocking dialogs in application user interfaces. Do not introduce native
browser `alert()`, `confirm()` or `prompt()` calls, or replace them with a blocking
custom modal by default. Prefer inline status/error messages, direct reversible
actions with Undo, and nonblocking contextual controls. When confirmation is
necessary, present it inline beside the action with clear consequences and an
explicit cancel path. Preserve user drafts and recovery information without
interrupting the workflow.
