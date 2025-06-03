# SriCharge MVP Setup Instructions

This document provides detailed setup instructions for the SriCharge MVP project, which uses Angular for the frontend and .NET Core for the backend.

## Prerequisites

- Node.js 16.x or later
- npm 8.x or later
- .NET 7.0 SDK or later
- Visual Studio 2022 or Visual Studio Code
- SQL Server (or alternative database supported by EF Core)

## Backend Setup (.NET Core)

Since the .NET CLI may not be available in all environments, follow these steps to initialize the backend project:

1. Open a terminal/command prompt
2. Navigate to the `SriCharge.API` directory
3. Run the following command to create a new Web API project:

```bash
dotnet new webapi -n SriCharge.API --no-https
```

4. Install required NuGet packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

5. Copy the provided entity models, controllers, and services into the appropriate directories
6. Update the connection string in `appsettings.json` to point to your database
7. Run the following commands to create the database:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

8. Start the API:

```bash
dotnet run
```

## Frontend Setup (Angular)

1. Open a terminal/command prompt
2. Navigate to the `SriCharge.Web` directory
3. Run the following command to create a new Angular project:

```bash
ng new sricharge-web --routing --style=scss
```

4. Install required npm packages:

```bash
npm install @angular/material @angular/cdk @angular/flex-layout
npm install @angular/pwa @angular/service-worker
npm install leaflet @types/leaflet
```

5. Copy the provided components, services, and modules into the appropriate directories
6. Update the environment files to point to your API endpoint
7. Start the Angular development server:

```bash
ng serve
```

## Project Structure

The project follows the structure outlined in `PROJECT_STRUCTURE.md`, with:

- Backend (.NET Core API)
  - Controllers for all required endpoints
  - Entity Framework Core data models
  - Repository pattern for data access
  - Service layer for business logic

- Frontend (Angular PWA)
  - Feature-based module organization
  - Core authentication and shared services
  - Role-based layouts for different user types
  - Component library structure

## UI/UX Guidelines

The UI/UX guidelines are provided in the `UI-UX-Guidelines` directory. These guidelines are essential for developers with limited UI/UX experience and should be followed closely to ensure a consistent, accessible, and mobile-friendly user experience.

## Next Steps

After setting up the project:

1. Review the `todo.md` file for a list of tasks to complete
2. Study the UI/UX guidelines to understand the design principles
3. Implement the remaining features according to the project structure
4. Test the application on various devices and screen sizes
5. Deploy the application to your preferred hosting environment
