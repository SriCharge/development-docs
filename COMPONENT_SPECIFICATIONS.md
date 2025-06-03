# SriCharge Component Specifications

This document provides detailed specifications for the key UI components in the SriCharge application. These specifications serve as a reference for developers implementing the components using Next.js and Firebase.

## Core Components

### 1. Map Component

**Purpose**: Display interactive map with charging station markers and filtering capabilities.

**Technical Specifications**:
- **Library**: Google Maps JavaScript API or Mapbox GL JS
- **Viewport**: Responsive, full-width with dynamic height (60% of viewport on mobile, 70% on desktop)
- **Performance**: Lazy-loaded markers with clustering for performance
- **Offline Support**: Cached map tiles for basic functionality without internet

**Visual Specifications**:
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                                                             │
│                        MAP AREA                             │
│                                                             │
│                                                             │
│                                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │
│ │ Filter 1 │ │ Filter 2 │ │ Filter 3 │ │ Filter 4 │ │  More   │ │
│ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Props**:
- `initialCenter`: {lat: number, lng: number} - Initial map center coordinates
- `initialZoom`: number - Initial zoom level (1-20)
- `markers`: Array<StationMarker> - Array of station marker data
- `selectedMarkerId`: string | null - ID of currently selected marker
- `filters`: Array<FilterOption> - Available filter options
- `onMarkerClick`: (markerId: string) => void - Handler for marker click
- `onFilterChange`: (filters: Array<string>) => void - Handler for filter changes
- `onMapMove`: (bounds: MapBounds) => void - Handler for map movement

**States**:
- Loading state: Show skeleton or loading indicator
- Error state: Display error message with retry option
- Empty state: Show message when no stations match filters
- Selected state: Highlight selected marker

**Accessibility**:
- Keyboard navigable markers
- Screen reader announcements for marker selection
- Alternative text for marker icons
- High contrast mode support

**Mobile Adaptations**:
- Fullscreen map option
- Bottom sheet for station details
- Collapsible filter bar
- Touch-friendly marker size

### 2. Charger Card Component

**Purpose**: Display summary information about a charging station in a card format.

**Technical Specifications**:
- **Rendering**: Server-side rendered for SEO and performance
- **Animations**: Subtle hover effects and status transitions
- **Responsiveness**: Adapts from list item to grid item based on viewport

**Visual Specifications**:
```
┌─────────────────────────────────────────────────┐
│ [STATUS INDICATOR]  Station Name                │
│                                                 │
│ ⚡ Type: CCS2, CHAdeMO  │  ⏱ Available: 2/3    │
│                                                 │
│ ⭐ 4.5 (42 reviews)    │  📍 2.3 km away       │
│                                                 │
│ [DIRECTIONS]          [VIEW DETAILS]            │
└─────────────────────────────────────────────────┘
```

**Props**:
- `station`: StationData - Complete station data object
- `distance`: number | null - Distance from user (if available)
- `onDirectionsClick`: () => void - Handler for directions button
- `onDetailsClick`: () => void - Handler for details button
- `compact`: boolean - Whether to show compact version

**States**:
- Available: Green status indicator
- In Use: Yellow status indicator
- Unavailable: Red status indicator
- Unverified: Gray status indicator
- Selected: Highlighted border and background

**Accessibility**:
- Status conveyed by both color and text
- Proper heading hierarchy
- Sufficient color contrast
- Focus indicators for interactive elements

**Mobile Adaptations**:
- Horizontal scrolling list on map view
- Stacked full-width cards in list view
- Touch targets minimum 44×44px

### 3. Review List & Form Components

**Purpose**: Display user reviews and allow submission of new reviews.

**Technical Specifications**:
- **Data Binding**: Real-time updates with Firestore listeners
- **Validation**: Client and server-side validation for submissions
- **Optimistic UI**: Show pending state while submitting

**Visual Specifications - Review List**:
```
┌─────────────────────────────────────────────────┐
│ REVIEWS (42)                     [WRITE REVIEW] │
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ ⭐⭐⭐⭐⭐ John D. • 2 days ago               │ │
│ │                                             │ │
│ │ Great charging station with fast service.   │ │
│ │ The location is convenient and well-lit.    │ │
│ │                                             │ │
│ │ [👍 Helpful (12)]                           │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ ⭐⭐⭐ Amal P. • 1 week ago                  │ │
│ │                                             │ │
│ │ Charging was slow during peak hours.        │ │
│ │                                             │ │
│ │ [👍 Helpful (3)]                            │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ [LOAD MORE REVIEWS]                             │
└─────────────────────────────────────────────────┘
```

**Visual Specifications - Review Form**:
```
┌─────────────────────────────────────────────────┐
│ Write a Review                                  │
│                                                 │
│ Rating                                          │
│ [☆][☆][☆][☆][☆]                               │
│                                                 │
│ Your Review                                     │
│ ┌─────────────────────────────────────────────┐ │
│ │                                             │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ Add Photos                                      │
│ [+ UPLOAD]                                      │
│                                                 │
│ [CANCEL]                         [SUBMIT REVIEW]│
└─────────────────────────────────────────────────┘
```

**Props - Review List**:
- `stationId`: string - ID of the station to show reviews for
- `limit`: number - Number of reviews to show initially
- `onWriteReviewClick`: () => void - Handler for write review button

**Props - Review Form**:
- `stationId`: string - ID of the station to review
- `onSubmit`: (review: ReviewData) => Promise<void> - Submit handler
- `onCancel`: () => void - Cancel handler
- `initialRating`: number - Pre-filled rating (if any)

**States**:
- Loading: Show skeleton while fetching reviews
- Empty: Display message when no reviews exist
- Error: Show error message with retry option
- Submitting: Disable form while submitting
- Success: Show confirmation after submission

**Accessibility**:
- Keyboard accessible star rating
- Form validation messages
- Proper label associations
- Screen reader announcements for submission status

**Mobile Adaptations**:
- Full-screen modal for review form
- Simplified review cards
- Native file picker for photos

### 4. Verification Prompt Component

**Purpose**: Notify verifiers when they are near a charging station that needs verification.

**Technical Specifications**:
- **Triggering**: Geofencing with Firebase Cloud Functions
- **Notifications**: Push notifications via Firebase Cloud Messaging
- **Persistence**: Remember dismissed prompts to avoid repeated notifications

**Visual Specifications**:
```
┌─────────────────────────────────────────────────┐
│ Verification Opportunity                     [×] │
│                                                 │
│ You're near Fast Charge Station                 │
│ 📍 200m away • Last verified 3 days ago         │
│                                                 │
│ Earn 10 points by verifying this station's      │
│ current status and availability.                │
│                                                 │
│ [DISMISS]                           [VERIFY NOW]│
└─────────────────────────────────────────────────┘
```

**Props**:
- `station`: StationData - Station data to verify
- `distance`: number - Distance from user in meters
- `points`: number - Points to earn for verification
- `onVerifyClick`: () => void - Handler for verify button
- `onDismiss`: () => void - Handler for dismiss button

**States**:
- New: Highlighted with animation
- Viewed: Normal display after user interaction
- Dismissed: Hidden from view
- Actioned: Show confirmation after verification

**Accessibility**:
- Non-modal for less interruption
- Screen reader announcement on appearance
- Keyboard dismissible
- High contrast mode support

**Mobile Adaptations**:
- Bottom sheet or toast notification
- Swipe to dismiss
- Haptic feedback on appearance

### 5. Charger Form Component

**Purpose**: Allow charger owners to add or edit charging station details.

**Technical Specifications**:
- **Validation**: Progressive field validation with error messages
- **Image Handling**: Client-side compression before upload
- **Geolocation**: Map picker with address autocomplete
- **Persistence**: Save draft in localStorage

**Visual Specifications**:
```
┌─────────────────────────────────────────────────┐
│ Add New Charging Station                        │
│                                                 │
│ Station Name*                                   │
│ [                                             ] │
│                                                 │
│ Location*                                       │
│ ┌─────────────────────────────────────────────┐ │
│ │                                             │ │
│ │               [MAP SELECTOR]                │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ Address*                                        │
│ [                                             ] │
│                                                 │
│ Charger Type*                                   │
│ [ ] CCS2  [ ] CHAdeMO  [ ] Type 2  [ ] Other   │
│                                                 │
│ Power Output (kW)*                              │
│ [         ]                                     │
│                                                 │
│ Number of Connectors*                           │
│ [         ]                                     │
│                                                 │
│ Operating Hours*                                │
│ [ ] 24/7  [ ] Custom Hours                      │
│                                                 │
│ Payment Methods*                                │
│ [ ] Credit Card  [ ] Mobile App  [ ] Free      │
│                                                 │
│ Photos                                          │
│ [+ UPLOAD]                                      │
│                                                 │
│ Additional Information                          │
│ [                                             ] │
│                                                 │
│ [SAVE DRAFT]  [CANCEL]  [SUBMIT FOR APPROVAL]   │
└─────────────────────────────────────────────────┘
```

**Props**:
- `initialData`: Partial<StationData> - Pre-filled data for editing
- `onSubmit`: (data: StationData) => Promise<void> - Submit handler
- `onSaveDraft`: (data: StationData) => void - Save draft handler
- `onCancel`: () => void - Cancel handler
- `isEdit`: boolean - Whether this is an edit or new station

**States**:
- Editing: Normal form state
- Validating: Show validation messages
- Submitting: Disable form while submitting
- Success: Show confirmation after submission
- Error: Display error message with retry option

**Accessibility**:
- Proper form labeling
- Grouped form controls
- Error messages linked to fields
- Keyboard accessible map selector

**Mobile Adaptations**:
- Multi-step form with progress indicator
- Collapsible sections
- Native date/time pickers
- Simplified map interface

## Admin Components

### 6. User Management Component

**Purpose**: Allow administrators to view, search, and manage user accounts.

**Technical Specifications**:
- **Data Loading**: Paginated queries with Firestore
- **Search**: Server-side search implementation
- **Role Management**: Direct Firestore updates with security rules

**Visual Specifications**:
```
┌─────────────────────────────────────────────────┐
│ User Management                    [+ ADD USER] │
│                                                 │
│ [Search users...            ]  [Filter ▼]       │
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ Name       │ Email      │ Role    │ Actions │ │
│ ├───────────────────────────────────────────┬─┤ │
│ │ John Doe   │ john@e.com │ General │ ••• ▼  │ │
│ ├───────────────────────────────────────────┼─┤ │
│ │ Amal Perera│ amal@e.com │ Verifier│ ••• ▼  │ │
│ ├───────────────────────────────────────────┼─┤ │
│ │ Lanka EV   │ info@ev.lk │ Owner   │ ••• ▼  │ │
│ ├───────────────────────────────────────────┼─┤ │
│ │ Admin User │ admin@e.com│ Admin   │ ••• ▼  │ │
│ └───────────────────────────────────────────┴─┘ │
│                                                 │
│ Showing 4 of 120 users                          │
│ [PREV]  Page 1 of 30  [NEXT]                    │
└─────────────────────────────────────────────────┘
```

**Props**:
- `onUserAdd`: () => void - Handler for add user button
- `onUserEdit`: (userId: string) => void - Handler for edit action
- `onUserDelete`: (userId: string) => Promise<void> - Handler for delete
- `onRoleChange`: (userId: string, role: UserRole) => Promise<void>

**States**:
- Loading: Show skeleton while fetching users
- Empty: Display message when no users match search
- Error: Show error message with retry option
- Editing: Highlight row being edited

**Accessibility**:
- Sortable columns with screen reader announcements
- Keyboard navigable table
- ARIA roles for table structure
- Focus management for action menus

**Mobile Adaptations**:
- Card-based layout instead of table
- Swipe actions for common operations
- Simplified filtering interface
- Full-screen edit modal

### 7. Banner Ad Component

**Purpose**: Display promotional content to users in designated areas of the app.

**Technical Specifications**:
- **Loading**: Lazy-loaded with placeholder
- **Tracking**: Click and impression tracking
- **Targeting**: Show based on user role and location
- **Rotation**: Support for multiple ads in same slot

**Visual Specifications**:
```
┌─────────────────────────────────────────────────┐
│                                                 │
│                 [BANNER IMAGE]                  │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Props**:
- `placement`: 'map' | 'details' | 'search' | 'profile' - Ad location
- `userRole`: UserRole - Current user's role for targeting
- `onImpression`: () => void - Handler for recording impression
- `onClick`: () => void - Handler for ad click
- `fallback`: ReactNode - Content to show if no ad available

**States**:
- Loading: Show placeholder or skeleton
- Loaded: Display banner image
- Error: Show fallback content
- Clicked: Track click and navigate

**Accessibility**:
- Alternative text for banner images
- Proper ARIA roles for advertisements
- Keyboard accessible
- Non-intrusive placement

**Mobile Adaptations**:
- Responsive sizing
- Touch-friendly click area
- Reduced size on smaller screens
- Collapsible on scroll

## Implementation Guidelines

### Component Architecture

1. **Atomic Design Methodology**:
   - Atoms: Buttons, inputs, icons
   - Molecules: Form fields, cards, list items
   - Organisms: Forms, lists, complex UI sections
   - Templates: Page layouts
   - Pages: Complete screens

2. **State Management**:
   - Use React Context for global state (auth, roles)
   - Use Firebase hooks for real-time data
   - Use local state for component-specific concerns

3. **Performance Considerations**:
   - Implement code splitting for large components
   - Use React.memo for pure components
   - Optimize re-renders with useMemo and useCallback
   - Implement virtualization for long lists

4. **Testing Strategy**:
   - Unit tests for individual components
   - Integration tests for component interactions
   - Snapshot tests for UI regression
   - Accessibility tests with jest-axe

### Firebase Integration

1. **Authentication**:
   - Use Firebase Auth for user management
   - Implement custom claims for role-based access
   - Create higher-order components for protected routes

2. **Firestore**:
   - Design schema for efficient querying
   - Implement security rules for data protection
   - Use transactions for complex operations

3. **Cloud Functions**:
   - Implement serverless functions for:
     - User role management
     - Verification notifications
     - Status updates
     - Analytics processing

4. **Storage**:
   - Store user uploads and images
   - Implement client-side validation and compression
   - Set up security rules for access control

### Responsive Design

1. **Mobile-First Approach**:
   - Design for smallest screens first
   - Progressively enhance for larger screens
   - Use CSS Grid and Flexbox for layouts

2. **Breakpoints**:
   - Small: 0-576px (Mobile)
   - Medium: 577-768px (Tablet)
   - Large: 769-992px (Desktop)
   - Extra Large: 993px+ (Large Desktop)

3. **Touch Considerations**:
   - Minimum touch target size: 44×44px
   - Implement swipe gestures where appropriate
   - Provide haptic feedback for important actions

### Accessibility Requirements

1. **WCAG 2.1 AA Compliance**:
   - Proper heading structure
   - Sufficient color contrast
   - Keyboard navigation
   - Screen reader compatibility

2. **Semantic HTML**:
   - Use appropriate HTML elements
   - Implement ARIA attributes when needed
   - Ensure proper focus management

3. **Testing Tools**:
   - Lighthouse for automated testing
   - Screen reader testing
   - Keyboard navigation testing
   - Color contrast analyzers
