# UniPlanner test-project preview — assignment and Storage checks

This repository is a browser-test deployment only. It connects to the isolated Supabase project `uniplanner test` (`rigsjtbaoxkdbumwuhdn`), not the production project.

The app also uses separate browser-local keys:
- `uniplanner_test_v2_data`
- `uniplanner_test_v2_prefs`
- `uniplanner_test_v2_updated`
- `uniplanner_test_v2_notification_ledger`

This prevents this test preview from overwriting the planner's local data keys used by the production GitHub Pages site on the same `github.io` origin.

## Browser test: two approved accounts

Use two existing approved accounts in the isolated test Supabase project. Do not use production-only accounts or production credentials.

1. Sign in as Account A and create a disposable private study workspace. Invite Account B and have B accept the invitation.
2. As A, open the workspace's **Assignments** tab. Create a disposable assignment with a clear test title and attach a real PDF as the original resource.
3. As B, open the same workspace, download the original resource, and upload a first submission file.
4. As B, upload a second submission with a different filename. Confirm the earlier version remains listed.
5. As A, download both of B's submission versions. Confirm both files open and their contents match the uploads.
6. As A, close submissions for the assignment. Confirm B no longer sees an upload control and that a direct attempt to insert a submission for the closed assignment is rejected by the database policies (do not weaken or bypass RLS to perform this check).
7. As A, delete the disposable workspace. The app should report successful deletion, then process the queued Storage cleanup while refreshing the workspace list.
8. In the test project's Supabase Dashboard, confirm the disposable workspace's object prefix no longer exists in the private `uniplanner-workspace-files` bucket and the corresponding cleanup-job row has been removed.
9. Repeat the file picker, upload, and download steps in iPhone Safari. Test both choosing a file and downloading a file.

## Expected behavior

- Only approved workspace members can see assignments and files for their workspace.
- Original assignment resources can be added only by the assignment creator or workspace owner.
- Members can submit only inside their own user-specific folder; other members may download submissions in the same workspace.
- A closed assignment rejects new uploads at both the UI and database/Storage policy layers.
- Each submission version has a distinct object path and remains visible until the submitter removes it or the workspace is deleted.
- Deleting a workspace removes its private file objects through the Storage API. It must not use SQL DELETE against `storage.objects`.
- If listing or file deletion fails, the cleanup job remains queued so another workspace-list load can retry it.

## Safety boundary

All test workspaces, assignments, metadata, and objects must be created in the isolated test project. Do not run these tests against the production Supabase project. Do not apply assignment migrations to production until desktop and iPhone file tests have passed.
