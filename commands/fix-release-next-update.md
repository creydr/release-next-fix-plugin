---
description: Discover and fix Jenkins update-to-head failures for openshift-knative projects
---

# Fix Release-Next Update Command

Systematically discover, fix, test, and create a PR for Jenkins update-to-head failures in the current openshift-knative project.

## Your Task

Invoke the `release-next-update-fix` skill to run the complete fix workflow.

Use the Skill tool to execute the skill:

```
Skill: release-next-update-fix
```

The skill will guide you through:
1. Discovering the problem (via discover-problem logic)
2. Checking for existing PRs
3. Creating a fix branch
4. Implementing the fix
5. Testing locally
6. Pushing to fork
7. Creating a PR

## Important

This command uses a skill to ensure all critical steps are followed systematically and nothing is skipped.