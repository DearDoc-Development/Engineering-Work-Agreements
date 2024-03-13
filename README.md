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


#### Clean Code Philosophy

This section will explain the coding philosophy for the project.

The objective is to have a clean code that facilitates the reading and writing of the code used for this project. It is based on the book "Clean Code" by Robert C. Martin.

The main principles and rules are:

- ##### The Boy Scout Rule
    - "Always leave the campsite cleaner than you found it". 
    - When you work on a piece of software, if you find something that you can improve, always try to improve it. Like renaming a variable to be more understandable, or extract a feature from a reusable function.
    - Try to improve the envirorement.
- ##### Names With Meaning
    - Variable naming rules:
        - The names of the variables should always try to reveal the context.
        - The names of variables should use the [Camel case](https://developer.mozilla.org/en-US/docs/Glossary/Camel_case) naming convention.
        - The variables should be sustantives and explain the purpose.
        - Example:
            ```
            Incorrect:
                const a: number = 21;
                const fn: string= "Diego";
                const dad: boolean = true;

            Correct:
                const age: number = 21;
                const firstName: string = "Diego";
                const isDad: boolean = true;
            ```
    - Method or function naming rules:
        - The names of the Methods or functions must be verbs and must describe their function.
        - Example:
            ```
            Incorrect:
                function load(f: File){
                ...
                }
                function validate(a: number){
                ...
                }

            Correct:
                function loadFile(file: File){
                ...
                }
                function validateAge(age: number){
                ...
                }
            ```
    - Class naming rules:
        - Classes must be nouns or noun phrases. Avoid generic names like "data" or "loader".
        - Example:
            ```
            Incorrect:
                class Convert {
                ...
                }
                class Parse {
                ..
                }

            Correct:
                class ConvertPDF {
                ...
                }
                class XMLParse { 
                ...
                }
            ```
    - Consistency rules
        - All functionalities must follow the same chosen nomenclature.
        - Example:
            ```
            Incorrect:
                function getUser() {
                ...
                }
                function retrieveCat () {
                ...
                }
                function fetchCustomer () {
                ...
                }

            Correct:
                function getUser() {
                ...
                }
                function getCat () {
                ...
                }
                function getCustomer () {
                ...
                }
            ```
    - Other naming rules
        - They must be pronounceable names.
        - Avoid misinformation or assumptions.
        - Avoid encodings and prefixes as much as possible such as: nameTxT, varCounter, _customer, etc.
- ##### Functions
    - Functions should only perform one task.
        - A function should only do one thing and it should be able to do it well, it is always best to separate responsibilities and create a new component if we require another functionality.
        - Example:
            ```
            Incorrect:
                function validateAgeAndEmail(age: int, email: string){
                    if(age < 18){
                        ...
                    }
                    if(!email.includes("@")) {
                        ...
                    }
                    ...
                }

            Correct:
                function validateAge(age: int){
                    if(age < 18){
                        ...
                    }
                    ...
                    }
                    function validateEmail(email: string){
                    if(!email.includes("@")) {
                        ...
                    }
                    ...
                }
            ```
            `Note`: A function with more than twenty lines is probably doing several things at once.
- ##### Comments
    - Citing Robert C. Martin in the book:
        _"-The proper use of comments is to compensate for our failure to express ourself in code."_
    - The use of comments is not advisable. Only when providing value to the context.
    - Avoid comments to mark a section inside the code.
        - Example:
            ```
                /* ------------ global variables ------------- */
                const mongoUri = process.env.MONGO_URI;
                .....

                /* ------------ classes ----------------------- */
                class UserContract {
                ...
                }
            ``` 
    - Delete commented code. With the versions control used, like Git, we can recover previously written code, making commented code useless.
        - Example:
            ```
                function getUserById(id: string){
                    ...
                }

                /*   
                const userContract = new UserContract();
                if(){
                    ....
                }
                */
            ``` 
### Modifications

This document is a living document and will be updated as the team grows and the company evolves. If you have any suggestions or want to propose a change, please open a pull request and submit it.