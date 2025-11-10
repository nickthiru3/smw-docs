# View Detailed Business Information - UI Mockups

## Overview

This directory contains mockups for the merchant listing detail page, where consumers view complete business information after selecting a provider from search results (Job 001).

## Screen Flow Diagram

```
┌─────────────────┐
│ Listings Page   │  Search results (Job 001)
│  (Job 001)      │  User clicks "View Details"
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  Listing Page   │  Individual provider details
│  (listing-page) │  - Business name & description
└─────────────────┘  - Operating hours
                     - Contact details (phone, email, website)
                     - Physical address with map
                     - Back to search results
```

## Key User Interactions

### Listing Detail Page

1. **View Business Info**: Read business name, description, and key details
2. **Check Operating Hours**: See weekly schedule and special hours/closures
3. **Contact Actions**: 
   - Tap phone number to call (mobile)
   - Click email to compose message
   - Click website to visit in new tab
4. **View Location**: See address and map with business location pin
5. **Navigate Back**: Return to search results with preserved filters

## User Flow

**Entry Point:** User navigates from Job 001 search results by clicking "View Details" on a provider card

**Actions Available:**
- View complete business information
- Contact business via phone, email, or website
- View location on map
- Navigate back to search results

**Exit Points:**
- Back button → Returns to search results (Job 001)
- Contact links → Opens phone dialer, email client, or external website
- Future: Navigate to other detail view features (reviews, photos, etc.)

## Design Decisions & Rationale

### Layout Structure

- **Hero Section**: Business name and primary category prominently displayed
- **Information Sections**: Organized into clear, scannable blocks
  - Business description
  - Operating hours
  - Contact details
  - Address and map
- **Mobile-First**: Optimized for mobile viewing with stacked sections
- **Responsive**: Adapts to tablet and desktop with wider layouts

### Contact Interaction Patterns

- **Phone Links**: `tel:` protocol for one-tap calling on mobile
- **Email Links**: `mailto:` protocol with pre-filled subject line
- **Website Links**: Open in new tab to preserve user's place
- **Visual Indicators**: Icons clearly indicate action type (call, email, web)

### Missing Data Handling

- **Graceful Degradation**: Sections with no data are hidden
- **"Not Provided" Labels**: Used sparingly for expected-but-missing fields
- **Layout Adjustment**: No empty gaps when optional fields are missing

### Color Scheme

- **Consistent with Job 001**: Same green primary color (#00D084)
- **Clear Hierarchy**: Headings, body text, and metadata use consistent typography
- **High Contrast**: Ensures readability on all devices

## Responsive Breakpoints

- **Mobile**: < 768px (single column, stacked sections, large touch targets)
- **Tablet**: 768px - 1024px (slightly wider layout, more breathing room)
- **Desktop**: > 1024px (two-column layout possible for some sections)

## State Variations

### Implemented in Mockups

- **Complete Information**: All fields populated (listing-page.png)

### Not Yet Mocked (Future Iterations)

- **Loading State**: Skeleton screen while fetching data
- **Error State**: Network/API failure handling
- **Missing Optional Fields**: Website not provided, special hours not set
- **Mobile View**: Specific mobile-optimized layout
- **Tablet View**: Medium-width layout

## Accessibility Considerations

- **Heading Hierarchy**: Proper H1 (business name) → H2 (section headings) structure
- **ARIA Labels**: Descriptive labels for contact links and map
- **Keyboard Navigation**: All interactive elements accessible via keyboard
- **Touch Targets**: Minimum 44x44px for mobile tap targets
- **Color Contrast**: All text meets WCAG AA standards
- **Screen Reader**: Semantic HTML ensures proper screen reader experience

## File Inventory

| File | Description | Dimensions |
|------|-------------|------------|
| `listing-page.png` | Merchant detail page with complete business info | 1440x900 |

## Design Tools & Assets

- **Tool**: UX Pilot (AI-generated mockups)
- **Framework**: Tailwind CSS-compatible design
- **Icons**: Font Awesome (phone, email, website, location icons)
- **Map**: Google Maps or Leaflet integration (to be determined)
- **Typography**: System fonts (to be refined with design system)

## Next Steps

1. **Mobile-Specific Mockups**: Create dedicated mobile view mockups
2. **State Variations**: Design loading, error, and missing-data states
3. **Component Breakdown**: Identify reusable components (ContactCard, HoursDisplay, etc.)
4. **Interaction Prototypes**: Create clickable prototype for user testing
5. **Accessibility Audit**: Validate against WCAG 2.1 AA standards

## Related Artefacts

- Job Card: `../job-card-002.md`
- Sequence Diagram: `../sequence-diagrams/01-fetch-merchant-details.puml` (TBD)
- API Contract: `svc-merchants/docs/api/openapi.yml` (GET /merchants/:id)
- Related Job: [Job 001 - Browse Providers by Waste Category](../../browse-providers-by-waste-category/job-card-001.md)
- Job 001 Mockups: [Search results page](../../browse-providers-by-waste-category/mockups/02-listings-page.png)
