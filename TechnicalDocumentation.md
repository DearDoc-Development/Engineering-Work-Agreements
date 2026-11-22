# Project Title

One sentence or two explaining what the project does or what it is for.

## Table of Contents

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Tests](#running-the-tests)
  - [Error and Monitoring](#error-and-monitoring)
  - [Deployment](#deployment)
- [Usage](#usage)
  - [Architecture](#architecture)
  - [Built With](#built-with)
  - [API Endpoints](#api-endpoints)
  - [Logging](#logging)
- [Contributing](#contributing)
  - [Versioning](#versioning)
  - [Authors](#authors)

## Introduction

A longer description about the project. What problem does it solve? Who is the target audience?

## Getting Started

These instructions will get a copy of the project up and running on your local machine for development and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

Tools required in your local machine before starting to work on the project.

### Installation

Please provide detailed steps on how to install and set up the project and its dependencies.

#### **Running the Tests**

Elaborate on how to run the automatic tests for this system:

- Specify any necessary setup or data required before testing. Use synthetic data only, never real patient data.
- Mention different types of testing that are available (unit tests, integration tests, etc.), and how to initiate each.
- List the CI checks that run on every PR: typecheck, lint, tests, build, and secret scan.

#### **Error and Monitoring**

Explain how errors are handled in the project, including logging and monitoring:

- Describe the logging system used, and how to access logs.
- Mention any monitoring tools or services that are integrated into the project, and how to use them.

#### **Deployment**

Describe the CI pipeline. There are no manual production steps.

- Merging to `staging` deploys to staging. Merging the release PR from `staging` to `main` deploys to production.
- Each production deploy is tagged `vYYYY.MM.DD.N`.
- To roll back, run the `rollback` workflow with the previous version tag (being built; until it exists, the Release Captain decides how to roll back).
- Cover required environment settings, configuration files, or dependencies the pipeline needs. Never write secrets here.

See the [release protocol](https://github.com/DearDoc-Development/Engineering-Work-Agreements#release-protocol).

## Usage

Provide step-by-step examples of how to use the software, highlighting common use cases and configurations. This could include simple code snippets or commands the user can execute.

### **Architecture**

Describe the logic architecture of the project and if possible, a diagram showing the different components and how they interact. Recommended if the project has more than two agents (a simple server-client project has two agents).
It Also includes a brief explanation of the design patterns used in the project. As well as communication patterns.
If the project is heavily based on Cloud Architecture, include the services used and how they interact.
Update this section in the same PR when a change alters the architecture.

### API Endpoints

Include the API Documentation if the project has less than five endpoints, otherwise, link to an external documentation. Preferably Swagger.

#### GET `/healthcheck`

A brief description of the endpoint.

Please also include a sample `request` and `response` if possible.

##### **Response**

```json
{
  "ok": true
}
```

### **Logging**

Explain how logging is handled in the project.

The projects should use a structured logging system, such as [pino](!https://github.com/pinojs/pino), to log events in a structured format. This allows for easier querying and analysis of logs. Describe the transport mechanism used for logs (e.g., console, file, third-party service) and the log levels used in the project (e.g., debug, info, error). Also, include instructions on how to use the logger.
If logs are sent to a centralized logging system, provide information on how to access them.

### **Built With**

List major frameworks, libraries, or tools that were used to create the project:

- Hyperlink the tool or framework to its official page to make it easier for new users to find the resources they need quickly.

If the project is a Node.js based project, a .nvmrc file with the version of node used in the project should be attached to the root folder of the project. Documentation on how to use [.nvmrc](https://github.com/nvm-sh/nvm?tab=readme-ov-file#nvmrc)

## **Contributing**

Provide guidelines on how prospective contributors can participate in the project:

- Link to the [Contributor Guide](https://github.com/DearDoc-Development/Engineering-Work-Agreements/blob/main/ContributorGuide.md) and the [release protocol](https://github.com/DearDoc-Development/Engineering-Work-Agreements#release-protocol). Add a CONTRIBUTING.md only for rules specific to this project.

### **Versioning**

Information on the versioning scheme used for the project, advice on how to access different versions, or how to see the change log. This could include:

- Each production deploy is tagged `vYYYY.MM.DD.N` (date, then the number of the deploy that day).
- Instructions or links on how to use Git tags to find specific releases or the change log.

### **Authors**

- List of people who have contributed significantly to the project.
- You might include their contact information, GitHub profiles, or other relevant links.
