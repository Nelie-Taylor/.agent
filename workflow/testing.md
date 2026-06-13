## QC / Testing Workflow

Process for the agent to QC (test) a card on the project's GitHub Project board.

> Project-specific values (project number/ID, repo, status field ID, column option
> IDs, `gh` path) live in the project's `CLAUDE.md`. Read them from there at run time
> — do NOT hardcode them in this file.

## Working rules
- Language for all GitHub content (comments, bug reports): follow the project convention
- Per run: handle **1 card** (front of queue, or a card specified by the user)
- Test method is **per-feature business logic** — NOT defined here. This doc defines the flow only.
- Only move through the columns below; do not auto-close issues / move to Done
- PASS = change status only, do NOT merge/push the PR (deploy is done by someone else)

## Columns used
`Ready for QC` → `In review` → (`Test fail` | `Ready for deploy`)

Resolve each column's status option ID from the project before editing:
```bash
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

## Flow

```
1. Get the first "Ready for QC" card (or a user-specified card)
   └─ empty → report "No card waiting for QC", stop
2. Read the issue: description, acceptance criteria, test report
   (test-case table), linked PR/diff
3. Move card → "In review"   ← DO THIS BEFORE testing
4. Test (per feature business logic)
5. Result:
   ├─ FAIL → move to "Test fail" + comment the bug (free-form)
   └─ PASS → move to "Ready for deploy"
6. Report back to user: which card, PASS/FAIL result, issue link
```

## Step details

### 1. Get "Ready for QC" cards
```bash
gh project item-list <PROJECT_NUMBER> --owner <OWNER> --format json
```
Filter items with Status = "Ready for QC". Take the first item (or the user-specified
card). You need the `item id` (to change status) and `content` (issue number/URL).

### 3. Move to "In review" (before testing)
```bash
gh project item-edit \
  --project-id <PROJECT_ID> \
  --id <ITEM_ID> \
  --field-id <STATUS_FIELD_ID> \
  --single-select-option-id <IN_REVIEW_OPTION_ID>
```

### 5a. FAIL → "Test fail" + bug comment
```bash
gh project item-edit --project-id <PROJECT_ID> --id <ITEM_ID> \
  --field-id <STATUS_FIELD_ID> --single-select-option-id <TEST_FAIL_OPTION_ID>
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "<bug description>"
```
Bug comment: describe freely per context (what's broken, where, impact). No fixed template required.

### 5b. PASS → "Ready for deploy"
```bash
gh project item-edit --project-id <PROJECT_ID> --id <ITEM_ID> \
  --field-id <STATUS_FIELD_ID> --single-select-option-id <READY_FOR_DEPLOY_OPTION_ID>
```
Do NOT merge the PR. No mandatory comment.

## Notes
- If a card has no linked issue (draft item) → report to the user, don't guess the test scope.
- Status-change errors are usually a wrong `ITEM_ID` (the project item id, DIFFERENT from the issue number).
