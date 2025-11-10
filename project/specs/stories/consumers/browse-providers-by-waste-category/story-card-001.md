# Story 001: Browse Providers by Waste Category

**Status:** Ready for Dev  
**Assignee:** TBD  
**Sprint:** TBD  
**Epic:** Browse providers by waste category

---

## Story

**Actor:** Consumer / End User

**When** I need to find circular-economy services nearby,  
**I want to** search by combining category, waste type/product, and distance filters,  
**So I can** quickly locate relevant providers that meet my specific needs.

**Priority:** High

**Estimated Complexity:** Large

---

## Bundled Stories

> This story bundles the following stories from the story map:

- [ ] Story: Display category taxonomy on home page `[001]`
- [ ] Story: Select primary waste category `[001]`
- [ ] Story: Filter by waste type within category `[001]`
- [ ] Story: Filter by specific product `[001]`
- [ ] Story: Set distance radius for search `[001]`
- [ ] Story: View filtered results list `[001]`

---

## Acceptance Criteria

- [ ] Home page allows users to select ONE activity type (Repair Shop, Grocery & Refill, Recycling Center, Composting Site) and ONE distance option (Within 5km, Within 10km, Within 20km, Near me)
- [ ] Users must enter an address or postcode before searching (unless "Near me" is selected)
- [ ] When "Near me" is selected, address input is disabled with placeholder "Using your current location..."
- [ ] "Near me" option uses browser geolocation API to get user's current coordinates
- [ ] Location permission is only requested when user clicks "Find Services" (not on selection of "Near me")
- [ ] If location permission is denied, user sees helpful error message and address input is re-enabled for manual entry
- [ ] If location permission was previously denied, user sees instructions to enable it in browser settings or enter address manually
- [ ] "Near me" defaults to 5km radius from current location
- [ ] Clicking "Find Services" takes users to the listings page with initial filters applied
- [ ] Listings page displays a "Filters" button that opens a modal for additional refinement
- [ ] Filters modal allows users to select multiple categories (Electronics, Appliances, Clothing, Furniture) and ratings (4+ stars, 3+ stars)
- [ ] Users can change distance filter on the listings page via the filters modal
- [ ] Results show key listing fields: name, category tags, distance, rating, and verification badge (if available)
- [ ] Selecting a result opens the listing detail view (handled by Story 002) without losing current filters
- [ ] Search supports both map and list presentations, kept in sync when filters change
- [ ] Filter selections are preserved when navigating back from detail view
- [ ] Active filters are displayed as chips that can be cleared individually or all at once
- [ ] Layout is responsive on mobile, tablet, and desktop

---

## Scenarios

### Scenario 1: Search with category and distance from home page

```gherkin
Given I am on the home page
And I can see the category options (Repair Shop, Grocery & Refill, Recycling Center, Composting Site)
When I select "Repair Shop" as the activity type
And I select "Within 5km" as the distance
And I enter my address or postcode
And I click "Find Services"
Then I should be taken to the listings page
And I should see providers that offer repair services within 5km
And the "Repair Shop" and "Within 5km" filters should be active
```

### Scenario 2: Refine search with additional filters on listings page

```gherkin
Given I am on the listings page
And I have searched for "Repair Shop" within "5km"
When I click the "Filters" button
Then I should see the filters modal
When I select "Electronics" category
And I click "Apply Filters"
Then the results should update to show only repair shops that handle electronics
And the "Electronics" filter should be displayed as an active chip
And the map and list should update to reflect the filtered results
```

### Scenario 3: View results on map and list

```gherkin
Given I am on the listings page viewing search results
When I see the map view with provider markers
And I see the list view with provider cards
Then the map markers should match the list results
And clicking a map marker should highlight the corresponding list item
And the map should show the selected distance radius
```

### Scenario 4: Change distance filter on listings page

```gherkin
Given I am on the listings page viewing search results
And I see 24 providers within "Within 5km"
When I click the "Filters" button
And I select "Within 10km" as the distance
And I click "Apply Filters"
Then the results should update to show providers within 10km
And I should see more providers than before
And the map should adjust to show the new radius
And the distance filter chip should update to "Within 10km"
```

### Scenario 5: Navigate to detail view and return

```gherkin
Given I am viewing search results with active filters
When I click "View Details" on a provider card
Then I should see the provider detail page (Story 002)
And when I click the back button
Then I should return to the listings page
And my previous filters should still be active
And my scroll position should be preserved
```

### Scenario 6: Search using "Near me" with location permission granted

```gherkin
Given I am on the home page
When I select "Repair Shop" as the activity type
And I select "Near me" as the distance option
Then the address input should be disabled
And the placeholder should show "Using your current location..."
When I click "Find Services"
Then the browser should request my location permission
When I grant permission
Then I should be taken to the listings page
And I should see providers within 5km of my current location
And the distance filter chip should show "Near me"
And the map should be centered on my current location
```

### Scenario 7: Search using "Near me" with location permission denied

```gherkin
Given I am on the home page
When I select "Repair Shop" as the activity type
And I select "Near me" as the distance option
And I click "Find Services"
Then the browser should request my location permission
When I deny permission
Then I should see an error message "Location access denied. Please enter your address below."
And the address input should become enabled
And I should be able to enter my address manually
And the distance option should revert to "Within 5km"
```

### Scenario 8: "Near me" with location permission previously denied

```gherkin
Given I am on the home page
And I have previously denied location permission for this site
When I select "Near me" as the distance option
Then I should see a warning message
And the message should say "Location access is blocked. Please enable it in your browser settings or enter your address below."
And the address input should remain enabled
When I click "Find Services" without entering an address
Then I should see a validation error "Please enter your address or enable location access"
```

### Scenario 9: Handle no results

```gherkin
Given I am on the home page
When I select "Composting Site" as the activity type
And I select "Within 5km" as the distance
And I enter my address
And I click "Find Services"
And no providers match my criteria
Then I should see the listings page with an empty state message
And I should see suggestions to broaden my search (e.g., increase distance)
And I should be able to adjust filters or clear them easily
```

---

## Extended Requirements

### Business Rules

- **Two-Stage Filtering:** The search experience is split into two stages:
  1. **Home Page (Initial Search):** Users select ONE activity type and ONE distance, then enter location to get initial results
  2. **Listings Page (Refinement):** Users can refine results using the filters modal to add multiple category filters, rating filters, and change distance
- **Required Fields:** Address or postcode is required before initiating a search from the home page (unless "Near me" is selected)
- **Geolocation Handling:**
  - "Near me" uses browser Geolocation API (`navigator.geolocation.getCurrentPosition()`)
  - Location permission is requested only when user clicks "Find Services", not when selecting "Near me"
  - If permission is denied, user is prompted to enter address manually and distance reverts to "Within 5km"
  - If permission was previously denied, user sees instructions to enable it in browser settings
  - "Near me" defaults to 5km radius from user's current coordinates
  - Address input is disabled when "Near me" is selected, with placeholder "Using your current location..."
- **Filter Persistence:** All active filters must persist when navigating to detail view and back
- **Filter Display:** Active filters are shown as removable chips on the listings page
- **Map-List Sync:** Map markers and list results must always show the same filtered set of providers

### Implementation Notes & Assumptions

- MVP uses auto-approved listings; manual verification workflow will arrive later.
- Initial implementation uses a mocked merchant directory (flat file or in-memory dataset). Future iterations will integrate with the Users service (Cognito + DynamoDB) and/or dedicated search indices.
- Geolocation error handling must be implemented for all permission states (granted, denied, prompt, unavailable)
- Browser geolocation accuracy varies by device: GPS (mobile, ~5-50m), WiFi triangulation (~50-100m), IP address (~city-level)
- Geolocation timeout should be set to 10 seconds to avoid indefinite waiting
- Requires baseline taxonomy for categories, waste types, and product tags sourced during dataset ingestion.
- Backend ownership: `Merchants Service` (`svc-merchants/`) exposes `merchants/search?category={categoryName}&lat={lat}&lng={lng}&radius={km}` per the OpenAPI contract.

### Phase Scope

- **Phase 1a (MVP)** Deliver combined filters with list/map parity, mocked dataset, and DoR artefacts (sequence diagram, actions/queries, access patterns, OpenAPI, ERD).
- **Phase 1b** Add enhanced filtering & sorting (sub-categories, availability toggles, result ordering) building on this story.
- **Phase 2** Personalised search & saved journeys extend this foundation with saved filters, trending categories, and notifications.

### Data Requirements Snapshot

**For search results display, consumers need to see:**

- **Core identity:** Business name, trading name (optional), short description, primary category
- **Geolocation:** GPS coordinates for distance calculation and map markers
- **Distance:** Calculated distance from user's location
- **Category metadata:** Primary category, waste types accepted, product tags
- **Trust signals:** Average rating, review count, verification badge (if available)
- **Quick info:** "Open now" indicator based on operating hours

**Deferred to Story 002 (Detail View):**

- Full business description and contact details
- Complete operating hours schedule
- Physical address display
- Photos and media
- Individual reviews and ratings
- Accepted materials list

**Data Sources:**

- `Merchants` table: Core business information
- `MerchantCategories` table: Category and waste type associations
- `MerchantMetrics` table: Ratings and review counts

See `docs/project/research/circular-economy-merchant-attributes.md` for full attribute catalogue.

---

## Technical Notes

**Data Sources:**

- Table: `Merchants`, `MerchantCategories`, `MerchantMetrics`
- Access Pattern: Search merchants by category, waste type, product, and location
- Query: GSI on category with filter expressions for waste type and geolocation

**Components:**

- Frontend Pages:
  - `routes/+page.svelte` (home page with initial search form)
  - `routes/listings/+page.svelte` (results page with map/list views)
- Components:
  - `ActivityTypeSelector.svelte` (home page category selection)
  - `DistanceSelector.svelte` (home page distance selection with "Near me" option)
  - `LocationInput.svelte` (address/postcode input with conditional disable)
  - `FiltersModal.svelte` (listings page refinement modal)
  - `FilterChips.svelte` (active filter display)
  - `ProviderCard.svelte` (list item display)
  - `MapView.svelte` (interactive map with markers)
  - `ListView.svelte` (scrollable provider list)
- Services:
  - `geolocation.service.ts` (wrapper for browser Geolocation API with error handling)
  - `geocoding.service.ts` (convert address to coordinates via geocoding API)
- BFF Route: `/api/merchants/search/+server.ts`
- Backend Service: `svc-merchants`

**API Endpoints:**

- `GET /api/merchants/search?category={cat}&wasteType={type}&product={prod}&lat={lat}&lng={lng}&radius={km}`

**Dependencies:**

- Depends on: None (foundational search story)
- Blocks: 002 (Detail view needs search context), 003+ (Other consumer features build on search)

**Performance Targets:**

- Search API response: <500ms
- Map rendering: <1s for 100 markers
- Filter updates: <200ms (client-side)
- Geolocation request timeout: 10s (with fallback to manual entry)

---

## Definition of Done

- [ ] Code written and unit tested
- [ ] BDD scenarios implemented as Playwright tests
- [ ] Responsive design verified (mobile: 375px, tablet: 768px, desktop: 1440px)
- [ ] Accessibility checked (WCAG 2.1 AA)
  - [ ] Screen reader tested
  - [ ] Keyboard navigation verified
  - [ ] Color contrast checked
  - [ ] ARIA labels added
- [ ] Map and list views stay synchronized
- [ ] Filter state persists across navigation
- [ ] Empty state and error handling implemented
- [ ] Geolocation permission handling tested for all states (granted, denied, prompt, unavailable)
- [ ] "Near me" functionality tested on mobile and desktop browsers
- [ ] Geolocation timeout and error fallback verified
- [ ] Address input disable/enable behavior verified for "Near me" selection
- [ ] Code reviewed and approved
- [ ] Integration tests passing
- [ ] Documentation updated
- [ ] Deployed to staging
- [ ] Demo to Product Owner completed
- [ ] Acceptance criteria validated

---

## Design References

**Mockups:**

- [Listings page with map and list view](./mockups/02-listings-page.png)
- [Mockups README with flow diagram](./mockups/README.md)

**Referenced from Other Stories:**

- Home page: See [Story: Access Home Page mockups](../access-home-page/mockups/home-page.png)
- Listing detail page: See [Story: View Detailed Business Information mockups](../view-detailed-business-information/mockups/listing-page.png)

---

## Related Artefacts

**Story Map:** `docs/project/specs/story-map.md` (Stories bundled in this story: All stories under "Browse providers by waste category" epic with ID `[001]`)

**Sequence Diagrams:**

- [sequence-diagrams/01-search-flow.puml](./sequence-diagrams/01-search-flow.puml) (TBD)

**API Specifications:**

- BFF API: [api.yml](./api.yml) (TBD)
- Backend API: `svc-merchants/docs/api/openapi.yml` (TBD extension for search endpoint)

**Data Model:**

- ERD: `docs/project/specs/erd.puml`
- Entity files: `docs/project/specs/entities/merchants.md`, `docs/project/specs/entities/merchant-categories.md`

**Actions & Queries:**

- [actions-queries.md](./actions-queries.md) (TBD)

**Status Tracking:**

- `docs/project/status-board.md`

---

## Notes & Questions

**Open Questions:**

- Should we implement real-time "Open Now" indicator in Phase 1a or defer to Phase 1b?
- Should map clustering be implemented in Phase 1a or can it be deferred?
- Which geocoding service should we use for address-to-coordinates conversion (Google Maps, Mapbox, OpenStreetMap/Nominatim)?
- Should we cache geolocation coordinates in session storage to avoid repeated permission requests?
- Should "Near me" be available on listings page filters modal, or only on home page?

**Decisions Log:**

- **2024-11-03:** Separated "View detailed business information" into Story 002 to reduce scope and allow parallel development
- **2024-11-03:** Decided to bundle all search/filter stories into Story 001 as they share the same components and data flow
- **2024-11-05:** Added "Near me" functionality with browser Geolocation API; location permission requested on "Find Services" click, not on option selection
- **2024-11-05:** Decided to disable address input when "Near me" is selected, with fallback to manual entry if permission denied
- **2024-11-06:** Completed data modeling (Faux-SQL approach) for Merchants entity with generic GSI attributes (GSI1PK) for flexibility
- **2024-11-06:** Data model validated against anti-patterns - passed all checks, ready for implementation

**Implementation Notes:**

- Using Leaflet.js for map implementation (lighter than Google Maps SDK)
- Filter state managed in URL query parameters for shareability
- Debouncing filter changes to avoid excessive API calls
- Geolocation implementation:
  - Use `navigator.geolocation.getCurrentPosition()` with 10s timeout
  - Enable high accuracy on mobile devices: `{ enableHighAccuracy: true, timeout: 10000, maximumAge: 300000 }`
  - Cache coordinates in session storage (5min TTL) to avoid repeated permission prompts
  - Handle all error codes: `PERMISSION_DENIED`, `POSITION_UNAVAILABLE`, `TIMEOUT`
  - Show loading state while waiting for geolocation response
