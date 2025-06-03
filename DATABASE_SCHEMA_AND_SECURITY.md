# Database Schema and Security Model for SriCharge

This document provides a comprehensive database schema design and security model for the SriCharge application, using Azure SQL Database and Azure Active Directory B2C for authentication.

## Database Schema

### Entity Relationship Diagram

```
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│     Users     │       │   Stations    │       │    Reviews    │
├───────────────┤       ├───────────────┤       ├───────────────┤
│ Id            │       │ Id            │       │ Id            │
│ Name          │       │ Name          │       │ StationId     │
│ Email         │       │ Location      │       │ UserId        │
│ Role          │       │ Type          │       │ Rating        │
│ CredScore     │       │ Status        │       │ Comment       │
│ CreatedAt     │◄──┐   │ OwnerId       │◄──┐   │ Timestamp     │
└───────────────┘   │   │ VerifiedBy    │   │   │ HelpfulCount  │
                    │   │ LastVerified  │   │   └───────┬───────┘
                    │   └───────┬───────┘   │           │
                    │           │           │           │
                    │           │           │           │
┌───────────────┐   │   ┌───────┴───────┐   │   ┌───────┴───────┐
│  PinRequests  │   │   │StationVerifier│   │   │VerifierPrompts│
├───────────────┤   │   ├───────────────┤   │   ├───────────────┤
│ Id            │   │   │ Id            │   │   │ Id            │
│ UserId        │───┘   │ StationId     │───┘   │ StationId     │
│ Location      │       │ UserId        │───────│ UserId        │
│ Reason        │       │ Timestamp     │       │ Status        │
│ Status        │       │ Status        │       │ CreatedAt     │
│ CreatedAt     │       └───────────────┘       │ ExpiresAt     │
└───────────────┘                               └───────────────┘

┌───────────────┐
│      Ads      │
├───────────────┤
│ Id            │
│ ImageUrl      │
│ Link          │
│ StartDate     │
│ EndDate       │
│ IsActive      │
└───────────────┘
```

### Table Definitions

#### Users

Stores user information and role assignments.

```sql
CREATE TABLE Users (
    Id NVARCHAR(128) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Email NVARCHAR(100) NOT NULL UNIQUE,
    Role NVARCHAR(20) NOT NULL CHECK (Role IN ('User', 'Verifier', 'Owner', 'Admin')),
    CredibilityScore INT NOT NULL DEFAULT 100,
    ProfileImageUrl NVARCHAR(255) NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    LastLogin DATETIME2 NULL
);
```

#### Stations

Stores charging station information.

```sql
CREATE TABLE Stations (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Name NVARCHAR(100) NOT NULL,
    Address NVARCHAR(255) NOT NULL,
    Latitude DECIMAL(9,6) NOT NULL,
    Longitude DECIMAL(9,6) NOT NULL,
    Type NVARCHAR(50) NOT NULL,
    PowerKW DECIMAL(5,2) NOT NULL,
    Status NVARCHAR(20) NOT NULL CHECK (Status IN ('Available', 'In-Use', 'Broken')),
    OwnerId NVARCHAR(128) NULL,
    VerifiedBy NVARCHAR(128) NULL,
    LastVerified DATETIME2 NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (OwnerId) REFERENCES Users(Id) ON DELETE SET NULL,
    FOREIGN KEY (VerifiedBy) REFERENCES Users(Id) ON DELETE SET NULL
);

CREATE INDEX IX_Stations_Location ON Stations(Latitude, Longitude);
CREATE INDEX IX_Stations_Status ON Stations(Status);
CREATE INDEX IX_Stations_OwnerId ON Stations(OwnerId);
```

#### Reviews

Stores user reviews for charging stations.

```sql
CREATE TABLE Reviews (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    StationId UNIQUEIDENTIFIER NOT NULL,
    UserId NVARCHAR(128) NOT NULL,
    Rating INT NOT NULL CHECK (Rating BETWEEN 1 AND 5),
    Comment NVARCHAR(1000) NULL,
    HelpfulCount INT NOT NULL DEFAULT 0,
    Timestamp DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (StationId) REFERENCES Stations(Id) ON DELETE CASCADE,
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE
);

CREATE INDEX IX_Reviews_StationId ON Reviews(StationId);
CREATE INDEX IX_Reviews_UserId ON Reviews(UserId);
```

#### PinRequests

Stores requests for new charging station locations.

```sql
CREATE TABLE PinRequests (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserId NVARCHAR(128) NOT NULL,
    Latitude DECIMAL(9,6) NOT NULL,
    Longitude DECIMAL(9,6) NOT NULL,
    Reason NVARCHAR(500) NULL,
    Status NVARCHAR(20) NOT NULL CHECK (Status IN ('Pending', 'Approved', 'Rejected')),
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    ProcessedAt DATETIME2 NULL,
    ProcessedBy NVARCHAR(128) NULL,
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE,
    FOREIGN KEY (ProcessedBy) REFERENCES Users(Id) ON DELETE SET NULL
);

CREATE INDEX IX_PinRequests_Status ON PinRequests(Status);
CREATE INDEX IX_PinRequests_UserId ON PinRequests(UserId);
```

#### StationVerifier

Tracks which users have verified which stations.

```sql
CREATE TABLE StationVerifier (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    StationId UNIQUEIDENTIFIER NOT NULL,
    UserId NVARCHAR(128) NOT NULL,
    Status NVARCHAR(20) NOT NULL CHECK (Status IN ('Available', 'In-Use', 'Broken')),
    Timestamp DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (StationId) REFERENCES Stations(Id) ON DELETE CASCADE,
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE
);

CREATE INDEX IX_StationVerifier_StationId ON StationVerifier(StationId);
CREATE INDEX IX_StationVerifier_UserId ON StationVerifier(UserId);
```

#### VerifierPrompts

Tracks prompts sent to verifiers when they are near a station.

```sql
CREATE TABLE VerifierPrompts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    StationId UNIQUEIDENTIFIER NOT NULL,
    UserId NVARCHAR(128) NOT NULL,
    Status NVARCHAR(20) NOT NULL CHECK (Status IN ('Pending', 'Completed', 'Ignored')),
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    ExpiresAt DATETIME2 NOT NULL,
    CompletedAt DATETIME2 NULL,
    FOREIGN KEY (StationId) REFERENCES Stations(Id) ON DELETE CASCADE,
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE
);

CREATE INDEX IX_VerifierPrompts_Status ON VerifierPrompts(Status);
CREATE INDEX IX_VerifierPrompts_UserId ON VerifierPrompts(UserId);
```

#### Ads

Stores banner advertisements for the application.

```sql
CREATE TABLE Ads (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    ImageUrl NVARCHAR(255) NOT NULL,
    Link NVARCHAR(255) NOT NULL,
    StartDate DATETIME2 NOT NULL,
    EndDate DATETIME2 NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CreatedBy NVARCHAR(128) NOT NULL,
    FOREIGN KEY (CreatedBy) REFERENCES Users(Id) ON DELETE NO ACTION
);

CREATE INDEX IX_Ads_DateRange ON Ads(StartDate, EndDate);
CREATE INDEX IX_Ads_IsActive ON Ads(IsActive);
```

### Stored Procedures

#### Get Nearby Stations

```sql
CREATE PROCEDURE GetNearbyStations
    @Latitude DECIMAL(9,6),
    @Longitude DECIMAL(9,6),
    @RadiusKm INT = 5
AS
BEGIN
    -- Approximate conversion of km to degrees (varies by latitude)
    DECLARE @LatDegrees DECIMAL(9,6) = @RadiusKm / 111.0;
    DECLARE @LonDegrees DECIMAL(9,6) = @RadiusKm / (111.0 * COS(RADIANS(@Latitude)));
    
    SELECT 
        s.*,
        u.Name AS OwnerName,
        v.Name AS VerifierName,
        (SELECT AVG(CAST(Rating AS FLOAT)) FROM Reviews WHERE StationId = s.Id) AS AverageRating,
        (SELECT COUNT(*) FROM Reviews WHERE StationId = s.Id) AS ReviewCount,
        -- Haversine formula for accurate distance calculation
        (
            6371 * ACOS(
                COS(RADIANS(@Latitude)) * 
                COS(RADIANS(s.Latitude)) * 
                COS(RADIANS(s.Longitude) - RADIANS(@Longitude)) + 
                SIN(RADIANS(@Latitude)) * 
                SIN(RADIANS(s.Latitude))
            )
        ) AS DistanceKm
    FROM 
        Stations s
    LEFT JOIN 
        Users u ON s.OwnerId = u.Id
    LEFT JOIN 
        Users v ON s.VerifiedBy = v.Id
    WHERE 
        s.Latitude BETWEEN @Latitude - @LatDegrees AND @Latitude + @LatDegrees
        AND s.Longitude BETWEEN @Longitude - @LonDegrees AND @Longitude + @LonDegrees
    ORDER BY 
        DistanceKm ASC;
END
```

#### Update Station Status

```sql
CREATE PROCEDURE UpdateStationStatus
    @StationId UNIQUEIDENTIFIER,
    @UserId NVARCHAR(128),
    @Status NVARCHAR(20)
AS
BEGIN
    BEGIN TRANSACTION;
    
    -- Update the station status
    UPDATE Stations
    SET 
        Status = @Status,
        VerifiedBy = @UserId,
        LastVerified = GETUTCDATE(),
        UpdatedAt = GETUTCDATE()
    WHERE 
        Id = @StationId;
    
    -- Record the verification
    INSERT INTO StationVerifier (StationId, UserId, Status)
    VALUES (@StationId, @UserId, @Status);
    
    -- Update any pending verifier prompts
    UPDATE VerifierPrompts
    SET 
        Status = 'Completed',
        CompletedAt = GETUTCDATE()
    WHERE 
        StationId = @StationId 
        AND UserId = @UserId 
        AND Status = 'Pending'
        AND ExpiresAt > GETUTCDATE();
    
    -- Update user credibility score
    UPDATE Users
    SET CredibilityScore = CredibilityScore + 5
    WHERE Id = @UserId;
    
    COMMIT TRANSACTION;
END
```

## Security Model

### Role-Based Access Control

The application uses four distinct roles:

1. **General User** - Basic access to view stations, submit reviews, and request new stations
2. **Verifier** - All User permissions plus ability to verify station status
3. **Charger Owner** - All User permissions plus ability to manage their own stations
4. **Admin** - Full access to all features and management capabilities

### Permission Matrix

| Resource | Action | User | Verifier | Owner | Admin |
|----------|--------|------|----------|-------|-------|
| Stations | View | ✓ | ✓ | ✓ | ✓ |
| Stations | Create | ✗ | ✗ | ✓ | ✓ |
| Stations | Update Own | ✗ | ✗ | ✓ | ✓ |
| Stations | Update Any | ✗ | ✗ | ✗ | ✓ |
| Stations | Delete | ✗ | ✗ | ✗ | ✓ |
| Reviews | View | ✓ | ✓ | ✓ | ✓ |
| Reviews | Create | ✓ | ✓ | ✓ | ✓ |
| Reviews | Update Own | ✓ | ✓ | ✓ | ✓ |
| Reviews | Delete Own | ✓ | ✓ | ✓ | ✓ |
| Reviews | Delete Any | ✗ | ✗ | ✗ | ✓ |
| PinRequests | Create | ✓ | ✓ | ✓ | ✓ |
| PinRequests | Approve/Reject | ✗ | ✗ | ✗ | ✓ |
| Station Status | Verify | ✗ | ✓ | ✓ | ✓ |
| Users | View | ✗ | ✗ | ✗ | ✓ |
| Users | Manage Roles | ✗ | ✗ | ✗ | ✓ |
| Ads | View | ✓ | ✓ | ✓ | ✓ |
| Ads | Manage | ✗ | ✗ | ✗ | ✓ |

### Azure AD B2C Integration

The application uses Azure AD B2C for authentication with the following configuration:

1. **User Flows**:
   - Sign-up and sign-in
   - Profile editing
   - Password reset

2. **Custom Attributes**:
   - Role (string)
   - CredibilityScore (number)

3. **Application Roles**:
   - User
   - Verifier
   - Owner
   - Admin

### API Security

1. **JWT Authentication**:
   - All API endpoints are secured with JWT tokens
   - Tokens are issued by Azure AD B2C
   - Tokens include user roles and claims

2. **Authorization Policies**:

```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdminRole", policy =>
        policy.RequireRole("Admin"));
        
    options.AddPolicy("RequireOwnerRole", policy =>
        policy.RequireRole("Owner", "Admin"));
        
    options.AddPolicy("RequireVerifierRole", policy =>
        policy.RequireRole("Verifier", "Owner", "Admin"));
        
    options.AddPolicy("StationOwnerOnly", policy =>
        policy.Requirements.Add(new StationOwnerRequirement()));
});

// Register the handler
builder.Services.AddSingleton<IAuthorizationHandler, StationOwnerAuthorizationHandler>();
```

3. **Custom Authorization Handlers**:

```csharp
// StationOwnerRequirement.cs
public class StationOwnerRequirement : IAuthorizationRequirement { }

// StationOwnerAuthorizationHandler.cs
public class StationOwnerAuthorizationHandler : AuthorizationHandler<StationOwnerRequirement, Station>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        StationOwnerRequirement requirement,
        Station station)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        if (userId == null)
        {
            return Task.CompletedTask;
        }
        
        // Check if user is the owner of the station
        if (station.OwnerId == userId)
        {
            context.Succeed(requirement);
        }
        
        // Check if user is an admin (admins can manage all stations)
        if (context.User.IsInRole("Admin"))
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}
```

4. **Controller Authorization**:

```csharp
// StationsController.cs
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requires authentication for all endpoints
public class StationsController : ControllerBase
{
    // Available to all authenticated users
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Station>>> GetStations()
    {
        // Implementation
    }
    
    // Only available to station owners and admins
    [HttpPost]
    [Authorize(Policy = "RequireOwnerRole")]
    public async Task<ActionResult<Station>> CreateStation(StationDto stationDto)
    {
        // Implementation
    }
    
    // Only available to the station owner or admins
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateStation(Guid id, StationDto stationDto)
    {
        var station = await _context.Stations.FindAsync(id);
        
        if (station == null)
        {
            return NotFound();
        }
        
        // Check if user is authorized to update this station
        var authResult = await _authorizationService.AuthorizeAsync(
            User, station, "StationOwnerOnly");
            
        if (!authResult.Succeeded)
        {
            return Forbid();
        }
        
        // Implementation
    }
    
    // Only available to verifiers, owners, and admins
    [HttpPost("{id}/verify")]
    [Authorize(Policy = "RequireVerifierRole")]
    public async Task<IActionResult> VerifyStationStatus(Guid id, StationStatusDto statusDto)
    {
        // Implementation
    }
    
    // Only available to admins
    [HttpDelete("{id}")]
    [Authorize(Policy = "RequireAdminRole")]
    public async Task<IActionResult> DeleteStation(Guid id)
    {
        // Implementation
    }
}
```

### Data Protection

1. **Sensitive Data**:
   - User emails and personal information are stored securely
   - Passwords are managed by Azure AD B2C and never stored in the application database

2. **Data Access Patterns**:
   - Repository pattern is used to abstract data access
   - Service layer enforces business rules and permissions

```csharp
// IStationRepository.cs
public interface IStationRepository
{
    Task<IEnumerable<Station>> GetAllAsync();
    Task<IEnumerable<Station>> GetNearbyAsync(double latitude, double longitude, int radiusKm);
    Task<Station> GetByIdAsync(Guid id);
    Task<Station> CreateAsync(Station station);
    Task<bool> UpdateAsync(Station station);
    Task<bool> DeleteAsync(Guid id);
    Task<bool> UpdateStatusAsync(Guid id, string userId, string status);
}

// StationService.cs
public class StationService : IStationService
{
    private readonly IStationRepository _stationRepository;
    private readonly IAuthorizationService _authorizationService;
    private readonly IUserContextService _userContextService;
    
    public StationService(
        IStationRepository stationRepository,
        IAuthorizationService authorizationService,
        IUserContextService userContextService)
    {
        _stationRepository = stationRepository;
        _authorizationService = authorizationService;
        _userContextService = userContextService;
    }
    
    public async Task<IEnumerable<Station>> GetNearbyStationsAsync(double latitude, double longitude, int radiusKm)
    {
        return await _stationRepository.GetNearbyAsync(latitude, longitude, radiusKm);
    }
    
    public async Task<Station> CreateStationAsync(StationDto stationDto)
    {
        var currentUser = _userContextService.GetCurrentUser();
        
        // Check if user has permission to create stations
        if (!currentUser.IsInRole("Owner") && !currentUser.IsInRole("Admin"))
        {
            throw new UnauthorizedAccessException("Only station owners and admins can create stations");
        }
        
        var station = new Station
        {
            Name = stationDto.Name,
            Address = stationDto.Address,
            Latitude = stationDto.Latitude,
            Longitude = stationDto.Longitude,
            Type = stationDto.Type,
            PowerKW = stationDto.PowerKW,
            Status = "Available", // Default status
            OwnerId = currentUser.Id,
            CreatedAt = DateTime.UtcNow,
            UpdatedAt = DateTime.UtcNow
        };
        
        return await _stationRepository.CreateAsync(station);
    }
    
    // Other methods with appropriate permission checks
}
```

### Client-Side Security

1. **Angular Route Guards**:

```typescript
// src/app/core/guards/auth.guard.ts
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.authService.isAuthenticated().pipe(
      map(isAuthenticated => {
        if (!isAuthenticated) {
          return this.router.createUrlTree(['/auth/login'], {
            queryParams: { returnUrl: state.url }
          });
        }
        
        // Check for required roles
        const requiredRoles = route.data.roles as string[];
        if (requiredRoles && requiredRoles.length > 0) {
          return this.authService.hasRole(requiredRoles).pipe(
            map(hasRole => {
              if (!hasRole) {
                return this.router.createUrlTree(['/unauthorized']);
              }
              return true;
            })
          );
        }
        
        return true;
      })
    );
  }
}
```

2. **Role-Based UI**:

```typescript
// src/app/core/services/role.service.ts
@Injectable({
  providedIn: 'root'
})
export class RoleService {
  private currentUserRoles: string[] = [];
  
  constructor(private authService: AuthService) {
    this.authService.currentUser.subscribe(user => {
      this.currentUserRoles = user?.role ? [user.role] : [];
    });
  }
  
  hasRole(roles: string[]): boolean {
    return this.currentUserRoles.some(role => roles.includes(role));
  }
  
  isAdmin(): boolean {
    return this.currentUserRoles.includes('Admin');
  }
  
  isOwner(): boolean {
    return this.currentUserRoles.includes('Owner') || this.isAdmin();
  }
  
  isVerifier(): boolean {
    return this.currentUserRoles.includes('Verifier') || this.isOwner() || this.isAdmin();
  }
}
```

```html
<!-- src/app/features/stations/station-detail/station-detail.component.html -->
<div class="station-actions">
  <button class="btn-primary" (click)="navigateToStation()">Navigate</button>
  
  <!-- Only show to verifiers, owners, and admins -->
  <button 
    *ngIf="roleService.isVerifier()"
    class="btn-secondary" 
    (click)="verifyStatus()">
    Verify Status
  </button>
  
  <!-- Only show to the station owner or admins -->
  <button 
    *ngIf="isOwner || roleService.isAdmin()"
    class="btn-secondary" 
    (click)="editStation()">
    Edit Station
  </button>
  
  <!-- Only show to admins -->
  <button 
    *ngIf="roleService.isAdmin()"
    class="btn-danger" 
    (click)="deleteStation()">
    Delete Station
  </button>
</div>
```

### Security Best Practices

1. **Input Validation**:
   - All user inputs are validated on both client and server
   - Use parameterized queries to prevent SQL injection

2. **HTTPS**:
   - All communication uses HTTPS
   - HSTS is enabled to prevent downgrade attacks

3. **CORS**:
   - Strict CORS policy is configured to allow only the frontend application

4. **Rate Limiting**:
   - API endpoints are protected with rate limiting to prevent abuse

5. **Logging and Monitoring**:
   - All authentication and authorization events are logged
   - Suspicious activities trigger alerts

6. **Regular Security Reviews**:
   - Conduct regular security reviews and penetration testing
   - Keep all dependencies updated

## Data Migration Strategy

For initial deployment and future updates, the following migration strategy is recommended:

1. **Entity Framework Core Migrations**:
   - Use EF Core's migration system to manage schema changes
   - Create a baseline migration for the initial schema

```csharp
// Initial migration command
dotnet ef migrations add InitialCreate

// Apply migrations to the database
dotnet ef database update
```

2. **Seed Data**:
   - Create seed data for essential records
   - Include default admin user and sample stations

```csharp
// ApplicationDbContextSeed.cs
public static class ApplicationDbContextSeed
{
    public static async Task SeedAsync(ApplicationDbContext context, UserManager<ApplicationUser> userManager)
    {
        // Seed admin user if none exists
        if (!userManager.Users.Any())
        {
            var adminUser = new ApplicationUser
            {
                UserName = "admin@sricharge.com",
                Email = "admin@sricharge.com",
                Name = "System Administrator",
                Role = "Admin",
                CredibilityScore = 100
            };
            
            await userManager.CreateAsync(adminUser, "SecureP@ssword1");
            await userManager.AddToRoleAsync(adminUser, "Admin");
        }
        
        // Seed sample stations if none exist
        if (!context.Stations.Any())
        {
            var stations = new List<Station>
            {
                new Station
                {
                    Name = "Central Mall Charging Station",
                    Address = "123 Main Street, Colombo",
                    Latitude = 6.927079,
                    Longitude = 79.861244,
                    Type = "Type 2",
                    PowerKW = 22,
                    Status = "Available",
                    CreatedAt = DateTime.UtcNow,
                    UpdatedAt = DateTime.UtcNow
                },
                // Add more sample stations
            };
            
            context.Stations.AddRange(stations);
            await context.SaveChangesAsync();
        }
    }
}
```

3. **Database Backup and Restore**:
   - Implement regular automated backups
   - Test restore procedures periodically

## Conclusion

This database schema and security model provide a robust foundation for the SriCharge application. The schema is designed to efficiently store and retrieve all necessary data, while the security model ensures that users can only access and modify data according to their assigned roles.

The integration with Azure AD B2C provides enterprise-grade authentication and authorization, while the custom authorization policies and handlers enforce fine-grained access control at the application level.

By following these patterns and best practices, the application will maintain data integrity, security, and performance as it scales to accommodate more users and charging stations.
