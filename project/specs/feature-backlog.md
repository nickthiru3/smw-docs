# SMW Feature Backlog

This backlog captures candidate jobs/features to flow through the methodology described in `docs/guides/design-principles/design-and-development-methodology.md`. Each bullet links to the primary outcome we need to elaborate into job cards, sequence diagrams, contracts, and implementations.

> **MVP assumption:** Merchant listings are auto-approved during Phase 1a to unblock directory delivery. Manual review tooling will be introduced once stakeholders define circular-economy qualification criteria.

## Consumer Jobs

### Phase 1a – Directory MVP

- **Search by category (MVP)** Users want to combine category, waste type, product, and distance filters so they can quickly locate nearby circular services. [docs/project/specs/jobs/consumers/search-by-category/job-card.md]
- **Map exploration** Users want an interactive map with clustering so they can visualise providers relative to their location and launch navigation in Google Maps/Waze.
- **Listing details** Users want rich business pages with operating info, accepted materials, media, and call-to-action links so they can decide to engage a provider.
- **Account & favorites** Users want to sign in and save providers so they can revisit preferred listings and track activity history.

### Phase 1b – Content Hook

- **Enhanced search filtering & sorting** Users want sub-category drill-downs, availability toggles, and result ordering so they can narrow results without leaving the listings view.
- **Recycling knowledge base** Users want guidance on how to recycle specific materials so they can act correctly and discover matching providers.
- **PWA enhancements** Returning users want offline access to key pages and install prompts so they can treat SMW like a native app on mobile devices.

### Phase 2 – Engagement & Revenue Expansion

- **Personalised search & saved journeys** Users want to save favourite filter combinations, see trending categories, and receive notifications so they can quickly repeat common searches.
- **Extended categories** Users want to browse cafes, green retailers, markets, and second-hand stores so they can broaden sustainable choices.
- **Reviews & ratings** Users want to share experiences so they can help the community evaluate providers. [docs/project/specs/jobs/consumers/review-rate-merchants/job-card.md]
- **Rewards & badges** Users want recognition and incentives so they stay motivated to take sustainable actions.
- **Events marketplace** Users want to discover green events and workshops so they can participate in community activities.

### Phase 3 – Integrated Ecosystem

- **Multi-stop planning & intelligent recommendations** Users want route planning, cross-category suggestions, and AI-powered guidance so they can complete circular errands efficiently.
- **Rental services** Users want to rent items (tools, furniture, equipment) so they can avoid one-time purchases.
- **Water refill network** Users want to find refill stations so they can reduce single-use bottles.
- **Subscription services** Users want recurring refill or take-back programs so they can simplify sustainable habits.
- **Community engagement** Users want chat/groups to share tips and organise actions so they can build local momentum.
- **Carbon tracking** Users want to log activities and view estimated impact so they can quantify their progress.

## Merchant Jobs

### Phase 1a – Directory MVP

- **Business onboarding** Service owners want to create and manage listings (info, media, categories) so they can reach sustainability-focused customers.
- **Subscription & billing** Business owners want to purchase tiered packages so they can unlock promotions and analytics.

### Phase 2 – Engagement & Revenue Expansion

- **Banner promotions** Business owners want to run advertisements so they can increase visibility for offers.
- **Business analytics** Business owners want dashboards showing traffic and engagement so they can measure ROI.
- **Advanced monetization** Business owners want premium placements (pay-per-click, highlighted listings) so they can amplify reach during campaigns.

### Phase 3 – Integrated Ecosystem

- **Marketplace enablement** Business owners want to list rental, subscription, and refill services so they can monetise circular offerings within SMW.

## Platform Admin & Operations Jobs

### Phase 1a – Directory MVP

- **Verification workflow** Operations staff want to review and approve new or edited listings so the directory remains trustworthy. _Dependency: requires stakeholder-defined qualification criteria; until then, rely on auto-approval and monitoring._
- **Dataset ingestion** Platform operators want to seed the directory from curated datasets so launch coverage meets baseline expectations.
- **Analytics foundation** Platform operators want tracking for searches, views, and conversions so they can validate demand and inform partners.
- **Qualification criteria definition** Platform admins want to document and enforce circular-economy benchmarks for listings so only compliant businesses appear on SMW.

### Phase 1b – Content Hook

- **Editorial publishing** Content teams want to post blogs, news, and videos so they can drive organic traffic and engagement.
- **Listing self-submission** Community members want to suggest new providers so the directory stays fresh with minimal operational overhead.

### Phase 2 – Engagement & Revenue Expansion

- **Reward catalogue management** Platform admins want to configure incentives so they can support loyalty and partnership deals.

### Phase 3 – Integrated Ecosystem

- **Community moderation** Platform admins want to oversee chat/groups so they can maintain safe and constructive discussions.
- **Impact reporting ops** Platform staff want to aggregate carbon-tracking data so they can publish impact metrics to stakeholders.

## Backlog Management Notes

- For each bullet, create a job card under `docs/specs/jobs/` (new location) or reuse `docs/specs-OLD/jobs/` as reference.
- Follow the methodology steps: job statement → sequence diagram → API contract → data alignment → implementation → testing.
- Use this document as the single text-based index that AI collaborators and team members can parse easily.
