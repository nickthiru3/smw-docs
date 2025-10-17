# Job: Services Search

## Statement
Consumers want to search for circular-economy providers by category, waste type, product, or distance so they can quickly find relevant services nearby.

## Business Rules / Acceptance Criteria
- Users can filter listings by category (Repair, Refill/Grocery, Recycling, Donate) and see matching results.
- Users can filter by waste type and product tags to refine results.
- Distance filtering offers preset radii (e.g., 5km, 10km, 25km) based on user location.
- Results show key listing fields: name, category tags, distance, verification badge (if available).
- Selecting a result opens the listing detail view (handled by separate job) without losing current filters.
- Search supports both map and list presentations, kept in sync when filters change.

## Notes & Assumptions
- MVP uses auto-approved listings; manual verification workflow will arrive later.
- Initial implementation uses a mocked merchant directory (flat file or in-memory dataset). Future iterations will integrate with the Users service (Cognito + DynamoDB) and/or dedicated search indices.
- For early iterations, assume location permissions are granted; fallback UX needed for manual location entry.
- Requires baseline taxonomy for categories, waste types, and product tags sourced during dataset ingestion.
- Backend ownership: `Merchants Search Service` (`merchants-search-service/`) exposes `/search/merchants` per the OpenAPI contract.

## Data Requirements Snapshot
- **Core identity** Mandatory fields at sign-up: legal name, trading name (optional), SSM/company number, business type, short description, primary circular-economy category, specific service types, verified owner contact (email + phone), and subscription tier selection.
- **Geolocation & coverage** Every listing must supply primary address, GPS coordinates, and service-area radius; additional branches are optional but share the same structure. These attributes underpin Google Maps/Waze navigation and distance filters.
- **Operational details** Required operating hours/days with optional schedule nuances (seasonal closures, appointments, delivery/pickup policies).
- **Classification metadata** Accepted waste types, product tags, secondary categories, and keyword tags enable multi-filter search; tiers determine maximum counts (Basic vs Silver/Gold/Platinum).
- **Trust & sustainability** Optional certifications, impact metrics, and verification badges surface as value-add signals; `MerchantMetrics` aggregates ratings, saves, check-ins for ordering.
- **Media & promotions** Tier-governed limits for photos, videos, keywords, and promotional banners; managed via `MerchantMedia` and `Promotion` entities.

See `merchants-search-service/docs/research/circular-economy-merchant-attributes.md` for the full attribute catalogue and tier thresholds feeding the ER model in `merchants-search-service/docs/er-diagram.puml`.

## Related Artefacts
- Sequence diagram: `docs/specs/jobs/consumers/services-search/sequence-diagram.puml`.
- API contract: `merchants-search-service/docs/api/openapi.yml` (TBD extension for this job).
- Data model notes: align with listing schema updates when defined.
- Status board entry: `docs/project/status-board.md`.
