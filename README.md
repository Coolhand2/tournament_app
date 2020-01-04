# Tournament Application

The big repository that combines all the services into one big stack that can be re-created in an AWS account.

**_This is a design document, and may be ahead of where the cloudformation stack files are._**

## How to Build

**_Fair Warning: These commands are not tested, and only built of a moderatly-educated guess to get to the type of development flow I wish to have._**

When a new release begins, we will expect each team to begin a release on their own repository:

```bash
git flow release start <X>
```

Then, the build manager for the application can start a release and pull in all of their updates:

```bash
git flow release start <X>
git submodule foreach 'git pull origin release/<X>'
git flow release publish <X>
```

We expect a CodeBuild pipeline to detect a new release branch and start a clean stack build of the application.

At this point you should notify the team to perform testing on the releasable version of the app, and push any updates to their submodules as necessary.

```bash
git push origin release/<X> --v --progress
```

So the buildmaster can then update their submodule. If you want, you can put this on a nightly build.

```bash
git submodule foreach 'git pull origin release/<X>'
git commit -am "Nightly release"
git push origin release/<X> --v --progress
```

After checkout is complete, each team finishes their respective releases.

```bash
git flow release finish <X>
git checkout master
git push origin master --tags -v --progress
```

Which then allows the build master to make a gold-copy for release.

```bash
git submodule foreach 'git pull origin master'
git flow release finish <X>
git checkout master
git push origin master --tags -v --progress
```

At this point we expect the CodePipeline process that's listening on origin/master to kick off and update all our stacks for us. Update times will likely vary depending on the severity of the changes.

**_I still need to answer the question of how do we separate the Test builds from the Ops updates in the stacks_**

## Current Design

* S3 Bucket serving a static website
* API Gateway/Lambda backend
* RDS Instance
* ElastiCache Instance

Currently the only thing this end of the project will do is setup the CI/CD pipeline to take push updates from it's own repository (allowing us to make automated releases.)

The API Backend will manage it's own RDS, and DynamoDB Cache, along with a build process to convert the code to API Gateway/Lambda combos.

The UI Frontend will manage it's own S3 bucket and build process. Likely also inducing a Route53 update as well. We shall see.

## Todo List

- [ ] Create starting-stack file (CI/CD Pipeline Setup)
  - Is there a way to auto-start that pipeline once it's done being setup?
  - Do you use CodeCommit for private applications not yet published (instead of GitHub), and push to that to start the pipeline?


## Management Policies

This specifies the current team positions and policies governing this project.

**_These policies are currently not enforced, due to this being a 1-man project._**

**_The following are liable to change at any time, upon approval of project and team leaders._**

### Team Positions

- Project Manager (creates release schedule, and schedules for hot fixes as necessitated by Team Managers)
- Build Manager (creates a release candidate for testing, or a final version for distribution)
- Team Manager (schedules issues for releases)

### Project Policies

The following policies are in place to help clarify the process of making a major, minor, or hotfix release.

**_These policies are liable to change at any time, with approval from project and team leads_**

#### Issue Severities

Issues will be ranked by the following levels, to induce the listed meanings. It will be up to the individual team managers to ensure proper classification of all issues.

- High (represents a major bug that team leads will have to schedule with the project lead and build manager to generate an out-of-cycle hotfix release)
- Medium (represents a minor bug that requires an investigation and ideally a resolution before the next minor release. Inclusion in a minor release will be discussed between team lead and project lead)
- Low (represents a minor bug that requires an investigation and ideally a resolution before the next major release. Inclusion in a major release will be discussed between team lead and project lead)
- Feature (represents a request for new functionality. May be split into further issues to ensure a timeline can be formed for inclusion into a projected release.)

#### Issue Statuses

- New (submitted by a user, awaiting verification)
- Verified (verified as unintended result of taking a specific action, or as a feature not yet implemented.)
- Closed (a fix has been included in a release.)

#### Issue Tags

A version tag will be utilized to identify the appropriate major version of the application that the bug was found in.

A special "on-hold" tag will be utilized for issues that have an external dependency before a fix can be applied.

#### Issue Management and Release Schedule relationships

- [ ] **_TODO: Clean up this section_**

A new major release should not be scheduled (at project lead discretion) while there are still open issues for the latest major release. (Open issues are identified as verified issues without the special "on-hold" tag.)

If a bug is found during a Release Candidate test, the severity of the issue (secondary to the complexity) will help calculate whether it's fixed before final release, or if it's pushed off to another (ideally the next minor) release.

No features will override the fixing of an open, verified bug. The only exception will be if implementing the feature will also fix the bug due to opportunistic refactoring required to implement said feature.

Each major release shall include a review of on-hold issues, and evaluate their justifications for being "on-hold". If this justification is found to be meritless, the appropriate team should work to include a fix for that bug in the next major release.

Each minor release shall strive to fix bugs identified in the previous major release. Not all bugs have to be addressed. It will be up to the team manager to schedule which issues could be resolved in upcoming minor releases.

A major release should not occur (at project lead discretion) while there are still open, verified issues identified for the previous major release.

A feature request should be broken down into constituent pieces that can lead towards it's formal inclusion in a major release. The constituent pieces should only apply to one team each (but not be limited to one piece per team), but may retain dependencies to ensure a timeline can be maintained.

A scheduled major release will never guarantee the inclusion of a feature. It should however, guarantee work on a constituent piece for that feature, at the team lead's discretion.

It will be assumed that if the release tagged on an issue is a previous version, and the status of the issue is not closed, that the issue still affects the functionality of the latest version.

Issues will be ranked (as previously mentioned) by the respective team lead. 

If the issue requires updates via multiple teams, the project manager will work with the team managers to ensure the issue is broken down into it's constituent pieces, that teams receive their own issues, and that they are linked appropriately to the parent issue for tracking purposes.
 
#### Release Schedules

Releases will be initially scheduled as such:

- At minimum one major release will be scheduled each year, as feature requests and minor issues permit.
- At minimum one minor release will be scheduled each month, as major issues permit.
- A hotfix will be scheduled as early as team manager can estimate completion of fix. It will be up to the team manager to ensure enough resources will be assigned to the issue to resolve it quickly and efficiently.
  - A suggestion is for the team lead to manage a list of team members that can be pulled to address the issue.
