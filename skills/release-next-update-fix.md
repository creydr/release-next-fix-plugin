---
name: release-next-update-fix
description: Systematic workflow to fix Jenkins update-to-head failures in openshift-knative projects
---

<EXTREMELY-IMPORTANT>
This skill provides a systematic checklist-driven workflow for fixing Jenkins update-to-head failures.

You MUST follow each step in order and use TodoWrite to track progress through the checklist.

DO NOT skip steps. DO NOT rationalize shortcuts. The checklist exists to prevent common mistakes.
</EXTREMELY-IMPORTANT>

# Release-Next Update Fix Workflow

## Prerequisites

This skill works best with a Jenkins MCP server configured, but can also work with direct authentication:
- **Preferred:** Jenkins MCP server configured with the Jenkins URL and credentials
- **Alternative:** User can provide Jenkins credentials when prompted (stored in memory for the session)

## Mandatory Checklist

Create TodoWrite todos for EACH of these items before starting:

```
☐ Detect project and analyze problem (via discover-problem)
☐ Check for existing PRs
☐ Create fix branch from openshift-knative remote
☐ Implement the fix
☐ Test fix locally (smart detection)
☐ Confirm fork remote
☐ Push to fork
☐ Offer to create PR
```

## Phase 1: Discovery

**Run the discover-problem workflow:**

1. Verify you're in a valid git repository

2. Detect the project and openshift-knative remote (`git remote -v`)
   - Look for `openshift-knative/<project-name>` pattern
   - Extract project name
   - **IMPORTANT: Store the remote name** that points to openshift-knative (e.g., `downstream`, `midstream`, `upstream`, `openshift`, etc.) - you'll need this in Phase 3

3. Determine Jenkins job: `job/ci/job/knative-nightly-ci-${PROJECT}`

4. Authenticate to Jenkins
   - **First, check if a Jenkins MCP server is available**
   - If Jenkins MCP server is available: use it (authentication is handled by the MCP server)
   - If NO Jenkins MCP server is available:
     - Try environment variables: `JENKINS_USER` and `JENKINS_TOKEN`
     - If not found, prompt the user:
       - "Jenkins credentials needed. Please provide your Jenkins username:"
       - "Please provide your Jenkins token (create at https://jenkins-csb-serverless-qe-master.dno.corp.redhat.com/user/[USERNAME]/security/):"
     - Store credentials in memory for the current session

5. Fetch latest build information
   - If using Jenkins MCP server: use MCP server tools
   - If using direct access: use WebFetch with authentication
   - Get the most recent build from the Jenkins job
   - Retrieve build number, status, and timestamp

6. Analyze logs to identify root cause
   - Fetch console logs (via MCP server or WebFetch with auth)
   - Parse logs to identify failures

7. Report findings to user

**If build passed:**
- Stop gracefully: "Latest build passed. No issues to fix."

**If build failed:**
- Continue to Phase 2

## Phase 2: Check for Existing PRs

1. Search for open and recently merged PRs in `openshift-knative/${PROJECT}`
2. Look for PRs that might address the same issue
3. If found:
   - Report: "Found existing PR #XXX (opened X days ago): [title]"
   - Ask user:
     ```
     1) Continue with new fix anyway
     2) Review existing PR
     3) Abort
     ```
   - Handle user choice
4. If none found: continue to Phase 3

## Phase 3: Create Fix Branch

1. Ensure you're on latest main from the openshift-knative remote (the remote name you stored in Phase 1):
   ```bash
   git fetch <openshift-knative-remote>
   ```

2. Create fix branch from that remote's main branch:
   ```bash
   git checkout -b fix-update-to-head-$(date +%Y%m%d-%H%M%S) <openshift-knative-remote>/main
   ```
   Example: If the remote is named `openshift`, use `openshift/main`

3. Confirm branch created successfully

## Phase 4: Implement the Fix

1. Based on root cause analysis, implement the fix
   - Patch conflicts → update patch files
   - Build failures → fix build configuration
   - Test failures → fix test code
   - Other issues → appropriate fix

2. Show user what was changed:
   - Files modified
   - Summary of changes
   - Reasoning

3. Commit the changes:
   ```bash
   git add <files>
   git commit -m "Fix update-to-head failure: <brief description>"
   ```

## Phase 5: Test Fix Locally (Smart Detection)

Based on the failure type, run appropriate verification:

**If patch conflict:**
- Verify patches apply cleanly
- Run: `make apply-patches` or equivalent

**If build failure:**
- Run the build command
- Example: `make build` or `go build ./...`

**If test failure:**
- Run the specific tests that failed
- Example: `go test ./path/to/package`

**If linting/formatting:**
- Run linter/formatter
- Example: `make lint` or `gofmt`

**Report test results:**
- If passed: "✓ Local verification passed"
- If failed:
  - Show error
  - Ask: "Fix needs adjustment. Analyze failure and iterate?"
  - If yes: return to Phase 4
  - If no: abort

## Phase 6: Confirm Fork Remote

1. List git remotes: `git remote -v`

2. Try to identify fork:
   - Look for remotes that are NOT the openshift-knative remote (from Phase 1)
   - Check for remotes with different GitHub user than `openshift-knative`
   - Common names: `origin`, `fork`, or your GitHub username

3. **If one fork candidate found:**
   - Ask: "Is `<remote-name>` (<url>) your fork? (yes/no)"
   - If yes: use it
   - If no: proceed to step 5

4. **If multiple candidates found:**
   - Show list: "Found multiple remotes: [list]"
   - Ask: "Which remote is your fork?"
   - Use selected remote

5. **If no fork found:**
   - Ask: "What's your fork repository URL or remote name?"
   - If URL provided: `git remote add fork <url>`
   - If remote name provided: use that remote
   - Verify the remote exists

## Phase 7: Push to Fork

1. Push branch to fork:
   ```bash
   git push <fork-remote> <branch-name>
   ```

2. Handle errors:
   - No permission → check remote URL and credentials
   - Network error → offer to retry
   - Other errors → show error, ask how to proceed

3. Confirm push succeeded

## Phase 8: Offer to Create PR

1. Ask user: "Ready to create a PR to openshift-knative/${PROJECT}:main?"

2. **If yes:**
   - Check if `gh` CLI is installed: `command -v gh`
   - **If `gh` installed:**
     - Create PR with:
       ```bash
       gh pr create --repo openshift-knative/${PROJECT} \
         --base main \
         --head <fork-user>:<branch-name> \
         --title "Fix update-to-head failure: <brief description>" \
         --body "<PR body>"
       ```
     - PR body template:
       ```markdown
       ## Problem
       Build #XXX failed on update-to-head job (release-next branch)
       Failed stage: <stage>
       Error: <error message>

       Jenkins: <build link>

       ## Solution
       <what was fixed and why>

       This fix will be included in the next release-next generation.

       ## Testing
       - [x] Verified patch applies cleanly (or appropriate tests)
       - [x] Build succeeds locally
       - [x] Tests pass

       🤖 Generated with Claude Code via release-next-fix plugin
       ```
     - If existing related PR: add "Related to #XXX" in body
     - Show PR URL to user

   - **If `gh` not installed:**
     - Show message: "GitHub CLI (`gh`) is required to create PRs"
     - Provide link: "Install from: https://cli.github.com/manual/installation"
     - Report: "Branch pushed to fork. You can create PR manually when ready."

3. **If no:**
   - Report: "Branch pushed to <fork-remote>. You can create PR manually when ready."

## Phase 9: Complete

Mark all todos as completed and summarize:
- ✓ Problem discovered: <summary>
- ✓ Fix implemented: <summary>
- ✓ Tests passed
- ✓ Branch pushed to fork
- ✓ PR created (if applicable): <URL>

## Error Handling

**Not in valid repo:**
- "Not in a git repository. Please run this from an openshift-knative project directory."

**Can't detect project:**
- "Can't detect openshift-knative project from git remotes. Are you in the right directory?"

**Jenkins authentication fails:**
- If using MCP server: report MCP authentication error, guide user to check MCP configuration
- If using direct access: verify credentials are correct, offer to re-prompt for credentials

**Build is passing:**
- "Latest build passed successfully. No issues to fix."
- Stop gracefully

**Local tests fail:**
- Allow iteration: analyze → adjust fix → retest
- Don't proceed to push until tests pass

**Fork push fails:**
- Show error clearly
- Offer to retry or abort

## Key Principles

- **One step at a time:** Complete each phase before moving to next
- **Always test:** Don't skip local verification
- **Always confirm:** Check fork remote with user before pushing
- **Clear communication:** Keep user informed at each phase
- **Handle errors gracefully:** Provide clear guidance when things fail
- **Use TodoWrite:** Track progress through the checklist