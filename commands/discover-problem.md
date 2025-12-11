---
description: Analyze latest Jenkins build for openshift-knative project and report any failures
---

# Discover Problem Command

Analyze the latest Jenkins update-to-head build for the current openshift-knative project and report any issues found.

## Your Task

1. **Verify you're in a valid git repository**
   - Check for `.git` directory
   - If not found: "Not in a git repository. Please run this from an openshift-knative project directory."

2. **Detect the project and openshift-knative remote**
   - Run `git remote -v`
   - Look for `openshift-knative/<project-name>` pattern in the remotes
   - Extract the project name (e.g., `eventing-kafka-broker`, `serving`, `eventing`, `func`)
   - Store the remote name that points to openshift-knative (e.g., `upstream`, `origin`, etc.)
   - If not found: "Can't detect openshift-knative project from git remotes. Are you in the right directory?"

3. **Determine the Jenkins job name**
   - Use pattern: `job/ci/job/knative-nightly-ci-${PROJECT}`
   - Jenkins URL: `https://jenkins-csb-serverless-qe-master.dno.corp.redhat.com`

4. **Authenticate to Jenkins**
   - **First, check if a Jenkins MCP server is available**
   - If Jenkins MCP server is available: use it (authentication is handled by the MCP server)
   - If NO Jenkins MCP server is available:
     - Try environment variables: `JENKINS_USER` and `JENKINS_TOKEN`
     - If not found, prompt the user:
       - "Jenkins credentials needed. Please provide your Jenkins username:"
       - "Please provide your Jenkins token (create at https://jenkins-csb-serverless-qe-master.dno.corp.redhat.com/user/[USERNAME]/security/):"
     - Store credentials in memory for the current session

5. **Fetch latest build information**
   - If using Jenkins MCP server: use MCP server tools
   - If using direct access: use WebFetch with authentication
   - Get the most recent build from the Jenkins job
   - Retrieve build number, status, and timestamp

6. **Analyze the build**
   - If status is SUCCESS: "Latest build #XXX passed successfully. No issues to fix."
   - If FAILURE:
     - Fetch console logs (via MCP server or WebFetch with auth)
     - Parse logs to identify:
       - What stage failed
       - Error messages
       - Stack traces
       - File conflicts or patch issues

7. **Check for existing PRs (optional context)**
   - Search recent PRs in `openshift-knative/${PROJECT}` repo
   - Look for PRs that might address this issue
   - If found, report: "Note: There's a recent PR #XXX that might be related: [title]"

8. **Report findings clearly**
   - Build number and status
   - When it failed
   - Summary of the failure
   - Key error messages
   - Link to Jenkins build
   - Any related PRs

## Example Output Format

```
Jenkins Job: knative-nightly-ci-eventing-kafka-broker
Latest Build: #456 - FAILED (2 hours ago)
Failed Stage: Update to HEAD

Error: Patch application failed
  File: config/reconciler/deployment.yaml
  Conflict: Resource already exists in upstream

Link: https://jenkins-csb-serverless-qe-master.dno.corp.redhat.com/job/ci/job/knative-nightly-ci-eventing-kafka-broker/456/console

Note: No recent PRs found addressing this issue.
```

## Important Notes

- Prefer using Jenkins MCP server if available (handles authentication automatically)
- If no MCP server is configured, fall back to direct access with user-provided credentials
- Be clear and concise in reporting
- If any step fails, provide helpful error messages and troubleshooting guidance