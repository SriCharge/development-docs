# Development Guide for SriCharge MVP

This guide provides detailed instructions for junior developers to continue building on the SriCharge MVP scaffold.

## Environment Setup

1. **Node.js and npm**: Ensure you have Node.js 18+ installed
2. **Code Editor**: VS Code with the following extensions is recommended:
   - ESLint
   - Prettier
   - Tailwind CSS IntelliSense
   - Firebase Explorer
3. **Firebase Tools**: Install Firebase CLI globally
   ```bash
   npm install -g firebase-tools
   ```

## Project Structure Overview

The project follows a modular structure organized by feature and role. Key directories:

- `src/components/`: UI components organized by feature
- `src/contexts/`: React contexts for state management
- `src/firebase/`: Firebase configuration and utility functions
- `src/layouts/`: Role-based layout components
- `src/pages/`: Next.js page components

## Getting Started with Development

### 1. Authentication Flow

The authentication flow is handled by `AuthContext.tsx` and `RoleContext.tsx`. To implement new authentication features:

```tsx
// Example: Using authentication in a component
import { useAuth } from '../contexts/AuthContext';
import { useRole } from '../contexts/RoleContext';

const MyComponent = () => {
  const { currentUser, userProfile } = useAuth();
  const { isAdmin, isOwner, isVerifier } = useRole();
  
  // Conditional rendering based on role
  if (isAdmin) {
    return <AdminFeature />;
  }
  
  return <RegularFeature />;
};
```

### 2. Working with Firestore

Use the utility functions in `src/firebase/firestore.ts` to interact with Firestore:

```tsx
// Example: Fetching and displaying chargers
import { useState, useEffect } from 'react';
import { getNearbyStations, Station } from '../firebase/firestore';
import { GeoPoint } from 'firebase/firestore';

const NearbyChargers = () => {
  const [stations, setStations] = useState<Station[]>([]);
  
  useEffect(() => {
    const fetchStations = async () => {
      const userLocation = new GeoPoint(7.8731, 80.7718); // Example coordinates
      const nearbyStations = await getNearbyStations(userLocation, 10);
      setStations(nearbyStations);
    };
    
    fetchStations();
  }, []);
  
  return (
    <div>
      {stations.map(station => (
        <ChargerCard key={station.id} {...station} />
      ))}
    </div>
  );
};
```

### 3. Adding New Pages

To add a new page:

1. Create a new file in the appropriate directory under `src/pages/`
2. Use the correct layout based on the intended user role
3. Import necessary components and hooks

```tsx
// Example: Creating a new page for charger details
import { useRouter } from 'next/router';
import { useState, useEffect } from 'react';
import UserLayout from '../../layouts/UserLayout';
import ChargerDetails from '../../components/chargers/ChargerDetails';
import { getStation, Station } from '../../firebase/firestore';

export default function ChargerDetailPage() {
  const router = useRouter();
  const { id } = router.query;
  const [station, setStation] = useState<Station | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    if (id) {
      const fetchStation = async () => {
        const stationData = await getStation(id as string);
        setStation(stationData);
        setLoading(false);
      };
      
      fetchStation();
    }
  }, [id]);
  
  if (loading) {
    return (
      <UserLayout>
        <div className="flex justify-center py-8">
          <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-green-500"></div>
        </div>
      </UserLayout>
    );
  }
  
  if (!station) {
    return (
      <UserLayout>
        <div className="p-4 bg-red-50 text-red-700 rounded-md">
          Charger not found
        </div>
      </UserLayout>
    );
  }
  
  return (
    <UserLayout>
      <ChargerDetails station={station} />
    </UserLayout>
  );
}
```

### 4. Mobile-First Development

Always design for mobile first, then enhance for larger screens:

```tsx
// Example: Mobile-first responsive component
const ResponsiveComponent = () => {
  return (
    <div>
      {/* Mobile layout (default) */}
      <div className="block md:hidden">
        <MobileView />
      </div>
      
      {/* Desktop layout */}
      <div className="hidden md:block">
        <DesktopView />
      </div>
    </div>
  );
};
```

### 5. Testing Components

Follow the example in `src/components/chargers/__tests__/ChargerCard.test.tsx` for testing:

```tsx
// Example: Testing a new component
import { render, screen } from '@testing-library/react';
import MyComponent from '../MyComponent';

describe('MyComponent', () => {
  test('renders correctly', () => {
    render(<MyComponent />);
    expect(screen.getByText('Expected Text')).toBeInTheDocument();
  });
});
```

## Common Tasks

### Implementing a New Feature

1. **Plan**: Define the feature requirements and user flow
2. **Component Design**: Create or modify components in the appropriate directories
3. **State Management**: Use React hooks and context as needed
4. **Firebase Integration**: Use the existing Firebase utility functions
5. **Testing**: Write tests for the new components
6. **Documentation**: Update documentation as needed

### Debugging Tips

1. Use React DevTools for component inspection
2. Check Firebase console for authentication and database issues
3. Use browser developer tools for network and console errors
4. For PWA issues, check the Application tab in Chrome DevTools

## Best Practices

1. **TypeScript**: Always define interfaces for props and state
2. **Component Structure**: Keep components focused and reusable
3. **Firebase Security**: Always respect security rules and user roles
4. **Performance**: Optimize renders and Firebase queries
5. **Accessibility**: Ensure components are accessible
6. **Mobile-First**: Design for mobile first, then enhance for larger screens

## Deployment

### Development Deployment

For local testing:

```bash
npm run dev
```

### Production Build

For production:

```bash
npm run build
npm run start
```

### Firebase Deployment

To deploy to Firebase Hosting:

```bash
firebase login
firebase init  # If not already initialized
firebase deploy
```

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Firebase Documentation](https://firebase.google.com/docs)
- [TailwindCSS Documentation](https://tailwindcss.com/docs)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
