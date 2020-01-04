# Tournament Application

The big repository that combines all the services into one big stack that can be re-created in an AWS account.

**_This is a design document, and may be ahead of where the cloudformation stack files are._**

## Current Design

* S3 Bucket serving a static website
* API Gateway/Lambda backend
* RDS Instance
* DynamoDB Instance with TTL (acts like a cache)

### CI/CD Pipeline

The pipeline-stack.yml file will handle the initial setup of the CodeBuild pipeline that will allow for the API code to be translated into Api Gateway Pathways and Lambda functions; as well as allowing for the UI code to be translated into an RIA (via nodejs compilation) and stored into it's own S3 bucket.

## Todo List

* [ ] Create pipeline-stack file (CI/CD Pipeline Setup)
* [ ] Create api-stack file (Translate code into API Gateway/Lambda, maintain RDS instance and DynamoDB Cache)
* [ ] Create ui-stack file (Compile Code into something that can be stored into an s3 bucket for static dissemination)

### Questions

* Is there a way to auto-start the pipeline once it's done being setup?
* Would you use CodeCommit for private applications (instead of GitHub), and push to that to start the pipeline?

## Management Policies

This specifies the current team positions and policies governing this project.

**_These policies are currently not enforced, due to this being a 1-man project._**

**_The following are liable to change at any time, upon gaining other people to handle certain functions; pending their feedback, of course._**

### Team Positions

* Project Manager (creates release schedule, and schedules for hot fixes as necessitated by Team Managers)
* Build Manager (creates a release candidate for testing, or a final version for distribution)
* Team Manager (schedules issues for releases)

### Project Policies

The following policies are in place to help clarify the process of making a major, minor, or hotfix release.

**_These policies are liable to change at any time, with approval from project and team leads_**

#### Issue Severities

Issues will be ranked by the following levels, to induce the listed meanings. It will be up to the individual team managers to ensure proper classification of all issues.

* High (represents a major bug that team leads will have to schedule with the project lead and build manager to generate an out-of-cycle hotfix release)
* Medium (represents a minor bug that requires an investigation and ideally a resolution before the next minor release. Inclusion in a minor release will be discussed between team lead and project lead)
* Low (represents a minor bug that requires an investigation and ideally a resolution before the next major release. Inclusion in a major release will be discussed between team lead and project lead)
* Feature (represents a request for new functionality. May be split into further issues to ensure a timeline can be formed for inclusion into a projected release.)

#### Issue Statuses

* New (submitted by a user, awaiting verification)
* Verified (verified as unintended result of taking a specific action, or as a feature not yet implemented.)
* Closed (a fix has been included in a release.)

#### Issue Tags

A version tag will be utilized to identify the appropriate major version of the application that the bug was found in.

A special "on-hold" tag will be utilized for issues that have an external dependency before a fix can be applied.

#### Issue Management and Release Schedule relationships

* [ ] **_TODO: Clean up this section_**

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

* At minimum one major release will be scheduled each year, as feature requests and minor issues permit.
* At minimum one minor release will be scheduled each month, as major issues permit.
* A hotfix will be scheduled as early as team manager can estimate completion of fix. It will be up to the team manager to ensure enough resources will be assigned to the issue to resolve it quickly and efficiently.
  * A suggestion is for the team lead to manage a list of team members that can be pulled to address the issue.
