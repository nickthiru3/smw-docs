# SMW Project Plan

## Overview
- **Mission** Establish a mobile-first platform that promotes circular economy behaviours by connecting users with repair, refill, recycling, and donation services.
- **Primary outcomes** Reduce waste sent to landfills, grow a community of sustainability-minded users, and create recurring revenue through business partnerships and listings.
- **Current status** Discovery complete; business plan captured in `docs/project/project-brief.pdf` and summarized in `docs/project/project-brief-summary.html`.
- **Product focus** Operate a B2B2C, directory-first marketplace where circular economy providers meet sustainability-minded consumers, supported by educational content.

## Platform Positioning
- **Core offering** Location-based directory of circular economy businesses with subscription-backed listing tiers and business analytics.
- **Content role** Recycling guides and editorial content function as the acquisition and engagement engine feeding directory usage.
- **User segments** Consumers seeking services, business owners managing listings, and platform operators handling verification and monetization.
- **Architecture baseline** Apply the serverless microservices and SvelteKit-embedded BFF approach documented in [`docs/guides/design-principles/serverless-microservices-architecture-v3.md`](../guides/design-principles/serverless-microservices-architecture-v3.md).
- **Delivery surfaces** Responsive web application (PWA-capable) for consumer and operator workflows, with optional future native wrapper if usage warrants.

## Success Metrics
- **User adoption** Active users, retention, and depth of engagement with listings and educational content.
- **Impact** Reported diversion of waste (items repaired, refilled, recycled, donated) and participation in sustainable activities.
- **Revenue** Business subscriptions, sponsored content, and partnership-generated income.
- **Geographic reach** Expansion of verified listings across target regions and readiness for multi-country rollout.

## Stakeholders & Roles
- **Product lead** TBD
- **Engineering lead** TBD
- **Design/UX** TBD
- **Content & Partnerships** TBD
- **Data & Analytics** TBD
- **Advisors** Reference stakeholders noted in project brief; confirm availability.

## Delivery Methodology
- **Approach** Iterative Agile (two-week sprints) with quarterly planning to align on phase objectives.
- **Ceremonies** Sprint planning, daily stand-ups, sprint review/demo, retrospective.
- **Artefacts** Product backlog, sprint backlog, roadmap, release notes, metrics dashboard.
- **Tools** Issue tracker (TBD), documentation within `docs/`, shared roadmap board, analytics instrumentation; reference methodology in [`docs/guides/design-principles/design-and-development-methodology.md`](../guides/design-principles/design-and-development-methodology.md).

## Roadmap & Release Plan

### Phase 1a – Directory MVP (Months 1-4)
- **Focus** Launch the location-based marketplace for Release 1 categories (Repair, Refill/Grocery, Recycling, Donate) with business onboarding.
- **Key capabilities**
  - Geospatial search with distance filters, category/waste/product faceting, and result weighting.
  - Map view with clustering, detailed listing pages, and favorites/history for authenticated users.
  - Business listing management portal with tiered subscription packages, media uploads, and verification workflows delivered via SvelteKit BFF routes.
  - Payments integration for business subscriptions and internal verification tooling backed by microservices created from the template repository.
  - Responsive/PWA experience optimized for desktop and mobile usage, including map deep-linking.
- **Dependencies** Curated datasets for core categories, geocoding services, payment gateway, SSM verification integration roadmap, SST infrastructure and environment setup for BFF + microservices deployments, PWA manifest/service worker framework.
- **Assumption** Merchant listings auto-approve at MVP to accelerate directory delivery; manual review tooling deferred until qualification criteria are defined.

### Phase 1b – Content Hook (Months 4-6)
- **Focus** Deploy the educational and SEO-driven experiences that funnel users into the directory.
- **Key capabilities**
  - Recycling information hub with cross-links to relevant directory listings.
  - Blog and editorial CMS workflow (headless WordPress or equivalent) for sustainability content.
  - User submission form for listings with admin approval and content moderation tooling.
  - PWA enhancements (offline caching for core pages, install prompts) to support repeat visits.
- **Dependencies** CMS implementation, content operations playbooks, moderation staffing plan, PWA asset strategy (service worker caching rules, manifest metadata).

### Phase 2 – Engagement & Revenue Expansion (Months 7-12)
- **Focus** Grow business value through additional categories, engagement loops, and monetization features.
- **Key capabilities**
  - New categories: Cafes & Restaurants, Green Retailers, Local Markets, Second-hand.
  - Reviews and ratings, reward points, badges, and banner advertising placements.
  - Business analytics dashboards (traffic, conversions) and enhanced promotion packages.
  - Events and marketplace listings surfaced within the directory experience.
- **Dependencies** Verification operations scaling, reward catalogue design, analytics instrumentation strategy.

### Phase 3 – Integrated Ecosystem (Months 13-18)
- **Focus** Connect all services with community, rentals, and advanced tracking.
- **Key capabilities**
  - Additional categories: Rentals, Water refill, Subscription services, Green groups/events.
  - Community chat and social sharing features.
  - Carbon footprint calculator and activity logging.
  - Advanced analytics for business partners and monetization overlays (ads, pay-per-click).
- **Dependencies** Scalable realtime infrastructure, carbon data models, event partnerships.

## Workstreams
- **Product & UX** Journey mapping, information architecture, design system, usability testing.
- **Engineering** Mobile/web clients, APIs, integrations, testing, deployment pipelines.
- **Content & Partnerships** Data sourcing for listings, recycling instructions, partner onboarding, verification SOPs.
- **Marketing & Growth** Acquisition strategy, SEO, community campaigns, engagement analytics.
- **Operations & Support** Customer support processes, moderation, compliance, localization planning.

## Technical Stack (Initial Proposal)
- **Client** SvelteKit responsive web application with PWA capabilities serving consumers, business owners, and operators across desktop and mobile browsers.
- **BFF & API Layer** SvelteKit application deployed with SST on AWS Lambda, exposing BFF endpoints consumed by mobile and web clients; handles orchestration, authentication, and aggregation per the architecture guide.
- **Backend Microservices** AWS Lambda-based services created from the [microservice template](https://github.com/nickthiru3/microservice-template), defined with AWS CDK, owning business capabilities, DynamoDB tables, and event flows (SNS/SQS, Step Functions, Parameter Store discovery).
- **Content Platform** Headless WordPress (or current CMS stack) consumed via BFF routes to deliver recycling guides and editorial content.
- **APIs & Integrations** Maps (Google Maps/Waze or Amazon Location Service), OAuth providers (Google, Facebook, Apple), payment gateway, multimedia hosting (YouTube), analytics (Mixpanel/Amplitude).
- **Data Storage** Service-owned DynamoDB single-table designs supplemented with OpenSearch/Elasticsearch or Amazon Location Service for geospatial querying, plus S3 for media assets and CMS storage.
- **Infrastructure** Cloud deployment (AWS/Azure/GCP) with CI/CD pipeline, environment segregation (dev/stg/prod), monitoring and logging stack.
- **Service template** Backend microservices to be bootstrapped from the template repository, following the "microservice construct" pattern in the architecture guide.
- **Mobile packaging (future option)** Evaluate lightweight wrappers (e.g., Capacitor, Expo) post-launch if app-store distribution becomes strategic.

## Data & Content Strategy
- **Listings** Blend of curated datasets, manual research, and user submissions with verification workflows.
- **Recycling knowledge base** Categorized material guides, disposal instructions, alternative usage suggestions, and cross-links to services.
- **Localization** Prepare taxonomy and copy for multilingual support and region-specific regulations.

## Risks & Mitigations
- **Data freshness** Risk of outdated listings; mitigate via verification cadence, user feedback, and partner SLAs.
- **Operational load** Verification and content moderation overhead; design tooling and staffing plan early.
- **Scope creep** Extensive feature set per phase; enforce prioritization and MVP guardrails.
- **Technical debt** Mixing legacy CMS with modern app stack; evaluate headless architecture to decouple concerns.
- **Revenue uncertainty** Validate pricing and conversion before investing heavily in monetization features.

## Dependencies & Assumptions
- **Partnerships** Access to councils, NGOs, and retailers for accurate listings.
- **Compliance** Alignment with regional data privacy and consumer protection laws.
- **Team availability** Core roles staffed before development kickoff.
- **Budget** Funding secured for 18-month roadmap including marketing and operations.

## Progress Tracking
| Area | Owner | Status | Notes |
| --- | --- | --- | --- |
| Product discovery | TBD | In progress | Validate MVP scope against market research.
| Data sourcing | TBD | Not started | Identify initial datasets for Phase 1 categories.
| Technical architecture | TBD | Not started | Decide on backend platform and hosting strategy.
| Design system | TBD | Not started | Kick off IA and low-fidelity wireframes.
| Marketing plan | TBD | Not started | Outline launch and community activation tactics.

## Decision Log
| Date | Decision | Context | Owner |
| --- | --- | --- | --- |
| TBD | Select backend platform (WordPress vs headless CMS) | Evaluate content workflow, scalability, integration effort. | TBD |
| TBD | Confirm mobile app framework | Balance time-to-market, team expertise, performance requirements. | TBD |

## Open Questions
- **What** datasets are currently available for core categories and who owns them?
- **Which** delivery platforms (native mobile, PWA) are prioritized for MVP launch?
- **How** will verification, moderation, and support be staffed and automated?
- **What** KPIs will govern go/no-go for expansion into Phase 2 categories?
- **Which** revenue experiments will run during MVP to validate pricing and partner interest?
- **When** should we refresh or update guidance in [`serverless-microservices-architecture-v3.md`](../guides/design-principles/serverless-microservices-architecture-v3.md) and [`design-and-development-methodology.md`](../guides/design-principles/design-and-development-methodology.md) to match SMW-specific decisions?
- **Who** will define the circular-economy qualification checklist for merchants, and what evidence will be required before enabling manual approvals?

## Reference Materials
- **Architecture guide** [`docs/guides/design-principles/serverless-microservices-architecture-v3.md`](../guides/design-principles/serverless-microservices-architecture-v3.md)
- **Delivery methodology** [`docs/guides/design-principles/design-and-development-methodology.md`](../guides/design-principles/design-and-development-methodology.md)
- **Service template repository** [nickthiru3/microservice-template](https://github.com/nickthiru3/microservice-template)
