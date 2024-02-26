## Engineering Work Agreements

The purpose of this repository is to provide a set of guidelines and best practices for the development of software projects at [DearDoc](https://getdeardoc.com/). This document is a living document and will be updated as the team grows and the company evolves.

### Table of Contents

- New projects (TBD)
- Source Control 
- Project Documentation (TBD)
- Deployment (TBD)

#### Source Control
This chapter will describe the best practices for source control, including branching, commits, and pull requests.

All the code should be stored in a source control system. We will use [Github](github.com) as our source control system.
Every project should be created under our company Github account, [DearDoc](https://github.com/DearDoc-Development). 


##### Branching

When working in a project it is important to follow a branching strategy that allows for a clean and organized codebase. 
We will use [Github Flow](https://githubflow.github.io/) as our branching strategy. This strategy is simple and effective, and it is based on the following principles:

- Anything in the `main` branch is always deployable.
- To work on something new, create a descriptively named branch off of `main` (ie: `new-oauth2-scopes`).
- Commit to that branch locally and regularly push your work to the same named branch on the server.
- When you need feedback or help, or you think the branch is ready for merging, open a pull request.
- After someone else has reviewed and signed off on the feature, you can merge it into `main`.
- Once it is merged and pushed to `main`, you can and should deploy immediately.

##### Commits

When pushing changes to a repository we should follow some guidelines on how to write the commit messages and the granularity of them. 

Regarding the periodicity of commits, we should commit often and in small chunks. This will allow for a cleaner history and will make it easier to understand the changes made to the codebase.

Regarding the commit messages, we should follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification. In this way we should have a semantic way of writing commit messages that will allow for automatic changelog generation and other benefits.
The structure of a commit shoulw have this structure:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

The types of commits are:
- `BREAKING CHANGE`: a commit that has a footer `BREAKING CHANGE: <description>` will be part of the next major release.
- `feat`: a new feature for the user, not a new feature for build script
- `fix`: a bug fix for the user, not a fix to a build script
- `docs`: changes to the documentation
- `chore`: others changes that don't modify src or test files

Example:
```
feat: new API endpoint to get user data, profile picture and email

Includes a database migration with a new model in the backend to handle the user data and a new endpoint to get the user data, profile picture and email.
```


### Modifications

This document is a living document and will be updated as the team grows and the company evolves. If you have any suggestions or want to propose a change, please open a pull request and submit it.