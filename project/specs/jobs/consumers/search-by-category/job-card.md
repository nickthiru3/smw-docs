# Job Story: Search by Category

**Actor**: Consumer

**Job Story**:  
When a consumer needs to find circular-economy services nearby,  
I want to search by combining category, waste type, product, and distance filters,  
so I can quickly locate relevant providers that meet my specific needs.

**Priority**: High

**Estimated Complexity**: Large

**Acceptance Criteria**:

- [ ] Users can filter listings by Release 1 categories (Repair, Refill/Grocery, Recycling, Donate) and see matching results
- [ ] Users can combine category, waste type, product tags, and distance filters in a single query
- [ ] Distance filtering offers preset radii (e.g., 5km, 10km, 25km) based on user location
- [ ] Results show key listing fields: name, category tags, distance, verification badge (if available)
- [ ] Selecting a result opens the listing detail view (handled by separate job) without losing current filters
- [ ] Search supports both map and list presentations, kept in sync when filters change

---

## Extended Requirements

### Business Rules

### Implementation Notes & Assumptions

- MVP uses auto-approved listings; manual verification workflow will arrive later.
- Initial implementation uses a mocked merchant directory (flat file or in-memory dataset). Future iterations will integrate with the Users service (Cognito + DynamoDB) and/or dedicated search indices.
- For early iterations, assume location permissions are granted; fallback UX needed for manual location entry.
- Requires baseline taxonomy for categories, waste types, and product tags sourced during dataset ingestion.
- Backend ownership: `Merchants Search Service` (`merchants-search-service/`) exposes `/search/merchants` per the OpenAPI contract.

### Phase Scope

- **Phase 1a (MVP)** Deliver combined filters with list/map parity, mocked dataset, and DoR artefacts (sequence diagram, actions/queries, access patterns, OpenAPI, ERD).
- **Phase 1b** Add enhanced filtering & sorting (sub-categories, availability toggles, result ordering) building on this job.
- **Phase 2** Personalised search & saved journeys extend this foundation with saved filters, trending categories, and notifications.

### Data Requirements Snapshot (Phase 1a - Consumer Search)

**For this job, consumers need to see:**

- **Core identity** Legal name, trading name (optional), short description, primary circular-economy category
- **Geolocation** Primary address, GPS coordinates for distance calculation and map display
- **Operational details** Operating hours/days for "open now" indicators
- **Services** List of services offered (e.g., "Smartphone screen repair", "Laptop hardware diagnostics")
- **Ratings & reviews** Average rating, review count, and individual reviews for trust signals

**Deferred to merchant-specific jobs:**
- Business registration details (SSM/company number, business type)
- Verification status and badges (all merchants pre-verified for MVP)
- Service area radius (consumers judge distance themselves)
- Subscription tier details (admin-only concern)
- Detailed waste types and product tags (focusing on service categories for MVP)
- Certifications, impact metrics, promotional content

See `docs/project/research/circular-economy-merchant-attributes.md` for the full attribute catalogue.

### Related Artefacts

- Sequence diagram: `docs/project/specs/jobs/consumers/search-by-category/sequence-diagram.puml`.
- API contract: `merchants-search-service/docs/api/openapi.yml` (TBD extension for this job).
- Data model notes: align with listing schema updates when defined.
- Status board entry: `docs/project/status-board.md`.
