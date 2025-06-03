# SriCharge MVP Features Implementation Guide

## Overview
This document provides a comprehensive guide to the five core MVP features implemented for the SriCharge application. Each feature has been built on top of the existing Next.js + TypeScript + TailwindCSS and Firebase scaffold, following mobile-first design principles and PWA best practices.

## Features Implemented

### 1. Verifier Logic (Location-triggered + Credibility Score)
The Verifier Logic feature enables users with the Verifier role to confirm charger status when they are physically near a charging station. The system tracks their verification accuracy and awards credibility points accordingly.

**Key Components:**
- `VerifierLocationTracker.tsx`: Monitors verifier's location and triggers prompts when near chargers
- `VerifierPrompt.tsx`: UI for verifiers to confirm charger status
- `verifierFunctions.ts`: Cloud Functions for credibility score calculation
- Firestore `verifications` collection with submission throttling

**How It Works:**
1. When a verifier is within 100 meters of a charger, they receive a prompt
2. Verifier selects the current status (Available, In Use, Broken)
3. System prevents multiple submissions for the same charger within 6 hours
4. Cloud Function compares with other verifications and adjusts credibility score
5. Station status is updated in real-time

### 2. Pin-Drop Request Flow (User + Admin)
This feature allows general users to request new charging stations by dropping pins on the map, and admins to review, approve, or reject these requests.

**Key Components:**
- `PinDropMap.tsx`: Interactive map for selecting locations
- `PinRequestForm.tsx`: Form for submitting new charger requests
- `PinRequestManagement.tsx`: Admin dashboard for managing requests
- Firestore `pinRequests` collection with approval workflow

**How It Works:**
1. User selects a location on the map and submits a request with reason and optional image
2. Request is saved to Firestore with PENDING status
3. Admin reviews requests in the dashboard
4. Admin can approve (converting to a station) or reject requests
5. Approved requests create new entries in the stations collection

### 3. Charger Rating with Amenities
This feature enhances the review system to include amenities selection, allowing users to indicate what facilities are available at charging stations.

**Key Components:**
- `ReviewForm.tsx`: Enhanced form with amenities selection
- `ReviewList.tsx`: Display of reviews with amenities and average rating
- Cloud Function for calculating average ratings and top amenities

**How It Works:**
1. Users can rate stations (1-5 stars) and select available amenities
2. Reviews are stored with amenities data in Firestore
3. System calculates average rating and identifies top amenities
4. Station cards display average rating and most common amenities
5. Users can filter stations by amenities

### 4. Admin Heatmap Dashboard
The Heatmap Dashboard provides admins with visualization of charger demand and usage patterns across geographic areas.

**Key Components:**
- `HeatmapDashboard.tsx`: Interactive heatmap visualization
- Cloud Functions for data aggregation and caching
- Firestore `dashboardStats` collection for performance optimization

**How It Works:**
1. System aggregates pin requests and verification data by location
2. Data is processed into heatmap format and cached in Firestore
3. Admins can view heatmap with different time ranges and data types
4. Dashboard identifies hotspot areas with high demand
5. Data updates daily via scheduled Cloud Functions

### 5. Charger Owner Promotions
This feature allows charging station owners to submit promotional content that appears as banner cards or highlighted pins on the map.

**Key Components:**
- `PromotionForm.tsx`: Form for owners to submit promotions
- `PromotionManagement.tsx`: Admin interface for approving/rejecting promotions
- `PromotionBanner.tsx`: Display component for approved promotions
- Firestore `promotions` collection with approval workflow

**How It Works:**
1. Station owners create promotions with message, image, and date range
2. Promotions are submitted for admin approval
3. Admins review and approve/reject promotions
4. Approved promotions appear as banners on the map view
5. System automatically expires promotions after end date

## Technical Implementation

### Firestore Schema Updates
The following collections have been added or modified:

1. **verifications**
   - stationId: string
   - userId: string
   - previousStatus: ChargerStatus
   - newStatus: ChargerStatus
   - timestamp: Timestamp
   - location: GeoPoint
   - distance: number

2. **pinRequests**
   - userId: string
   - location: GeoPoint
   - reason: string
   - status: RequestStatus
   - createdAt: Timestamp
   - reviewedAt?: Timestamp
   - reviewedBy?: string
   - imageUrl?: string

3. **reviews** (updated)
   - stationId: string
   - userId: string
   - rating: number
   - comment: string
   - timestamp: Timestamp
   - amenities?: string[]

4. **dashboardStats**
   - heatmap data
   - request counts
   - verification counts
   - hotspot areas
   - lastUpdated: Timestamp

5. **promotions**
   - stationId: string
   - ownerId: string
   - message: string
   - imageUrl: string
   - startDate: Timestamp
   - endDate: Timestamp
   - status: RequestStatus
   - createdAt: Timestamp
   - reviewedAt?: Timestamp
   - reviewedBy?: string

### Cloud Functions
Several Cloud Functions have been implemented to handle backend logic:

1. **updateVerifierCredibility**: Adjusts verifier credibility scores based on agreement with other verifications
2. **updateStationReviewStats**: Calculates average ratings and top amenities when reviews are submitted
3. **updateDashboardStats**: Aggregates data for the heatmap dashboard (runs daily)
4. **cleanupOldVerifications**: Removes verifications older than 90 days (runs daily)
5. **handlePromotionStatusChange**: Manages notifications when promotion status changes
6. **cleanupExpiredPromotions**: Deactivates promotions past their end date (runs daily)

### Security Rules
Firestore security rules have been updated to ensure:

1. Verifiers can only submit verifications when within proximity of stations
2. Users can only submit one verification per station within 6 hours
3. Only station owners can submit promotions for their stations
4. Only admins can approve/reject pin requests and promotions
5. Public read access for approved promotions and station data

## Mobile-First and PWA Considerations

All features have been implemented with mobile-first design principles:
- Touch-friendly UI elements
- Responsive layouts that adapt to different screen sizes
- Efficient use of screen real estate
- Location services integration
- Offline support via service workers

The application maintains PWA reliability through:
- Caching strategies for offline access
- Background sync for pending submissions
- Minimal network requests
- Progressive enhancement

## Integration with Existing Scaffold
All new features integrate seamlessly with the existing scaffold:
- Leveraging the authentication and role-based access system
- Extending the existing Firestore schema
- Maintaining consistent UI patterns
- Following the established project structure

## Conclusion
These five core MVP features significantly enhance the SriCharge application, providing a comprehensive solution for EV charging in Sri Lanka. The implementation follows best practices for Firebase, TypeScript, mobile-first design, and PWA reliability, ensuring a robust and scalable application.
