# Search by Category - UI Mockups

## Overview

This directory contains high-fidelity mockups for the "Search by Category" feature, designed to help consumers quickly locate circular-economy providers by combining category, waste type, product, and distance filters.

## Screen Flow Diagram

```
┌─────────────────┐
│   Home Page     │  Entry point (Story 003)
│  (Story 003)    │  See: ../access-home-page/mockups/
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Listings Page   │  Search results display
│ (Story 001)     │  - Filter summary & refinement
└────────┬────────┘  - Interactive map view
         │           - List of matching providers
         │           - Sort options (Distance/Rating/Name)
         ↓
┌─────────────────┐
│  Listing Page   │  Individual provider details (Story 002)
│  (Story 002)    │  See: ../view-detailed-business-information/mockups/
└─────────────────┘
```

## Key User Interactions

### Home Page (Story 003)

See [Story 003 mockups](../../access-home-page/mockups/) for home page interactions.

### Listings Page

1. **Filter Refinement**: Adjust active filters via top bar
2. **Map Interaction**: Click pins to preview business details
3. **List Navigation**: Scroll through provider cards, click "View Details"
4. **Sort Control**: Reorder results by distance, rating, or name
5. **Recenter Map**: "My Location" button to reset map view

### Listing Page (Story 002)

See [Story 002 mockups](../../view-detailed-business-information/mockups/) for detail view interactions.

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
- **Detail View**: See Story 002 mockups

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

| File                   | Description                         | Dimensions |
| ---------------------- | ----------------------------------- | ---------- |
| `02-listings-page.png` | Search results with map + list view | 1440x900   |

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

- Story Card: `../story-card-001.md`
- Sequence Diagram: `../sequence-diagrams/01-search-flow.puml` (TBD)
- API Contract: `svc-merchants/docs/api/openapi.yml` (TBD)
- Related Stories:
  - [Story 003 - Access Home Page](../../access-home-page/story-card-003.md) (entry point)
  - [Story 002 - View Detailed Business Information](../../view-detailed-business-information/story-card-002.md) (detail view)
