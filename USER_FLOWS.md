# SriCharge User Flow Diagrams

This document provides visual representations of the key user flows for each role in the SriCharge application. These diagrams illustrate the step-by-step interactions users will experience when performing common tasks.

## General User Flows

### Finding and Using a Charging Station

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│   Map View  │────▶│ Apply Filter│────▶│ Browse Map  │
│             │     │             │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│Submit Review│◀────│  Check In   │◀────│Get Directions│◀────│Select Marker│
│ (Optional)  │     │ (Optional)  │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Requesting a New Charger

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│   Map View  │────▶│ Request New │────▶│Select Location│
│             │     │             │     │   Charger   │     │   on Map    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│ Confirmation│◀────│   Submit    │◀────│ Add Photos  │◀────│ Fill Details │
│   Screen    │     │   Request   │     │ (Optional)  │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Submitting a Review

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│Charger Details│──▶│Write Review │────▶│ Rate Station │
│             │     │             │     │             │     │  (1-5 stars) │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│Review Posted│◀────│   Submit    │◀────│ Add Photos  │◀────│Write Comments│
│             │     │   Review    │     │ (Optional)  │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

## Verifier Flows

### Responding to Verification Prompt

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Receive    │────▶│   Open App  │────▶│ Verification│────▶│Review Station│
│Notification │     │             │     │   Prompt    │     │   Details   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Receive    │◀────│   Submit    │◀────│ Add Photos  │◀────│Confirm/Update│
│   Points    │     │Verification │     │ and Notes   │     │   Status    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Checking Verification Status

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│Verification │────▶│View Credibility│─▶│Check Recent │
│             │     │ Dashboard   │     │    Score    │     │Verifications│
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
                                                            ┌─────────────┐
                                                            │             │
                                                            │   View      │
                                                            │Leaderboard  │
                                                            └─────────────┘
```

## Charger Owner Flows

### Adding a New Station

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│   Station   │────▶│  Add New    │────▶│Enter Station│
│             │     │ Dashboard   │     │  Station    │     │  Details    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│ Confirmation│◀────│   Submit    │◀────│Set Operating│◀────│Upload Photos│
│   Screen    │     │ for Approval│     │   Hours     │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Updating Station Status

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│   Station   │────▶│Select Station│───▶│Choose Status│
│             │     │ Dashboard   │     │  to Update  │     │   Option    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
                                                            ┌─────────────┐
                                                            │             │
                                                            │ Confirmation│
                                                            │   Screen    │
                                                            └─────────────┘
```

### Responding to Reviews

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Receive    │────▶│   Open App  │────▶│Review Management│─▶│ Read User  │
│Notification │     │             │     │    Screen    │     │   Review   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐
│             │     │             │
│Response Posted│◀──│Write Response│
│             │     │             │
└─────────────┘     └─────────────┘
```

## Admin Flows

### Approving a New Station

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│    Admin    │────▶│   Pending   │────▶│Review Station│
│             │     │  Dashboard  │     │  Approvals  │     │   Details   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │
│Notification │◀────│Approve/Reject│◀───│ Edit Details│
│   Sent      │     │   Station   │     │ (Optional)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

### Managing User Roles

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│    Admin    │────▶│    User     │────▶│ Search/Find │
│             │     │  Dashboard  │     │ Management  │     │    User     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │
│Confirmation │◀────│Save Changes │◀────│Change User  │
│   Screen    │     │             │     │    Role     │
└─────────────┘     └─────────────┘     └─────────────┘
```

### Creating an Ad Campaign

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│  Open App   │────▶│    Admin    │────▶│ Banner Ad   │────▶│ Create New  │
│             │     │  Dashboard  │     │ Management  │     │  Campaign   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│Campaign Live│◀────│   Publish   │◀────│Set Schedule │◀────│Upload Banner│
│             │     │  Campaign   │     │ & Targeting │     │   Image     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

## Cross-Role Interactions

### Charger Verification Process

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│Owner Submits│────▶│Admin Reviews│────▶│Admin Approves│───▶│Station Added│
│New Station  │     │  Station    │     │   Station   │     │   to Map    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│Status Updated│◀───│Verifier Submits│◀─│Verifier Visits│◀──│Verifier Gets│
│  in System  │     │Verification  │     │  Station    │     │ Notification│
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### User Request to Station Creation

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │     │             │
│User Submits │────▶│Admin Reviews│────▶│Admin Approves│───▶│Owner Notified│
│  Request    │     │  Request    │     │   Request   │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │
│Station Added│◀────│Admin Approves│◀───│Owner Creates │
│   to Map    │     │   Station   │     │  Station    │
└─────────────┘     └─────────────┘     └─────────────┘
```

## Implementation Notes

1. **Error Handling**: Each flow should include appropriate error states and recovery paths
2. **Loading States**: Implement loading indicators for operations that may take time
3. **Confirmation**: Use confirmation dialogs for destructive or important actions
4. **Offline Support**: Design flows to handle offline scenarios gracefully
5. **Accessibility**: Ensure all flows can be completed using keyboard navigation and screen readers
6. **Progressive Enhancement**: Core flows should work even with limited JavaScript support
