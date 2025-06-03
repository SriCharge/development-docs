# SriCharge MVP - Project Structure

This document outlines the folder structure and organization for the SriCharge Progressive Web App (PWA) using Angular and .NET.

## Overall Solution Structure

```
SriCharge/
├── SriCharge.API/              # .NET Core API backend
│   ├── Controllers/            # API controllers
│   ├── Models/                 # Domain models
│   ├── Data/                   # Data access layer
│   │   ├── Entities/           # Entity Framework entities
│   │   ├── Repositories/       # Repository pattern implementations
│   │   └── ApplicationDbContext.cs
│   ├── Services/               # Business logic services
│   ├── Middleware/             # Custom middleware
│   ├── Helpers/                # Helper classes and utilities
│   ├── Extensions/             # Extension methods
│   ├── appsettings.json        # Configuration settings
│   └── Program.cs              # Application entry point
│
├── SriCharge.Web/              # Angular PWA frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── core/           # Core functionality
│   │   │   │   ├── auth/       # Authentication services
│   │   │   │   ├── http/       # HTTP interceptors
│   │   │   │   ├── guards/     # Route guards
│   │   │   │   └── models/     # Shared models/interfaces
│   │   │   │
│   │   │   ├── shared/         # Shared components and services
│   │   │   │   ├── components/ # Reusable components
│   │   │   │   ├── directives/ # Custom directives
│   │   │   │   ├── pipes/      # Custom pipes
│   │   │   │   └── services/   # Shared services
│   │   │   │
│   │   │   ├── features/       # Feature modules
│   │   │   │   ├── map/        # Map feature
│   │   │   │   ├── chargers/   # Charger management
│   │   │   │   ├── reviews/    # Review system
│   │   │   │   ├── admin/      # Admin dashboard
│   │   │   │   ├── owner/      # Charger owner portal
│   │   │   │   └── verifier/   # Verifier functionality
│   │   │   │
│   │   │   ├── layouts/        # Layout components
│   │   │   │   ├── user-layout/
│   │   │   │   ├── admin-layout/
│   │   │   │   ├── owner-layout/
│   │   │   │   └── verifier-layout/
│   │   │   │
│   │   │   └── app.module.ts   # Main app module
│   │   │
│   │   ├── assets/             # Static assets
│   │   │   ├── icons/          # App icons
│   │   │   └── images/         # Images
│   │   │
│   │   ├── environments/       # Environment configurations
│   │   │
│   │   ├── styles/             # Global styles
│   │   │   ├── _variables.scss # SCSS variables
│   │   │   ├── _mixins.scss    # SCSS mixins
│   │   │   └── main.scss       # Main styles
│   │   │
│   │   ├── index.html          # Main HTML file
│   │   └── manifest.webmanifest # PWA manifest
│   │
│   ├── angular.json            # Angular configuration
│   └── package.json            # NPM dependencies
│
├── SriCharge.Tests/            # Test projects
│   ├── SriCharge.API.Tests/    # Backend tests
│   └── SriCharge.Web.Tests/    # Frontend tests
│
└── UI-UX-Guidelines/           # UI/UX documentation
    ├── design-system/          # Design system documentation
    ├── component-library/      # Component library guidelines
    ├── accessibility/          # Accessibility guidelines
    ├── responsive-design/      # Responsive design patterns
    └── best-practices/         # UI/UX best practices
```

## Backend Structure Details

### Controllers

```
SriCharge.API/Controllers/
├── AuthController.cs           # Authentication endpoints
├── UsersController.cs          # User management
├── StationsController.cs       # Charging station endpoints
├── ReviewsController.cs        # Review system endpoints
├── RequestsController.cs       # Charger requests
└── AdsController.cs            # Advertisement management
```

### Data Models

```
SriCharge.API/Data/Entities/
├── User.cs                     # User entity
├── Station.cs                  # Charging station entity
├── Review.cs                   # Review entity
├── PinRequest.cs               # Charger request entity
└── Ad.cs                       # Advertisement entity
```

## Frontend Structure Details

### Core Components

```
SriCharge.Web/src/app/shared/components/
├── map/                        # Map components
│   ├── map-container/          # Main map container
│   ├── charger-marker/         # Charger marker component
│   ├── info-window/            # Info window for markers
│   ├── search-bar/             # Map search functionality
│   ├── filter-controls/        # Charger filtering controls
│   └── location-button/        # Current location button
│
├── chargers/                   # Charger components
│   ├── charger-card/           # Charger info card
│   ├── charger-status/         # Status indicator
│   ├── charger-details/        # Detailed charger info
│   ├── review-list/            # List of reviews
│   ├── review-form/            # Form to submit reviews
│   └── request-form/           # Form to request new charger
│
├── admin/                      # Admin components
│   ├── dashboard/              # Admin dashboard
│   ├── user-management/        # User management panel
│   ├── request-approval/       # Charger request approval
│   ├── heatmap-view/           # Demand heatmap
│   └── ad-management/          # Banner ad management
│
├── owner/                      # Owner components
│   ├── owner-dashboard/        # Owner dashboard
│   ├── charger-form/           # Add/edit charger form
│   ├── charger-list/           # List of owner's chargers
│   └── status-update/          # Update charger status
│
└── verifier/                   # Verifier components
    ├── verifier-prompt/        # Nearby verification prompt
    ├── status-verification/    # Status verification form
    └── points-display/         # Credibility points display
```

## Database Schema Design

### Users Table
- Id (PK)
- Name
- Email
- Role (enum: "user", "verifier", "owner", "admin")
- CredibilityScore
- CreatedAt
- LastLogin
- ProfileImageUrl (optional)

### Stations Table
- Id (PK)
- Name
- Latitude
- Longitude
- Address
- Type (enum: "Type 1", "Type 2", "CCS", "CHAdeMO", etc.)
- Power (kW)
- Status (enum: "available", "in-use", "broken")
- OwnerId (FK to Users)
- LastVerified
- CreatedAt
- UpdatedAt
- ImageUrl (optional)

### StationVerifiers Table (Many-to-Many)
- StationId (PK, FK to Stations)
- UserId (PK, FK to Users)
- VerifiedAt

### Reviews Table
- Id (PK)
- StationId (FK to Stations)
- UserId (FK to Users)
- Rating (1-5)
- Comment
- Timestamp

### PinRequests Table
- Id (PK)
- UserId (FK to Users)
- Latitude
- Longitude
- Reason
- Status (enum: "pending", "approved", "rejected")
- CreatedAt
- ReviewedAt (optional)
- ReviewedBy (FK to Users, optional)

### Ads Table
- Id (PK)
- ImageUrl
- Link
- StartDate
- EndDate
- Active
- Impressions
- Clicks

## UI/UX Design System

The UI/UX design system will be documented in detail in the UI-UX-Guidelines directory, covering:

1. **Design Principles**: Core principles guiding the UI/UX design
2. **Color System**: Primary, secondary, and accent colors with accessibility considerations
3. **Typography**: Font families, sizes, and weights for different contexts
4. **Spacing System**: Consistent spacing units for margins and padding
5. **Component Library**: Reusable UI components with usage guidelines
6. **Responsive Patterns**: Mobile-first design patterns and breakpoints
7. **Accessibility Guidelines**: WCAG compliance and best practices
8. **Interaction Patterns**: Standard interactions for common tasks

This structure provides a solid foundation for the SriCharge MVP while allowing for future expansion and feature additions.
