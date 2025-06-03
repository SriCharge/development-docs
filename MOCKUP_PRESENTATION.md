# SriCharge Mockup Presentation for Development Agency

## Introduction

This document serves as a comprehensive presentation of the SriCharge mobile-first Progressive Web App (PWA) mockups and specifications. It is designed to provide your development team with a clear understanding of the application's design, functionality, and technical requirements.

## Project Overview

SriCharge is a mobile-first PWA that unifies the EV charging experience in Sri Lanka by providing:

- Real-time charger location and status information
- User-submitted reviews and ratings
- Verification system for data accuracy
- Role-based access for different user types
- Administrative tools for platform management

## User Roles and Permissions

The application supports four distinct user roles, each with specific capabilities:

1. **General User**
   - View map with charging stations
   - Check real-time charger status
   - Submit reviews and ratings
   - Request new charger locations

2. **Verifier**
   - All General User capabilities
   - Verify charger status when nearby
   - Earn credibility points for accurate verifications
   - View verification history and statistics

3. **Charger Owner**
   - Add and manage charging stations
   - Update station status and details
   - Respond to user reviews
   - View station usage statistics

4. **Admin**
   - Manage user accounts and roles
   - Approve or reject new station submissions
   - Moderate content and reviews
   - Configure system settings
   - Manage promotional content

## Design System

The SriCharge application follows a consistent design system with these key elements:

### Color Palette

- **Primary**: #3E64FF (Blue) - Main brand color, used for primary actions and navigation
- **Secondary**: #28A745 (Green) - Used for success states and available status
- **Accent**: #FFC107 (Yellow) - Used for warnings and in-use status
- **Danger**: #DC3545 (Red) - Used for errors and unavailable status
- **Neutral**: #6C757D (Gray) - Used for text and inactive elements
- **Background**: #F8F9FA (Light Gray) - Used for page backgrounds
- **Surface**: #FFFFFF (White) - Used for cards and content areas

### Typography

- **Primary Font**: Poppins
- **Headings**: Poppins Bold (24px, 18px, 16px)
- **Body**: Poppins Regular (14px)
- **Labels**: Poppins Medium (14px)
- **Captions**: Poppins Regular (12px)

### Spacing System

- Base unit: 8px
- Spacing scale: 8px, 16px, 24px, 32px, 48px, 64px

### Component Library

The application uses a consistent set of components across all screens:

- Buttons (primary, secondary, text)
- Input fields (text, select, checkbox, radio)
- Cards (station, review, verification)
- Navigation elements (tabs, bottom bar, sidebar)
- Modals and dialogs
- Maps and markers
- Status indicators

## Key Screens by User Role

### General User

1. **Map View**
   - Interactive map with station markers
   - Filter controls for charger types and availability
   - Current location indicator
   - Search functionality

2. **Charger Details**
   - Station information and photos
   - Real-time availability status
   - Reviews and ratings
   - Directions and reporting options

3. **Review Submission**
   - Star rating input
   - Comment field
   - Photo upload option

4. **New Charger Request**
   - Location selection on map
   - Charger type and details form
   - Reason for request

### Verifier

1. **Verification Dashboard**
   - Credibility score display
   - Nearby verification opportunities
   - Verification history
   - Leaderboard position

2. **Verification Prompt**
   - Station details to verify
   - Current reported status
   - Options to confirm or update status
   - Photo upload capability

3. **Verification Form**
   - Status selection
   - Connector type confirmation
   - Additional notes field

### Charger Owner

1. **Station Management Dashboard**
   - Overview of all owned stations
   - Status indicators and quick actions
   - Alerts for issues or new reviews

2. **Add/Edit Station Form**
   - Location selection on map
   - Comprehensive station details form
   - Photo upload capability
   - Operating hours and payment information

3. **Station Statistics**
   - Usage metrics and charts
   - Review summary
   - Performance indicators

### Admin

1. **Admin Dashboard**
   - System overview statistics
   - Pending approvals count
   - User registration metrics
   - Quick action buttons

2. **User Management Interface**
   - User list with search and filter
   - Role assignment controls
   - Account status management

3. **Charger Approval Interface**
   - Queue of pending station submissions
   - Detailed view of each submission
   - Approval/rejection controls
   - Edit capability before approval

4. **Banner Ad Management**
   - Campaign creation and scheduling
   - Performance metrics
   - Targeting options

5. **System Settings**
   - Application configuration
   - Map and feature settings
   - User role management
   - Integration configuration

## User Flows

The application supports several key user flows:

### General User Flows
- Finding and using a charging station
- Requesting a new charger location
- Submitting reviews and ratings

### Verifier Flows
- Responding to verification prompts
- Checking verification status and history
- Proactively verifying nearby stations

### Charger Owner Flows
- Adding new charging stations
- Updating station status and details
- Responding to user reviews

### Admin Flows
- Approving new station submissions
- Managing user roles and permissions
- Creating and monitoring ad campaigns
- Configuring system settings

## Technical Implementation

### Frontend Technologies
- **Framework**: Next.js with React
- **Styling**: CSS-in-JS or TailwindCSS
- **State Management**: React Context API and hooks
- **Maps**: Google Maps or Mapbox

### Backend Technologies
- **Authentication**: Firebase Authentication
- **Database**: Firestore
- **Storage**: Firebase Storage
- **Functions**: Firebase Cloud Functions
- **Hosting**: Firebase Hosting

### Progressive Web App Features
- Offline support with service workers
- Add to home screen capability
- Push notifications for verifiers
- Responsive design for all device sizes

## Component Specifications

Detailed specifications for key components are provided in the COMPONENT_SPECIFICATIONS.md document, including:

- Props and state management
- Visual specifications
- Accessibility requirements
- Mobile adaptations
- Firebase integration points

## Next Steps for Development

1. **Environment Setup**
   - Configure Next.js project with TypeScript
   - Set up Firebase project and services
   - Implement authentication and role management

2. **Core Functionality**
   - Implement map integration with station markers
   - Create station detail views and cards
   - Build review system and forms
   - Develop verification functionality

3. **Role-Based Features**
   - Implement owner station management
   - Create admin interfaces and tools
   - Build verifier notification system

4. **Progressive Enhancement**
   - Add offline support and caching
   - Implement push notifications
   - Optimize performance and loading

5. **Testing and Deployment**
   - Conduct cross-device testing
   - Implement analytics and monitoring
   - Deploy to Firebase Hosting

## Conclusion

These mockups and specifications provide a comprehensive blueprint for developing the SriCharge application. The design system, component specifications, and user flows are designed to create a cohesive, user-friendly experience across all roles and devices.

The development team should refer to the detailed documentation in the following files for implementation guidance:

- USER_FLOWS.md - Detailed user journey diagrams
- COMPONENT_SPECIFICATIONS.md - Technical component details
- role-specifications.md - Comprehensive role capabilities
- design-system.md - Visual design guidelines

Each user role directory contains detailed mockups for specific screens and functionality.
