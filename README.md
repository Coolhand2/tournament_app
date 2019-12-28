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
git commit -m "Nightly release"
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

* See vpc-stack.yml

We're currently attempting to build a simple VPC with an onion of security layers.

### Subnet Design

There will be eventually four subnets, one for each AZ in us-west-2. Currently the stack is only listing 3.

**Public Subnets:**

* 10.0.0.0/24 (us-west-2a) _Name: Tournament Public A_
* 10.0.1.0/24 (us-west-2b) _Name: Tournament Public B_
* 10.0.2.0/24 (us-west-2c) _Name: Tournament Public C_
* 10.0.3.0/24 (us-west-2d) _Name: Tournament Public D_

**Private Subnets:**

* 10.1.0.0/16 (us-west-2a) _Name: Tournament Private A_
* 10.2.0.0/16 (us-west-2b) _Name: Tournament Private B_
* 10.3.0.0/16 (us-west-2c) _Name: Tournament Private C_
* 10.4.0.0/16 (us-west-2d) _Name: Tournament Private D_

### Security Layers

#### NACLs

##### Outbound Connections

We currently have no need to restrict outbound connections.

_We will be revisiting this to examine ports/protocols of outbound-container communication and lock it down._

* allow out 0.0.0.0/0:_All Ports_:_All Protocols_

##### Inbound Connections

The UI will be served from an S3 bucket, which removes our need to open up port 80/443 for the frontend. We will be requiring SSL Client-Cert verification for API calls however, which still requires us to open port 443. We also wish to ensure all public/private subnet is explicitly allowed through.

_We will be revisiting this to examine ports/protocols of inter-container communication and lock it down._

* 0.0.0.0/0:443:_TCP_
* 10.0.0.0/24:_All Ports_:_All Protocols_
* 10.0.1.0/24:_All Ports_:_All Protocols_
* 10.0.2.0/24:_All Ports_:_All Protocols_
* 10.0.3.0/24:_All Ports_:_All Protocols_
* 10.1.0.0/16:_All Ports_:_All Protocols_
* 10.2.0.0/16:_All Ports_:_All Protocols_
* 10.3.0.0/16:_All Ports_:_All Protocols_
* 10.4.0.0/16:_All Ports_:_All Protocols_

#### Security Groups

A design choice has to be made here. Do we want individual security groups for every instance we launch, or do we wish to build generic ones that can be applied in groups, based on functionality needs?

As of right now, allowing containers to communicate with eachother as necessary for heartbeat and others, we will have at least a SG that allows for that.

_We will be revisiting this to examine ports/protocols of inter-container communication, categorize them into Securiy Groups, and lock down the VPC._

* _All Protocols_:_All Ports_:_This Security Group_ Tournament Default Group
* _TCP_:_3306_:_10.0.0.0/8_ Potential Future MySQL only security group.
* _TCP_:_443_:_10.0.0.0/8_ Potential Future SSL only security group.
* _TCP_:_11211_:_10.0.0.0/8_ Potential Future Memcached only security group.

## Todo List

A list of things to finish. Not as good as a Kanban board but will suffice.

- [ ] VPC with Subnets **_In Progress_**
- [ ] Route53 info
- [ ] IAM Users Stack for future Database considerations
- [ ] buildspec.yml to run all the submodule stacks.
- [ ] Examine Outbound API ports and connections for NACLs.
- [ ] Examine inter-subnet communication usage for NACLs.
- [ ] Examine api container communications for SGs.
- [ ] New VPC for Testing? How do we version control that?

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
