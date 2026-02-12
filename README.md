# Engineering Work Agreements

The purpose of this repository is to provide a set of guidelines and best practices for the development of software projects at [DearDoc](https://getdeardoc.com/).

This document is a living document and will be updated as the team grows and the company evolves.

## Table of Contents

- [Engineering Work Agreements](#engineering-work-agreements)
  - [Table of Contents](#table-of-contents)
  - [Project Management](#project-management)
    - [Tools](#tools)
  - [People](#people)
  - [Source Control](#source-control)
    - [Branching](#branching)
    - [Commits](#commits)
    - [PR and Deploy Mandatory Steps](#pr-and-deploy-mandatory-steps)
      - [Team commitments](#team-commitments)
      - [Versioning Definitions](#versioning-definitions)
      - [Mandatory steps](#mandatory-steps)
  - [Project Documentation](#project-documentation)
  - [Modifications](#modifications)

## Project Management

This section outlines key practices for effective project management at DearDoc. We predominantly adopt [Agile methodologies](https://agilemanifesto.org/) to manage our work, adapting our approach based on team preferences and project demands.

At DearDoc, we primarily utilize two Agile methodologies: [Scrum](https://www.scrum.org/) and [Kanban](https://www.atlassian.com/agile/kanban). The choice between Scrum and Kanban depends on the specific requirements of the project as well as team dynamics.

**Key Differences**:

- **Scrum:** Operates on fixed-duration cycles known as sprints, with defined roles (Product Owner, Scrum Master, Development Team) and ceremonies (Sprint Planning, Daily Standup, Sprint Review, Sprint Retrospective). Scrum focuses on regular cadences and defined deliverables.
- **Kanban:** Focuses on continuous workflow with no set time frames for iterations. It emphasizes ongoing delivery without predefined roles or formal ceremonies.

So, while Scrum is more structured and prescriptive, Kanban is more flexible and adaptive, useful for a continuous task such as maintenance or projects developed by individual teams. The choice between the two depends on the project requirements and team dynamics.

### Tools

- **[Slack](https://slack.com/):** Our primary tool for team communication.
- **[Monday.com](https://monday.com/):** Utilized for managing projects and facilitating collaboration. It allows teams to create task boards, assign responsibilities, track progress, and integrate with other tools, effectively supporting both Scrum and Kanban methodologies.
- **[Google Drive](https://drive.google.com/):** Used for storing and sharing project documentation, reports, and other files.

Monday.com is integral to project management at our company, providing a customizable platform suitable for diverse project needs. So, everything regarding the project management should be handled through Monday.com.

## People

Agile methodologies guide us in defining roles and responsibilities within project teams, emphasizing clarity but not hierarchy.

Regarding people management, every team member should have a touching point with another team member in the company, usually a team leader or a project manager. Some of the shared responsibilities are:

- **Role Clarity:** Understanding individual roles within the team.
- **Alignment with Company Vision:** Awareness of the company’s vision and mission.
- **Responsibilities:** Clear definition of what is expected from each team member.
- **Career Growth:** Opportunities for professional development including training and career paths.
- **Goal-Oriented Performance:** Focus on achieving specific, measurable goals.
- **Feedback and Evaluation:** Regular coaching and feedback sessions to foster growth and improvement.
- **Holidays and Time Off:** Clear guidelines on holidays, time off, and work-life balance.

**Development is fostered through**:

- **Regular 1-on-1s:** Scheduled weekly and quarterly, these sessions are dedicated to personal development and are distinct from regular project management activities.

**Tools**:

Our tool for handling personal development is [Ninety](https://www.ninety.io/). Using the [EOS](https://www.eosworldwide.com/) methodology, we run meetings with a clear agenda and goals. Manage Rocks and To-Dos, and have a clear view of the company's vision and mission. This tool is also helpful for evaluations, with tools such as [People Analyzer](https://www.eosworldwide.com/people).

## Source Control

This chapter will describe the best practices for source control, including branching, commits, and pull requests.

All the code should be stored in a source control system. We will use [Github](github.com) as our source control system.

Every project should be created under our company Github account, [DearDoc](https://github.com/DearDoc-Development).

### Branching

When working in a project it is important to follow a branching strategy that allows for a clean and organized codebase.
We will use [Github Flow](https://githubflow.github.io/) as our branching strategy. This strategy is simple and effective, and it is based on the following principles:

- Anything in the `main` branch is always deployable.
- To work on something new, create a descriptively named branch off of `main` (ie: `new-oauth2-scopes`).
- Commit to that branch locally and regularly push your work to the same named branch on the server.
- When you need feedback or help, or you think the branch is ready for merging, open a pull request.
- `Staging Environment`: If a Staging environment exists, it must be used for verification before merging to `main` (Production) following the instructions bellow.
- `No Staging Environment`: If no Staging environment exists, all mandatory checks and reviews must be performed before merging directly to `main`.
- After someone else has reviewed and signed off on the feature, you can merge it into `main`.
- Once it is merged and pushed to `main`, you can and should deploy immediately. 
- After checking the `main` deploy delete the `feature/fix branch ot keep the repo clean

### Commits

When pushing changes to a repository we should follow some guidelines on how to write the commit messages and the granularity of them.

Regarding the periodicity of commits, we should commit often and in small chunks. This will allow for a cleaner history and will make it easier to understand the changes made to the codebase.

Regarding the commit messages, we should follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification. In this way we should have a semantic way of writing commit messages that will allow for automatic changelog generation and other benefits.

The structure of a commit should have this structure:

```md
<type>[optional scope]: <description>

[optional body]
[optional Monday Ticket if related with any task or support issue - Format: "Item Name #123456" or just "#123456"]

[optional footer(s)]
```

The types of commits are:

- `BREAKING CHANGE`: a commit that has a footer `BREAKING CHANGE: <description>` will be part of the next major release.
- `feat`: a new feature for the user, not a new feature for build script
- `fix`: a bug fix for the user, not a fix to a build script
- `docs`: changes to the documentation
- `chore`: others changes that don't modify src or test files

Example:

```md
feat: new API endpoint to get user data, profile picture and email

Includes a database migration with a new model in the backend to handle the user data and a new endpoint to get the user data, profile picture and email.
```

### PR and Deploy Mandatory Steps

#### Team commitments
- `24h review`: if one is mentioned as a reviwer then in less than 24 hours he must respond or designate another person for the job.
- `Monday link`: If the PR is related with a ticket in Monday Dev platform the number of the item must be writen in your GitHub PR title or description (check commit example).

#### Versioning Definitions
- `Fix/small PR`: Treated as a minor version.
- `Feature`: Treated as a major version.

#### Mandatory steps
1. **Fix** - `fix/docs/chore`
     + Step 1: Localhost - Develop and test changes locally.
     + Step 2: PR to Staging
       + **Required Reviews:** 1 Peer Reviewer + Copilot Review.
       + **Required Checks:** CI/CD (GitHub Action checks) must pass.
     + Step 3: PR to Production (main)
       + **Required Reviews:** Copilot Review.
       + **Required Checks:** CI/CD (GitHub Action checks) must pass.

2. **Small PRs group (Minor Version)** - `feat/s` (small)
      + Step 1: Localhost - Develop and test changes locally.
      + Step 2: PR to Staging
        + **Required Reviews:** 1 Peer Reviewer + Copilot Review + QA Reviewer.
        + **Required Checks:** CI/CD (GitHub Action checks) must pass.
      + Step 3: PR to Production (main)
        + **Required Reviews:** 1 Peer Reviewer + Copilot Review.
        + **Required Checks:** CI/CD (GitHub Action checks) must pass.

3. **Feature (Major Version)** `feat/BREAKING CHANGE`
      + Step 1: Localhost - Develop and test changes locally.
      + Step 2: PR to Staging
        + **Required Reviews:** 1 Peer Reviewer + Copilot Review + QA Reviewer.
        + **Required Checks:** CI/CD (GitHub Action checks) must pass.
      + Step 3: PR to Production (main)
        + **Tagging: Tag the Feature code.**
        + **Required Reviews:** Copilot Review.
        + **Required Checks:** CI/CD (GitHub Action checks) must pass.


## Project Documentation

To maintain a standard structure while developing new projects, it is recommended to use the [Technical Documentation](./TechnicalDocumentation.md) file as a template for your project's README.md file.

## Modifications

This document and every other file in this repository are living documents and will be updated as the team grows and the company evolves.

If you have any suggestions or want to propose a change, please open a pull request and submit it.
