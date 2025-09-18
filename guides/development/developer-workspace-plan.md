# Developer Workspace Plan

This document tracks the plan for the Serverless microservices developer workspace and umbrella project (starting with `deals-ms`). It will evolve iteratively as we discuss and finalize decisions.

## Literature Review (in progress)

- Read the relevant chapters and subsections of “Microservices Up and Running”.
- While reviewing, keep our Serverless (AWS Lambda + CDK) architecture in mind.
- Adapt the book’s recommendations pragmatically to our context.
- Capture deltas/decisions and action items in this document.

Guidance sources to consider alongside the book:

- `docs/guides/development/localstack-guide.md`
- `docs/guides/development/cicd-guide-v2.md`
- Design methodology: `docs/guides/design-principles/design-and-development-methodology.md`

> You (Nick) will review the chapters/subsections and add notes here; we’ll iterate until the plan is finalized, then implement.

### CHAPTER 8 - Developer Workspace

#### 10 Workspace Guidelines for a Superior Developer Experience

1. Make Docker the only dependency: This needs to be considered in the context of localstack and how I have set it up i.e. using docker desktop with the docker desktop localstack extension
2. Remote or local should not matter: I'm not so sure about this.
3. Ensure a heterogeneous-ready workspace: This is not applicable to our setup as we are specifically using AWS CDK with TypeScript.
4. Running a single microservice and/or a subsystem of several ones should be equally easy: I think the umbrella project will take of this. It is meant to allow a developer to download, work on, and deploy any of the microservices that are part of the overall app.
5. Run databases locally, if possible: Given our tech stack, we will be running any of the AWS DB services on localstack.
6. Implement containerization guidelines: This is dependent on the way we will be running the microservices in localstack. I will return to this point later to see if anything is applicable at that time.
7. Establish rules for painless database migrations: Not applicable at the moment.
8. Determine a pragmatic automated testing practice: I have implemented any tests yet. I will return to this once I have.
9. Branching and merging: We have addressed this with guidance in `guides/development/cicd-guide-v2.md`. That may need a re-look at a later stage to see if it is up-to-date.
10. Common targets should be codified in a makefile: This point recommends defining and implementing the following standard targets for the microservice makefiles. This definitely needs to a look-into for possible implementation, depending upon needs of our architecture.

#### Setting Up a Containerized Environment Locally

The containerized environment for our given tech stack requires Docker and LocalStack. I am running Docker via Docker Desktop and have the LocalStack extension installed. I'm not sure if other ways of accessing LocalStack (I think there are about four of them) are required; this can be looked into later.

### CHAPTER 9 - Developing Microservices

#### Implementing Code for a Microservice

This section recommends starting microservices with reusable templates. When the time comes, I plan to make a copy of the `deals-ms` microservice project and modify it into a reusable GitHub template project.

The section also mentions that it assumes we have a working Docker installation and the GNU Make; there are no other expectations. Again, we are using Docker Desktop with the LocalStack extension to deploy to and run the microservices' resources on LocalStack. Whether we choose follow what was suggested in the 10 workspace guidelines about using common make targets is something we need to think about and whether to implement it, based on pros and cons of doing so. Remember, the ability to run Make commands would mean using Windows Subsystem for Linux (WSL) on Windows, which I am doing; we need to consider what this means for other developers using other host operating systems.

##### Health Checks

This sub-section mentions "To manage the life cycle of the containers that the app will be deployed into, most container-management solutions (e.g., Kubernetes, which we will use later in this book) need a service to expose a health endpoint". What does this mean for our tech stack? I'm not sure if this is applicable to our serverless microservices architecture. Perhaps there is an equivalent concept in serverless microservices architecture that we have implemented (I am assuming this is not only for the local environment but for aws deployment).

#### Hooking Services Up with an Umbrella Project

For now, we are focused on the correct setup of the local environment for deployment of the `deals-ms` microservice, as it exists now, to LocalStack and making sure that other microservices can also be seamlessly deployed to it. All of this has to be done within the umbrella project.

## Notes



## Phases (draft)

### Phase 1: Developer workspace for LocalStack (deals-ms)

- Add Makefile/npm scripts for common dev flows.
- Seed script and local data lifecycle.
- Quickstart docs in `README.md`.

### Phase 2: Umbrella project for LocalStack (deals-ms)

- Compose LocalStack + `deals-ms`.
- Deploy to LocalStack (`cdklocal`).
- Verify API health endpoint is reachable.

### Phase 3: Web integration

- Route the `web/` app to the local API endpoint.
- E2E smoke: create deal.

### Phase 4: CI parity

- Ensure local commands mirror CI steps per `cicd-guide-v2.md`.
- Trunk-based flows and rollback hooks.

### Phase 5: Multi-service umbrella

- Add `users-ms` (stub or real) and shared data seeding.
- Service discovery patterns locally.

### Phase 6: Template extraction

- Extract `deals-ms` into a GitHub template project.
- Bootstrap automation for new services.

## Open Questions / To Refine

- Which book subsections most directly apply to serverless vs container orchestration guidance?
- Local service discovery conventions and API gateway emulation boundaries.
- Contract-driven testing and consumer/provider test placement for our repos.

## Next Steps

1. Nick: review the specified chapters/subsections and add notes under “Literature Review”.
2. We will convert notes into concrete decisions and update each Phase.
3. Once finalized, proceed with implementation tasks linked to this plan.
