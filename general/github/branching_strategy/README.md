# Branching Strategy
Branch names should be modeled after their associated environment.  Here at AMU, those environments are commonly "staging" and 
"production".  The "staging" branch contains the "next" version of the application and is used for final stress testing and client/product owner approvals before going live.
The "production" branch is the currently released version of the application, accessible to the client/end users.  Occasionally, projects will also have a "demo" remote branch where specific versions can be tested/approved for release.  In most cases, these should be the only branches that exist on the remote within each project.

# Creating a Branch Locally

All development should be done in a local environment to ensure code & data stability.  
1. In almost all development scenarios, you'll begin with cloning the default branch, which at AMU is the "staging" branch.
2. Create your branch name based on associate JIRA ticket number, for example `TICKET-123-name-of-branch`.  This not only allows for utilization of [git hooks](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/general/github/git-hooks) for product owners to track issues but also assists other developers to reference scope for pull requests and releases.
3. Once your code changes are made and successfully passing tests locally, you'll need to perform a [pull request](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/general/github/pull-requests) to merge your changes into "staging".
