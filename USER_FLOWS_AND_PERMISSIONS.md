# SriCharge User Flows and Permissions

This document outlines the complete user flows and permissions for each role in the SriCharge application. It serves as a comprehensive reference for developers implementing the role-based functionality.

## Role Overview

SriCharge has four distinct user roles, each with specific permissions and capabilities:

1. **General User** - Basic app users who can view chargers, submit reviews, and request new locations
2. **Verifier** - Trusted users who can confirm charger status when nearby and earn credibility points
3. **Charger Owner** - Users who manage charging stations, update status, and respond to reviews
4. **Admin** - System administrators who manage users, approve content, and oversee the platform

## Permission Matrix

| Feature/Action | General User | Verifier | Charger Owner | Admin |
|----------------|--------------|----------|---------------|-------|
| **Map Viewing** |
| View map with chargers | ✓ | ✓ | ✓ | ✓ |
| Filter chargers by type | ✓ | ✓ | ✓ | ✓ |
| View charger details | ✓ | ✓ | ✓ | ✓ |
| **Charger Interaction** |
| Submit reviews/ratings | ✓ | ✓ | ✓ | ✓ |
| Request new charger location | ✓ | ✓ | ✓ | ✓ |
| Verify charger status | ✗ | ✓ | ✓ | ✓ |
| Receive verification prompts | ✗ | ✓ | ✗ | ✗ |
| Earn credibility points | ✗ | ✓ | ✗ | ✗ |
| **Charger Management** |
| Add new chargers | ✗ | ✗ | ✓ | ✓ |
| Edit charger details | ✗ | ✗ | ✓ | ✓ |
| Update charger status | ✗ | ✗ | ✓ | ✓ |
| View station statistics | ✗ | ✗ | ✓ | ✓ |
| Respond to reviews | ✗ | ✗ | ✓ | ✓ |
| **Administrative** |
| Manage users | ✗ | ✗ | ✗ | ✓ |
| Approve/reject charger requests | ✗ | ✗ | ✗ | ✓ |
| Assign user roles | ✗ | ✗ | ✗ | ✓ |
| View system dashboard | ✗ | ✗ | ✗ | ✓ |
| Manage banner ads | ✗ | ✗ | ✗ | ✓ |

## User Flows

### General User Flows

1. **Map Exploration Flow**
   - User opens app → Map view loads with nearby chargers
   - User can filter by connector type, availability, or rating
   - User taps on charger marker → Charger details view opens
   - User can view status, specifications, reviews, and directions

2. **Review Submission Flow**
   - User views charger details → Taps "Leave Review"
   - User enters rating and comments → Submits review
   - System confirms submission → Updates charger rating

3. **New Charger Request Flow**
   - User taps "Request New Charger" → Request form opens
   - User enters location and details → Submits request
   - System confirms submission → Admin receives notification for review

### Verifier Flows

1. **Verification Prompt Flow**
   - Verifier comes within range of charger → Receives notification
   - Verifier taps notification → Verification prompt opens
   - Verifier confirms status (available/in-use/broken) → Submits verification
   - System awards credibility points → Updates charger status

2. **Verification Dashboard Flow**
   - Verifier taps "Verification" tab → Dashboard opens
   - Verifier sees nearby verification opportunities
   - Verifier views verification history and credibility score
   - Verifier can filter by distance or charger type

### Charger Owner Flows

1. **Station Management Flow**
   - Owner logs in → Station management dashboard opens
   - Owner sees list of owned stations with status overview
   - Owner can add new station or select existing station to manage

2. **Add/Edit Station Flow**
   - Owner taps "Add Station" → New station form opens
   - Owner enters details (location, type, availability) → Submits for approval
   - OR: Owner selects existing station → Edit form opens
   - Owner updates details → Saves changes

3. **Review Management Flow**
   - Owner views station details → Taps "Manage Reviews"
   - Owner sees list of reviews with ratings
   - Owner can respond to reviews or flag inappropriate content

4. **Statistics Viewing Flow**
   - Owner selects station → Taps "View Statistics"
   - Owner sees usage data, revenue, and user engagement metrics
   - Owner can filter by time period and export reports

### Admin Flows

1. **Dashboard Overview Flow**
   - Admin logs in → Admin dashboard opens
   - Admin sees platform metrics, system status, and recent activity
   - Admin can navigate to specific management interfaces

2. **User Management Flow**
   - Admin taps "User Management" → User list opens
   - Admin can search, filter, and sort users
   - Admin selects user → User details modal opens
   - Admin can change roles, update status, or send notifications

3. **Charger Approval Flow**
   - Admin views pending approvals → Selects charger request
   - Admin reviews details and location → Approves or rejects
   - System notifies requestor of decision

## Role Transition Flows

1. **Verifier Application Flow**
   - General User applies for Verifier role → Submits application
   - Admin reviews application → Approves or rejects
   - If approved, user receives notification and new permissions

2. **Charger Owner Registration Flow**
   - User requests Charger Owner role → Submits business details
   - Admin reviews details → Approves or rejects
   - If approved, user receives notification and access to owner dashboard

## Error Handling and Edge Cases

1. **Offline Mode**
   - All users can view cached map and charger data when offline
   - Actions requiring server connection are queued for when connectivity returns

2. **Verification Conflicts**
   - If multiple verifiers submit conflicting status reports, system uses credibility score to determine priority
   - Admin is notified of conflicts for manual review

3. **Account Recovery**
   - All users can reset password via email
   - Admins can manually reset user passwords or unlock accounts

4. **Suspended Accounts**
   - Suspended users receive clear notification about suspension reason
   - Appeal process available through customer support

## Implementation Considerations

1. **Role-Based Access Control**
   - Implement both client-side and server-side permission checks
   - Use Firebase Authentication custom claims for role storage
   - Enforce Firestore security rules based on user role

2. **UI Adaptation**
   - Navigation elements should adapt based on user role
   - Hide functionality that isn't available to current role
   - Provide clear feedback when permission is denied

3. **Role Transitions**
   - Handle role changes gracefully without requiring re-login
   - Update UI immediately when role changes
   - Provide onboarding for new capabilities when role is upgraded
