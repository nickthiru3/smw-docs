# Search by Category - UI Mockups

## Overview

This directory contains high-fidelity mockups for the "Search by Category" feature, designed to help consumers quickly locate circular-economy providers by combining category, waste type, product, and distance filters.

## Screen Flow Diagram

```
┌─────────────────┐
│   Home Page     │  Entry point with search interface
│  (01-home)      │  - Hero section with tagline
└────────┬────────┘  - Category/waste/product filters
         │           - Distance selector
         │           - "Find Services" CTA
         ↓
┌─────────────────┐
│ Listings Page   │  Search results display
│ (02-listings)   │  - Filter summary & refinement
└────────┬────────┘  - Interactive map view
         │           - List of matching providers
         │           - Sort options (Distance/Rating/Name)
         ↓
┌─────────────────┐
│  Listing Page   │  Individual provider details
│  (03-listing)   │  - Business information & services
└─────────────────┘  - Operating hours & contact
                     - Reviews & ratings
                     - Map with directions
                     - Similar businesses
```

## Key User Interactions

### Home Page
1. **Search Entry**: Users select category, waste type, product filters
2. **Location Input**: Distance radius selection (5km/10km/25km) or manual location entry
3. **Search Trigger**: "Find Services" button initiates search
4. **Category Browse**: Quick access via category cards in "Explore Categories" section

### Listings Page
1. **Filter Refinement**: Adjust active filters via top bar
2. **Map Interaction**: Click pins to preview business details
3. **List Navigation**: Scroll through provider cards, click "View Details"
4. **Sort Control**: Reorder results by distance, rating, or name
5. **Recenter Map**: "My Location" button to reset map view

### Listing Page
1. **Contact Actions**: Phone, website, directions buttons
2. **Quick Actions**: Check-in, save to favorites, share business
3. **Review Interaction**: Read reviews, write review (if authenticated)
4. **Similar Businesses**: Discover related providers nearby

## Design Decisions & Rationale

### Color Scheme
- **Primary Green (#00D084)**: Represents sustainability, growth, circular economy
- **Light Green Background (#E8F9F3)**: Soft, eco-friendly aesthetic
- **Dark Text (#1A1A1A)**: High contrast for readability
- **White Cards**: Clean, modern, content-focused design

### Layout Choices
- **Mobile-First**: Responsive design optimized for mobile devices
- **Map + List Parity**: Synchronized views to support different user preferences
- **Progressive Disclosure**: Essential information upfront, details on demand
- **Persistent Header**: Logo, auth buttons, "Add Listing" CTA always accessible

### Component Patterns
- **Filter Chips**: Visual, removable tags for active filters
- **Category Cards**: Icon + label for quick recognition
- **Provider Cards**: Compact summary with key info (name, tags, distance, rating)
- **CTA Buttons**: Prominent green buttons for primary actions
- **Verification Badge**: Trust signal for approved businesses

## Responsive Breakpoints

- **Mobile**: < 768px (single column, bottom nav consideration)
- **Tablet**: 768px - 1024px (adjusted spacing, larger touch targets)
- **Desktop**: > 1024px (multi-column layouts, expanded map view)

## State Variations

### Implemented in Mockups
- **Default/Success**: Normal search results display
- **Populated Results**: 24 providers found (02-listings)
- **Detail View**: Full business information (03-listing)

### Not Yet Mocked (Future Iterations)
- **Loading State**: Skeleton screens while fetching results
- **Empty State**: No providers found message with suggestions
- **Error State**: Network/API failure handling
- **Location Permission Denied**: Manual location entry fallback
- **Unauthenticated Actions**: Prompt to sign in for favorites/check-ins

## Accessibility Considerations

- **Color Contrast**: All text meets WCAG AA standards
- **Focus States**: Visible keyboard navigation indicators (to be implemented)
- **Touch Targets**: Minimum 44x44px for mobile interactions
- **Alt Text**: Descriptive labels for icons and images
- **Semantic HTML**: Proper heading hierarchy and landmark regions

## File Inventory

| File | Description | Dimensions |
|------|-------------|------------|
| `01-home-page.png` | Landing page with search interface | 1440x900 |
| `02-listings-page.png` | Search results with map + list view | 1440x900 |
| `03-listing-page.png` | Individual provider detail page | 1440x900 |

## Design Tools & Assets

- **Tool**: UX Pilot (AI-generated mockups)
- **Framework**: Tailwind CSS-compatible design
- **Icons**: Font Awesome (repair, recycle, location icons)
- **Typography**: System fonts (to be refined with design system)

## Next Steps

1. **Design System Integration**: Extract color tokens, spacing, typography into reusable design system
2. **Component Library**: Build Svelte components matching these mockups
3. **State Variations**: Create mockups for loading, empty, error states
4. **Interaction Prototypes**: Add clickable prototypes for user testing
5. **Accessibility Audit**: Validate against WCAG 2.1 AA standards

## Related Artefacts

- Job Card: `../job-card.md`
- Sequence Diagram: `../sequence-diagram.puml` (pending)
- ERD: `../erd.puml` (pending)
- API Contract: `merchants-search-service/docs/api/openapi.yml` (pending)
