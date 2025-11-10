# Story 002: View Detailed Business Information

**Status:** Ready for Dev  
**Assignee:** TBD  
**Sprint:** TBD  
**Epic:** View detailed business information

---

## Story

**Actor:** Consumer / End User

**When** a consumer has found a provider through search and wants to learn more,  
**I want to** view complete business information including description, hours, contact details, and address,  
**So I can** evaluate if the provider meets my needs and decide how to contact them.

**Priority:** High

**Estimated Complexity:** Medium

---

## Bundled Stories

> This story bundles the following stories from the story map:

- [ ] Story: Display business name and description `[002]`
- [ ] Story: Show operating hours `[002]`
- [ ] Story: Display contact details (phone, email, website) `[002]`
- [ ] Story: Show physical address `[002]`

---

## Acceptance Criteria

- [ ] Display business name as page heading
- [ ] Show full business description with proper formatting
- [ ] Display complete operating hours schedule including special hours/holidays
- [ ] Show contact details with interactive links (tel:, mailto:, external)
- [ ] Display physical address with map integration
- [ ] All fields handle missing data gracefully (show "Not provided" or hide section)
- [ ] Contact links are clickable and functional on mobile (tel: opens dialer, mailto: opens email client)
- [ ] Layout is responsive on mobile, tablet, and desktop
- [ ] Accessibility: proper heading hierarchy, ARIA labels, keyboard navigation
- [ ] Back navigation returns to search results with preserved filters

---

## Scenarios (BDD Style)

### Scenario 1: View complete business information

```gherkin
Given I have navigated from search results to a merchant listing page
And the listing has all business details populated
When the page loads
Then I should see the business name as the page heading
And I should see the full business description
And I should see formatted operating hours for each day of the week
And I should see clickable phone, email, and website links
And I should see the full physical address
And I should see a map showing the business location
```

### Scenario 2: Handle missing optional fields gracefully

```gherkin
Given I am viewing a merchant listing page
And the business has not provided a website URL
When the page loads
Then I should not see a website link
And the layout should adjust gracefully without empty space
And other contact methods (phone, email) should still be visible
```

### Scenario 3: Click phone number on mobile device

```gherkin
Given I am viewing a listing on a mobile device
And the business has provided a phone number
When I tap the phone number
Then my phone's dialer should open
And the number should be pre-filled
```

### Scenario 4: Open website in new tab

```gherkin
Given I am viewing a merchant listing page
And the business has provided a website URL
When I click the website link
Then the website should open in a new browser tab
And I should remain on the listing page in the original tab
```

### Scenario 5: View operating hours with special schedules

```gherkin
Given I am viewing a merchant listing page
And the business has special holiday hours
When I view the operating hours section
Then I should see regular weekly hours
And I should see a notice about upcoming holiday closures
And I should see the date range for special hours
```

### Scenario 6: Navigate back to search results

```gherkin
Given I navigated to this listing from search results
And I had active filters (category: Repair, distance: 10km)
When I click the back button
Then I should return to the search results page
And my previous filters should still be active
And my scroll position should be preserved
```

---

## Extended Requirements

### Business Rules

- Business name is mandatory (cannot be null)
- Description has a maximum length of 500 characters
- Phone numbers must be formatted consistently (e.g., +60 12-345 6789)
- Email addresses must be validated format
- Website URLs must include protocol (https://)
- Operating hours support multiple formats:
  - Standard weekly schedule (Mon-Fri: 9am-5pm)
  - Special hours (Holidays, seasonal closures)
  - "By appointment only" flag
  - "24/7" flag
- Address must include at minimum: street, city, postcode

### Implementation Notes & Assumptions

- Data source: `Merchants` table in DynamoDB
- MVP uses pre-verified merchant data (no real-time verification status)
- Phone/email/website are optional fields
- Address geocoding is pre-computed during merchant onboarding
- Map integration uses embedded Google Maps or Leaflet (separate component)
- For Phase 1a, assume all merchants have at least name and address
- Backend ownership: `Merchants Service` (`svc-merchants/`) exposes `GET /merchants/:id`
- This page is accessed from Story 001 search results

### Phase Scope

- **Phase 1a (MVP):** Display static business information with responsive layout and basic error handling
- **Phase 1b:** Add "Open Now" indicator based on current time and operating hours, add directions button
- **Phase 2:** Add real-time availability, booking integration, and merchant-verified badge

### Data Requirements Snapshot

**Core fields needed from `Merchant` entity:**

- `merchantId` (PK)
- `businessName` (required)
- `tradingName` (optional)
- `description` (required, max 500 chars)
- `phone` (optional)
- `email` (optional)
- `websiteUrl` (optional)
- `address` (object with street, city, state, postcode, country)
- `geoLocation` (lat/lng for map)
- `operatingHours` (array of schedule objects)
- `specialHours` (array of exception schedules)

**Deferred to later stories:**

- Verification status and badges (Phase 1b)
- Ratings and reviews (separate epic)
- Photos and media gallery (separate epic)
- Accepted materials list (separate epic)
- Call-to-action buttons (separate epic)

See `docs/project/research/circular-economy-merchant-attributes.md` for full attribute catalogue.

---

## Technical Notes

**Data Sources:**

- Table: `Merchants`
- Access Pattern: Get merchant by ID
- Query: `PK = MERCHANT#<merchantId>`

**Components:**

- Frontend Page: `routes/listings/[id]/+page.svelte`
- Child components: `BusinessInfo.svelte`, `ContactDetails.svelte`, `OperatingHours.svelte`, `AddressMap.svelte`
- BFF Route: `/api/merchants/[id]/+server.ts`
- Backend Service: `svc-merchants`

**API Endpoints:**

- `GET /api/merchants/:id` - Retrieve merchant details

**Dependencies:**

- Depends on: 001 (Search provides navigation context)
- Blocks: 003+ (Other detail view features like reviews, photos, etc.)

**Performance Targets:**

- Page load: <1.5s
- API response: <200ms
- Time to Interactive: <2s
- Map component lazy-loaded to improve initial load

---

## Definition of Done

- [ ] Code written and unit tested
- [ ] BDD scenarios implemented as Playwright tests
- [ ] Responsive design verified (mobile: 375px, tablet: 768px, desktop: 1440px)
- [ ] Accessibility checked (WCAG 2.1 AA)
  - [ ] Screen reader tested with NVDA
  - [ ] Keyboard navigation verified
  - [ ] Color contrast checked
  - [ ] ARIA labels added
- [ ] Back navigation preserves search context
- [ ] Missing fields handled gracefully
- [ ] Contact links functional on mobile
- [ ] Code reviewed and approved
- [ ] Integration tests passing
- [ ] Documentation updated
- [ ] Deployed to staging
- [ ] Demo to Product Owner completed
- [ ] Acceptance criteria validated

---

## Design References

**Mockups:**

- [Listing detail page](./mockups/listing-page.png)
- [Mockups README](./mockups/README.md) (to be created)

**Referenced from Other Stories:**

- Related Stories:
  - [Story 003 - Access Home Page](../../access-home-page/story-card-003.md) (entry point)
  - [Story 002 - View Detailed Business Information](../../view-detailed-business-information/story-card-002.md) (detail view)

---

## Related Artefacts

**Story Map:** `docs/project/specs/story-map.md` (Stories bundled in this story: All stories under "View detailed business information" epic with ID `[002]`)

**Sequence Diagrams:**

- [sequence-diagrams/01-fetch-merchant-details.puml](./sequence-diagrams/01-fetch-merchant-details.puml) (TBD)

**API Specifications:**

- BFF API: [api.yml](./api.yml) (TBD)
- Backend API: `svc-merchants/docs/api/openapi.yml` (GET /merchants/:id endpoint)

**Data Model:**

- ERD: `docs/project/specs/erd.puml`
- Entity file: `docs/project/specs/entities/merchants.md`

**Actions & Queries:**

- [actions-queries.md](./actions-queries.md) (TBD)

**Status Tracking:**

- `docs/project/status-board.md`

---

## Notes & Questions

**Open Questions:**

- Should we show "Verified" badge in Phase 1a or defer to Phase 1b?
- What's the fallback behavior if geocoding fails for the address?
- Should we include a "Report incorrect information" link in Phase 1a?

**Decisions Log:**

- **2024-11-03:** Separated from Story 001 to reduce scope and allow parallel development
- **2024-11-03:** Decided to bundle all basic business info display into one story (002) since they share the same component and data source
- **2024-11-03:** Map component will be lazy-loaded to improve initial page load performance

**Implementation Notes:**

- Using Svelte stores for merchant data caching
- Phone number formatting handled by `libphonenumber-js` library
- Map component is lazy-loaded to improve initial page load performance
- Back navigation state managed via SvelteKit's navigation API
