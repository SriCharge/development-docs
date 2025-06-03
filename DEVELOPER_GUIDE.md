# SriCharge Developer Guide

This guide provides detailed instructions for continuing development on the SriCharge EV charging application. It's specifically tailored for developers with .NET and Angular experience who may have limited UI/UX knowledge.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Getting Started](#getting-started)
3. [Architecture Overview](#architecture-overview)
4. [Backend Development (.NET)](#backend-development-net)
5. [Frontend Development (Angular)](#frontend-development-angular)
6. [UI/UX Implementation](#uiux-implementation)
7. [Testing](#testing)
8. [Deployment](#deployment)
9. [Common Tasks](#common-tasks)

## Project Overview

SriCharge is a mobile-first Progressive Web App (PWA) that unifies the EV charging experience in Sri Lanka. The application supports four user roles:

1. **General User** - Search for chargers, view status, submit requests, leave reviews
2. **Verifier** - Confirm charger status when nearby, earn credibility points
3. **Charger Owner** - Add/update charger listings
4. **Admin** - Manage users, approve pin drops, manage ads

The application is built using Angular for the frontend and .NET Core for the backend, with Azure services for authentication, database, storage, and notifications.

## Getting Started

### Prerequisites

- Visual Studio 2022 or later
- Visual Studio Code (recommended for Angular development)
- .NET 6.0 SDK or later
- Node.js 14.x or later
- Angular CLI 14.x or later
- Azure subscription

### Setting Up the Development Environment

1. **Clone the repository**

```bash
git clone https://github.com/your-organization/sricharge.git
cd sricharge
```

2. **Set up the backend**

```bash
cd SriCharge.API
dotnet restore
dotnet build
```

3. **Set up the frontend**

```bash
cd ../SriCharge.Web
npm install
```

4. **Configure local settings**

- Copy `appsettings.example.json` to `appsettings.Development.json` in the API project
- Update the connection strings and Azure service configurations
- Copy `environment.example.ts` to `environment.development.ts` in the Web project
- Update the API URL and Azure AD B2C settings

5. **Run the application**

In one terminal:
```bash
cd SriCharge.API
dotnet run
```

In another terminal:
```bash
cd SriCharge.Web
ng serve
```

The application will be available at `http://localhost:4200/` with the API running at `http://localhost:5000/`.

## Architecture Overview

### Backend Architecture

The backend follows a clean architecture pattern with the following layers:

1. **API Layer** - Controllers and middleware
2. **Service Layer** - Business logic and service implementations
3. **Repository Layer** - Data access and persistence
4. **Domain Layer** - Entity models and business rules

Key components:

- **Controllers** - Handle HTTP requests and responses
- **Services** - Implement business logic and orchestrate operations
- **Repositories** - Provide data access abstraction
- **Entities** - Define the domain model
- **DTOs** - Transfer data between layers

### Frontend Architecture

The Angular frontend is organized by feature modules with a core module for shared services:

1. **Core Module** - Authentication, guards, interceptors, and shared services
2. **Shared Module** - Common components, directives, and pipes
3. **Feature Modules** - Role-specific functionality
4. **Layouts** - Role-based layout components

Key components:

- **Services** - Handle API communication and state management
- **Components** - UI building blocks
- **Guards** - Control route access based on roles
- **Interceptors** - Handle HTTP requests and responses
- **Models** - TypeScript interfaces for data structures

## Backend Development (.NET)

### Adding a New Entity

1. Create a new entity class in `Data/Entities`:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace SriCharge.API.Data.Entities
{
    public class ChargingSession
    {
        public int Id { get; set; }
        
        [Required]
        public int StationId { get; set; }
        public Station Station { get; set; }
        
        [Required]
        public string UserId { get; set; }
        public User User { get; set; }
        
        [Required]
        public DateTime StartTime { get; set; }
        
        public DateTime? EndTime { get; set; }
        
        public decimal EnergyConsumed { get; set; }
        
        public decimal Cost { get; set; }
        
        public string Status { get; set; } = "active";
    }
}
```

2. Add the entity to the `ApplicationDbContext`:

```csharp
public DbSet<ChargingSession> ChargingSessions { get; set; }

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // Existing configurations...
    
    modelBuilder.Entity<ChargingSession>()
        .HasOne(cs => cs.Station)
        .WithMany()
        .HasForeignKey(cs => cs.StationId)
        .OnDelete(DeleteBehavior.Restrict);
        
    modelBuilder.Entity<ChargingSession>()
        .HasOne(cs => cs.User)
        .WithMany()
        .HasForeignKey(cs => cs.UserId)
        .OnDelete(DeleteBehavior.Restrict);
}
```

3. Create a migration:

```bash
dotnet ef migrations add AddChargingSession
dotnet ef database update
```

### Creating a Repository

1. Define the repository interface:

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using SriCharge.API.Data.Entities;

namespace SriCharge.API.Data.Repositories
{
    public interface IChargingSessionRepository
    {
        Task<IEnumerable<ChargingSession>> GetAllAsync();
        Task<ChargingSession> GetByIdAsync(int id);
        Task<IEnumerable<ChargingSession>> GetByUserIdAsync(string userId);
        Task<IEnumerable<ChargingSession>> GetByStationIdAsync(int stationId);
        Task<ChargingSession> CreateAsync(ChargingSession session);
        Task<ChargingSession> UpdateAsync(ChargingSession session);
        Task DeleteAsync(int id);
    }
}
```

2. Implement the repository:

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using SriCharge.API.Data.Entities;

namespace SriCharge.API.Data.Repositories
{
    public class ChargingSessionRepository : IChargingSessionRepository
    {
        private readonly ApplicationDbContext _context;
        
        public ChargingSessionRepository(ApplicationDbContext context)
        {
            _context = context;
        }
        
        public async Task<IEnumerable<ChargingSession>> GetAllAsync()
        {
            return await _context.ChargingSessions
                .Include(cs => cs.Station)
                .Include(cs => cs.User)
                .ToListAsync();
        }
        
        public async Task<ChargingSession> GetByIdAsync(int id)
        {
            return await _context.ChargingSessions
                .Include(cs => cs.Station)
                .Include(cs => cs.User)
                .FirstOrDefaultAsync(cs => cs.Id == id);
        }
        
        public async Task<IEnumerable<ChargingSession>> GetByUserIdAsync(string userId)
        {
            return await _context.ChargingSessions
                .Include(cs => cs.Station)
                .Where(cs => cs.UserId == userId)
                .ToListAsync();
        }
        
        public async Task<IEnumerable<ChargingSession>> GetByStationIdAsync(int stationId)
        {
            return await _context.ChargingSessions
                .Include(cs => cs.User)
                .Where(cs => cs.StationId == stationId)
                .ToListAsync();
        }
        
        public async Task<ChargingSession> CreateAsync(ChargingSession session)
        {
            _context.ChargingSessions.Add(session);
            await _context.SaveChangesAsync();
            return session;
        }
        
        public async Task<ChargingSession> UpdateAsync(ChargingSession session)
        {
            _context.Entry(session).State = EntityState.Modified;
            await _context.SaveChangesAsync();
            return session;
        }
        
        public async Task DeleteAsync(int id)
        {
            var session = await _context.ChargingSessions.FindAsync(id);
            if (session != null)
            {
                _context.ChargingSessions.Remove(session);
                await _context.SaveChangesAsync();
            }
        }
    }
}
```

3. Register the repository in `Startup.cs`:

```csharp
services.AddScoped<IChargingSessionRepository, ChargingSessionRepository>();
```

### Creating a Service

1. Define the service interface:

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using SriCharge.API.Models;

namespace SriCharge.API.Services
{
    public interface IChargingSessionService
    {
        Task<IEnumerable<ChargingSessionDto>> GetAllAsync();
        Task<ChargingSessionDto> GetByIdAsync(int id);
        Task<IEnumerable<ChargingSessionDto>> GetByUserIdAsync(string userId);
        Task<IEnumerable<ChargingSessionDto>> GetByStationIdAsync(int stationId);
        Task<ChargingSessionDto> StartSessionAsync(StartSessionDto startSessionDto);
        Task<ChargingSessionDto> EndSessionAsync(int id, EndSessionDto endSessionDto);
        Task DeleteSessionAsync(int id);
    }
}
```

2. Implement the service:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using AutoMapper;
using SriCharge.API.Data.Entities;
using SriCharge.API.Data.Repositories;
using SriCharge.API.Models;

namespace SriCharge.API.Services
{
    public class ChargingSessionService : IChargingSessionService
    {
        private readonly IChargingSessionRepository _repository;
        private readonly IMapper _mapper;
        
        public ChargingSessionService(IChargingSessionRepository repository, IMapper mapper)
        {
            _repository = repository;
            _mapper = mapper;
        }
        
        public async Task<IEnumerable<ChargingSessionDto>> GetAllAsync()
        {
            var sessions = await _repository.GetAllAsync();
            return _mapper.Map<IEnumerable<ChargingSessionDto>>(sessions);
        }
        
        public async Task<ChargingSessionDto> GetByIdAsync(int id)
        {
            var session = await _repository.GetByIdAsync(id);
            return _mapper.Map<ChargingSessionDto>(session);
        }
        
        public async Task<IEnumerable<ChargingSessionDto>> GetByUserIdAsync(string userId)
        {
            var sessions = await _repository.GetByUserIdAsync(userId);
            return _mapper.Map<IEnumerable<ChargingSessionDto>>(sessions);
        }
        
        public async Task<IEnumerable<ChargingSessionDto>> GetByStationIdAsync(int stationId)
        {
            var sessions = await _repository.GetByStationIdAsync(stationId);
            return _mapper.Map<IEnumerable<ChargingSessionDto>>(sessions);
        }
        
        public async Task<ChargingSessionDto> StartSessionAsync(StartSessionDto startSessionDto)
        {
            var session = new ChargingSession
            {
                StationId = startSessionDto.StationId,
                UserId = startSessionDto.UserId,
                StartTime = DateTime.UtcNow,
                Status = "active"
            };
            
            var createdSession = await _repository.CreateAsync(session);
            return _mapper.Map<ChargingSessionDto>(createdSession);
        }
        
        public async Task<ChargingSessionDto> EndSessionAsync(int id, EndSessionDto endSessionDto)
        {
            var session = await _repository.GetByIdAsync(id);
            if (session == null)
            {
                return null;
            }
            
            session.EndTime = DateTime.UtcNow;
            session.EnergyConsumed = endSessionDto.EnergyConsumed;
            session.Cost = endSessionDto.Cost;
            session.Status = "completed";
            
            var updatedSession = await _repository.UpdateAsync(session);
            return _mapper.Map<ChargingSessionDto>(updatedSession);
        }
        
        public async Task DeleteSessionAsync(int id)
        {
            await _repository.DeleteAsync(id);
        }
    }
}
```

3. Register the service in `Startup.cs`:

```csharp
services.AddScoped<IChargingSessionService, ChargingSessionService>();
```

### Creating a Controller

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using SriCharge.API.Models;
using SriCharge.API.Services;

namespace SriCharge.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    [Authorize]
    public class ChargingSessionsController : ControllerBase
    {
        private readonly IChargingSessionService _service;
        
        public ChargingSessionsController(IChargingSessionService service)
        {
            _service = service;
        }
        
        [HttpGet]
        [Authorize(Roles = "Admin")]
        public async Task<ActionResult<IEnumerable<ChargingSessionDto>>> GetAll()
        {
            var sessions = await _service.GetAllAsync();
            return Ok(sessions);
        }
        
        [HttpGet("{id}")]
        public async Task<ActionResult<ChargingSessionDto>> GetById(int id)
        {
            var session = await _service.GetByIdAsync(id);
            if (session == null)
            {
                return NotFound();
            }
            
            return Ok(session);
        }
        
        [HttpGet("user/{userId}")]
        public async Task<ActionResult<IEnumerable<ChargingSessionDto>>> GetByUserId(string userId)
        {
            // Check if the current user is requesting their own data or is an admin
            if (User.FindFirst("sub")?.Value != userId && !User.IsInRole("Admin"))
            {
                return Forbid();
            }
            
            var sessions = await _service.GetByUserIdAsync(userId);
            return Ok(sessions);
        }
        
        [HttpGet("station/{stationId}")]
        public async Task<ActionResult<IEnumerable<ChargingSessionDto>>> GetByStationId(int stationId)
        {
            var sessions = await _service.GetByStationIdAsync(stationId);
            return Ok(sessions);
        }
        
        [HttpPost]
        public async Task<ActionResult<ChargingSessionDto>> StartSession(StartSessionDto startSessionDto)
        {
            // Set the user ID from the authenticated user
            startSessionDto.UserId = User.FindFirst("sub")?.Value;
            
            var session = await _service.StartSessionAsync(startSessionDto);
            return CreatedAtAction(nameof(GetById), new { id = session.Id }, session);
        }
        
        [HttpPut("{id}/end")]
        public async Task<ActionResult<ChargingSessionDto>> EndSession(int id, EndSessionDto endSessionDto)
        {
            var session = await _service.GetByIdAsync(id);
            if (session == null)
            {
                return NotFound();
            }
            
            // Check if the current user owns this session or is an admin
            if (session.UserId != User.FindFirst("sub")?.Value && !User.IsInRole("Admin"))
            {
                return Forbid();
            }
            
            var updatedSession = await _service.EndSessionAsync(id, endSessionDto);
            return Ok(updatedSession);
        }
        
        [HttpDelete("{id}")]
        [Authorize(Roles = "Admin")]
        public async Task<ActionResult> DeleteSession(int id)
        {
            var session = await _service.GetByIdAsync(id);
            if (session == null)
            {
                return NotFound();
            }
            
            await _service.DeleteSessionAsync(id);
            return NoContent();
        }
    }
}
```

### Implementing Role-Based Authorization

1. Configure Azure AD B2C in `Startup.cs`:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAdB2C", options);
        options.TokenValidationParameters.NameClaimType = "name";
        options.TokenValidationParameters.RoleClaimType = "extension_UserRole";
    },
    options => { Configuration.Bind("AzureAdB2C", options); });

services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdminRole", policy => policy.RequireRole("Admin"));
    options.AddPolicy("RequireOwnerRole", policy => policy.RequireRole("Owner", "Admin"));
    options.AddPolicy("RequireVerifierRole", policy => policy.RequireRole("Verifier", "Admin"));
});
```

2. Use role-based policies in controllers:

```csharp
[Authorize(Policy = "RequireAdminRole")]
[HttpGet("admin/users")]
public async Task<ActionResult<IEnumerable<UserDto>>> GetAllUsers()
{
    var users = await _userService.GetAllAsync();
    return Ok(users);
}

[Authorize(Policy = "RequireOwnerRole")]
[HttpPost("stations")]
public async Task<ActionResult<StationDto>> CreateStation(CreateStationDto createStationDto)
{
    // Implementation
}

[Authorize(Policy = "RequireVerifierRole")]
[HttpPost("stations/{id}/verify")]
public async Task<ActionResult<StationDto>> VerifyStation(int id, VerifyStationDto verifyStationDto)
{
    // Implementation
}
```

## Frontend Development (Angular)

### Adding a New Component

1. Generate a new component:

```bash
ng generate component features/charging-sessions/session-list
```

2. Implement the component:

```typescript
// session-list.component.ts
import { Component, OnInit } from '@angular/core';
import { ChargingSessionService } from '../../../core/services/charging-session.service';
import { ChargingSession } from '../../../core/models/charging-session.model';
import { RoleContext } from '../../../core/contexts/role.context';

@Component({
  selector: 'app-session-list',
  templateUrl: './session-list.component.html',
  styleUrls: ['./session-list.component.scss']
})
export class SessionListComponent implements OnInit {
  sessions: ChargingSession[] = [];
  isLoading = true;
  
  constructor(
    private sessionService: ChargingSessionService,
    public roleContext: RoleContext
  ) { }

  ngOnInit(): void {
    this.loadSessions();
  }
  
  loadSessions(): void {
    this.isLoading = true;
    
    if (this.roleContext.isAdmin()) {
      this.sessionService.getAllSessions().subscribe(
        sessions => {
          this.sessions = sessions;
          this.isLoading = false;
        },
        error => {
          console.error('Error loading sessions', error);
          this.isLoading = false;
        }
      );
    } else {
      this.sessionService.getUserSessions().subscribe(
        sessions => {
          this.sessions = sessions;
          this.isLoading = false;
        },
        error => {
          console.error('Error loading sessions', error);
          this.isLoading = false;
        }
      );
    }
  }
  
  endSession(sessionId: number): void {
    // Implementation for ending a session
  }
}
```

3. Create the HTML template:

```html
<!-- session-list.component.html -->
<div class="session-list">
  <!-- Loading State -->
  <div class="loading-overlay" *ngIf="isLoading">
    <div class="spinner"></div>
    <p>Loading sessions...</p>
  </div>
  
  <!-- Page Header -->
  <div class="page-header">
    <h1>Charging Sessions</h1>
    <p class="page-description">View and manage your charging sessions</p>
  </div>
  
  <!-- Sessions Table -->
  <div class="sessions-table-container" *ngIf="!isLoading">
    <table class="sessions-table" *ngIf="sessions.length > 0">
      <thead>
        <tr>
          <th>Station</th>
          <th>Start Time</th>
          <th>End Time</th>
          <th>Energy (kWh)</th>
          <th>Cost</th>
          <th>Status</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let session of sessions">
          <td>{{session.stationName}}</td>
          <td>{{session.startTime | date:'medium'}}</td>
          <td>{{session.endTime ? (session.endTime | date:'medium') : 'In progress'}}</td>
          <td>{{session.energyConsumed | number:'1.2-2'}}</td>
          <td>{{session.cost | currency:'LKR':'symbol':'1.2-2'}}</td>
          <td>
            <span class="status-badge" [ngClass]="session.status">
              {{session.status | titlecase}}
            </span>
          </td>
          <td>
            <button 
              class="btn-primary" 
              *ngIf="session.status === 'active'"
              (click)="endSession(session.id)">
              End Session
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    
    <div class="empty-state" *ngIf="sessions.length === 0">
      <div class="empty-icon">
        <i class="icon-battery"></i>
      </div>
      <h3>No Charging Sessions</h3>
      <p>You haven't started any charging sessions yet.</p>
    </div>
  </div>
</div>
```

4. Add the styles:

```scss
// session-list.component.scss
.session-list {
  padding: 1rem;
  max-width: 1200px;
  margin: 0 auto;
}

/* Loading Overlay */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.9);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  
  .spinner {
    width: 40px;
    height: 40px;
    border: 4px solid #e5e7eb;
    border-top: 4px solid #10b981;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 1rem;
  }
  
  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
}

/* Page Header */
.page-header {
  margin-bottom: 2rem;
  
  h1 {
    font-size: 1.875rem;
    font-weight: 600;
    margin: 0 0 0.5rem 0;
    color: #1f2937;
  }
  
  .page-description {
    color: #6b7280;
    margin: 0;
  }
}

/* Sessions Table */
.sessions-table-container {
  overflow-x: auto;
  background-color: white;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.sessions-table {
  width: 100%;
  border-collapse: collapse;
  
  th, td {
    padding: 1rem;
    text-align: left;
    border-bottom: 1px solid #e5e7eb;
  }
  
  th {
    font-weight: 500;
    color: #6b7280;
    background-color: #f9fafb;
  }
  
  tr {
    &:last-child td {
      border-bottom: none;
    }
    
    &:hover {
      background-color: #f9fafb;
    }
  }
}

.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
  
  &.active {
    background-color: #fef3c7;
    color: #92400e;
  }
  
  &.completed {
    background-color: #d1fae5;
    color: #065f46;
  }
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 3rem 1.5rem;
  
  .empty-icon {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 4rem;
    height: 4rem;
    border-radius: 50%;
    background-color: #f3f4f6;
    margin-bottom: 1.5rem;
    
    i {
      color: #9ca3af;
      font-size: 2rem;
    }
  }
  
  h3 {
    font-size: 1.25rem;
    font-weight: 600;
    margin: 0 0 0.75rem 0;
    color: #1f2937;
  }
  
  p {
    color: #6b7280;
    margin: 0;
  }
}

/* Button */
.btn-primary {
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background-color: #10b981;
  border: none;
  border-radius: 0.375rem;
  color: white;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  
  &:hover {
    background-color: #059669;
  }
}
```

5. Create a service for API communication:

```typescript
// charging-session.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../../environments/environment';
import { ChargingSession } from '../../models/charging-session.model';
import { AuthService } from '../auth.service';

@Injectable({
  providedIn: 'root'
})
export class ChargingSessionService {
  private apiUrl = `${environment.apiUrl}/api/chargingSessions`;
  
  constructor(
    private http: HttpClient,
    private authService: AuthService
  ) { }
  
  getAllSessions(): Observable<ChargingSession[]> {
    return this.http.get<ChargingSession[]>(this.apiUrl);
  }
  
  getUserSessions(): Observable<ChargingSession[]> {
    const userId = this.authService.getUserId();
    return this.http.get<ChargingSession[]>(`${this.apiUrl}/user/${userId}`);
  }
  
  getStationSessions(stationId: number): Observable<ChargingSession[]> {
    return this.http.get<ChargingSession[]>(`${this.apiUrl}/station/${stationId}`);
  }
  
  getSessionById(id: number): Observable<ChargingSession> {
    return this.http.get<ChargingSession>(`${this.apiUrl}/${id}`);
  }
  
  startSession(stationId: number): Observable<ChargingSession> {
    return this.http.post<ChargingSession>(this.apiUrl, { stationId });
  }
  
  endSession(id: number, energyConsumed: number, cost: number): Observable<ChargingSession> {
    return this.http.put<ChargingSession>(`${this.apiUrl}/${id}/end`, {
      energyConsumed,
      cost
    });
  }
  
  deleteSession(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

6. Define the model:

```typescript
// charging-session.model.ts
export interface ChargingSession {
  id: number;
  stationId: number;
  stationName: string;
  userId: string;
  userName: string;
  startTime: Date;
  endTime?: Date;
  energyConsumed: number;
  cost: number;
  status: 'active' | 'completed';
}

export interface StartSessionDto {
  stationId: number;
}

export interface EndSessionDto {
  energyConsumed: number;
  cost: number;
}
```

7. Add the component to the routing module:

```typescript
const routes: Routes = [
  // Existing routes...
  {
    path: 'sessions',
    component: SessionListComponent,
    canActivate: [AuthGuard]
  }
];
```

### Implementing Role-Based UI

1. Create a role context service:

```typescript
// role.context.ts
import { Injectable } from '@angular/core';
import { AuthService } from '../services/auth.service';

export type UserRole = 'User' | 'Verifier' | 'Owner' | 'Admin';

@Injectable({
  providedIn: 'root'
})
export class RoleContext {
  constructor(private authService: AuthService) { }
  
  getUserRole(): UserRole {
    return this.authService.getUserRole();
  }
  
  isAdmin(): boolean {
    return this.getUserRole() === 'Admin';
  }
  
  isOwner(): boolean {
    return this.getUserRole() === 'Owner' || this.isAdmin();
  }
  
  isVerifier(): boolean {
    return this.getUserRole() === 'Verifier' || this.isAdmin();
  }
  
  canManageStations(): boolean {
    return this.isOwner();
  }
  
  canVerifyStations(): boolean {
    return this.isVerifier();
  }
  
  canManageUsers(): boolean {
    return this.isAdmin();
  }
}
```

2. Create a role guard:

```typescript
// role.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { RoleContext, UserRole } from '../contexts/role.context';

@Injectable({
  providedIn: 'root'
})
export class RoleGuard implements CanActivate {
  constructor(
    private roleContext: RoleContext,
    private router: Router
  ) { }
  
  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    const requiredRoles = route.data.roles as UserRole[];
    
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }
    
    const userRole = this.roleContext.getUserRole();
    
    if (requiredRoles.includes(userRole)) {
      return true;
    }
    
    this.router.navigate(['/access-denied']);
    return false;
  }
}
```

3. Use the role guard in routing:

```typescript
const routes: Routes = [
  {
    path: 'admin',
    component: AdminLayoutComponent,
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['Admin'] },
    children: [
      {
        path: 'users',
        component: UserManagementComponent
      },
      // Other admin routes...
    ]
  },
  {
    path: 'owner',
    component: OwnerLayoutComponent,
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['Owner', 'Admin'] },
    children: [
      {
        path: 'stations',
        component: StationManagementComponent
      },
      // Other owner routes...
    ]
  },
  // Other routes...
];
```

4. Conditionally show UI elements based on role:

```html
<nav class="side-nav">
  <!-- Common Navigation Items (All Users) -->
  <a routerLink="/map" routerLinkActive="active" class="nav-item">
    <i class="icon-map"></i>
    <span>Map</span>
  </a>
  
  <!-- Verifier-specific Navigation Items -->
  <ng-container *ngIf="roleContext.canVerifyStations()">
    <div class="nav-divider"></div>
    <div class="nav-group-title">Verifier</div>
    
    <a routerLink="/verifier/dashboard" routerLinkActive="active" class="nav-item">
      <i class="icon-verify"></i>
      <span>Verification Dashboard</span>
    </a>
  </ng-container>
  
  <!-- Owner-specific Navigation Items -->
  <ng-container *ngIf="roleContext.canManageStations()">
    <div class="nav-divider"></div>
    <div class="nav-group-title">Owner</div>
    
    <a routerLink="/owner/stations" routerLinkActive="active" class="nav-item">
      <i class="icon-manage"></i>
      <span>Manage Stations</span>
    </a>
  </ng-container>
  
  <!-- Admin-specific Navigation Items -->
  <ng-container *ngIf="roleContext.isAdmin()">
    <div class="nav-divider"></div>
    <div class="nav-group-title">Admin</div>
    
    <a routerLink="/admin/users" routerLinkActive="active" class="nav-item">
      <i class="icon-users"></i>
      <span>User Management</span>
    </a>
  </ng-container>
</nav>
```

## UI/UX Implementation

### Following the Design System

The SriCharge application uses a consistent design system defined in the UI/UX Guidelines. Here's how to implement it:

1. **Use the color variables**:

```scss
:root {
  --color-primary: #10b981;
  --color-primary-dark: #059669;
  --color-primary-light: #d1fae5;
  
  --color-status-available: #10b981;
  --color-status-in-use: #f59e0b;
  --color-status-broken: #ef4444;
  
  --color-text-dark: #1f2937;
  --color-text-medium: #6b7280;
  --color-text-light: #9ca3af;
  --color-background: #f9fafb;
  --color-border: #e5e7eb;
}

.btn-primary {
  background-color: var(--color-primary);
  color: white;
  
  &:hover {
    background-color: var(--color-primary-dark);
  }
}

.status-badge.available {
  background-color: var(--color-primary-light);
  color: var(--color-primary-dark);
}
```

2. **Follow the typography guidelines**:

```scss
body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  font-size: 1rem;
  line-height: 1.5;
  color: var(--color-text-dark);
}

h1 {
  font-size: 1.875rem;
  font-weight: 600;
  line-height: 1.2;
  margin-bottom: 1rem;
}

.text-secondary {
  color: var(--color-text-medium);
}
```

3. **Use consistent component patterns**:

```html
<!-- Button pattern -->
<button class="btn btn-primary">
  <i class="icon-plus"></i>
  Add Station
</button>

<!-- Card pattern -->
<div class="card">
  <div class="card-header">
    <h3>Station Details</h3>
  </div>
  <div class="card-body">
    <!-- Content -->
  </div>
  <div class="card-footer">
    <!-- Actions -->
  </div>
</div>

<!-- Form pattern -->
<div class="form-group">
  <label for="stationName">Station Name</label>
  <input type="text" id="stationName" class="form-control">
  <div class="error-message" *ngIf="stationForm.get('name').invalid && stationForm.get('name').touched">
    Please enter a station name
  </div>
</div>
```

### Implementing Mobile-First Design

1. **Use responsive containers**:

```scss
.container {
  width: 100%;
  padding: 1rem;
  
  @media (min-width: 768px) {
    padding: 1.5rem;
  }
  
  @media (min-width: 1024px) {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

2. **Create responsive grids**:

```scss
.grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
  
  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
  }
  
  @media (min-width: 1024px) {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

3. **Adapt layouts for different screen sizes**:

```scss
.page-layout {
  display: flex;
  flex-direction: column;
  
  @media (min-width: 1024px) {
    flex-direction: row;
  }
  
  .sidebar {
    width: 100%;
    
    @media (min-width: 1024px) {
      width: 250px;
      min-width: 250px;
    }
  }
  
  .main-content {
    flex: 1;
  }
}
```

4. **Use touch-friendly sizing**:

```scss
.interactive-element {
  min-height: 44px;
  min-width: 44px;
  padding: 0.75rem;
}
```

### Ensuring Accessibility

1. **Use semantic HTML**:

```html
<header>
  <h1>Station Management</h1>
</header>

<main>
  <section aria-labelledby="stationsHeading">
    <h2 id="stationsHeading">Your Stations</h2>
    <!-- Content -->
  </section>
</main>

<footer>
  <!-- Footer content -->
</footer>
```

2. **Provide proper ARIA attributes**:

```html
<button 
  aria-label="Add new station"
  aria-pressed="false"
  (click)="addStation()">
  <i class="icon-plus" aria-hidden="true"></i>
  Add Station
</button>

<div 
  role="dialog"
  aria-labelledby="modalTitle"
  aria-modal="true">
  <h2 id="modalTitle">Edit Station</h2>
  <!-- Modal content -->
</div>
```

3. **Ensure sufficient color contrast**:

```scss
// Good contrast
.text-primary {
  color: #059669; // Dark green on white background (contrast ratio > 4.5:1)
}

// Bad contrast - don't do this
.text-light {
  color: #a1a1aa; // Light gray on white background (contrast ratio < 4.5:1)
}
```

4. **Make forms accessible**:

```html
<div class="form-group">
  <label for="stationName" id="stationNameLabel">Station Name</label>
  <input 
    type="text" 
    id="stationName"
    class="form-control" 
    aria-labelledby="stationNameLabel"
    aria-required="true"
    [attr.aria-invalid]="stationForm.get('name').invalid && stationForm.get('name').touched"
    placeholder="e.g., Central Mall Charging Station">
  <div 
    class="error-message" 
    *ngIf="stationForm.get('name').invalid && stationForm.get('name').touched"
    id="stationNameError"
    aria-live="assertive">
    Please enter a station name
  </div>
</div>
```

## Testing

### Unit Testing Components

```typescript
// session-list.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { of } from 'rxjs';
import { SessionListComponent } from './session-list.component';
import { ChargingSessionService } from '../../../core/services/charging-session.service';
import { RoleContext } from '../../../core/contexts/role.context';

describe('SessionListComponent', () => {
  let component: SessionListComponent;
  let fixture: ComponentFixture<SessionListComponent>;
  let mockSessionService: jasmine.SpyObj<ChargingSessionService>;
  let mockRoleContext: jasmine.SpyObj<RoleContext>;
  
  const mockSessions = [
    {
      id: 1,
      stationId: 1,
      stationName: 'Test Station',
      userId: 'user1',
      userName: 'Test User',
      startTime: new Date(),
      energyConsumed: 10,
      cost: 500,
      status: 'active' as const
    }
  ];
  
  beforeEach(async () => {
    mockSessionService = jasmine.createSpyObj('ChargingSessionService', [
      'getAllSessions',
      'getUserSessions'
    ]);
    mockRoleContext = jasmine.createSpyObj('RoleContext', ['isAdmin']);
    
    mockSessionService.getAllSessions.and.returnValue(of(mockSessions));
    mockSessionService.getUserSessions.and.returnValue(of(mockSessions));
    
    await TestBed.configureTestingModule({
      declarations: [ SessionListComponent ],
      providers: [
        { provide: ChargingSessionService, useValue: mockSessionService },
        { provide: RoleContext, useValue: mockRoleContext }
      ]
    })
    .compileComponents();
  });
  
  beforeEach(() => {
    fixture = TestBed.createComponent(SessionListComponent);
    component = fixture.componentInstance;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should load all sessions for admin users', () => {
    mockRoleContext.isAdmin.and.returnValue(true);
    
    fixture.detectChanges();
    
    expect(mockSessionService.getAllSessions).toHaveBeenCalled();
    expect(mockSessionService.getUserSessions).not.toHaveBeenCalled();
    expect(component.sessions).toEqual(mockSessions);
    expect(component.isLoading).toBeFalse();
  });
  
  it('should load user sessions for non-admin users', () => {
    mockRoleContext.isAdmin.and.returnValue(false);
    
    fixture.detectChanges();
    
    expect(mockSessionService.getAllSessions).not.toHaveBeenCalled();
    expect(mockSessionService.getUserSessions).toHaveBeenCalled();
    expect(component.sessions).toEqual(mockSessions);
    expect(component.isLoading).toBeFalse();
  });
});
```

### Integration Testing

```typescript
// app.component.spec.ts
import { TestBed, ComponentFixture } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { AppComponent } from './app.component';
import { NavbarComponent } from './core/components/navbar/navbar.component';
import { AuthService } from './core/services/auth.service';
import { of } from 'rxjs';

describe('AppComponent', () => {
  let fixture: ComponentFixture<AppComponent>;
  let app: AppComponent;
  let mockAuthService: jasmine.SpyObj<AuthService>;
  
  beforeEach(async () => {
    mockAuthService = jasmine.createSpyObj('AuthService', ['isAuthenticated', 'getUserName']);
    mockAuthService.isAuthenticated.and.returnValue(of(true));
    mockAuthService.getUserName.and.returnValue('Test User');
    
    await TestBed.configureTestingModule({
      imports: [
        RouterTestingModule,
        HttpClientTestingModule
      ],
      declarations: [
        AppComponent,
        NavbarComponent
      ],
      providers: [
        { provide: AuthService, useValue: mockAuthService }
      ]
    }).compileComponents();
  });
  
  beforeEach(() => {
    fixture = TestBed.createComponent(AppComponent);
    app = fixture.componentInstance;
    fixture.detectChanges();
  });
  
  it('should create the app', () => {
    expect(app).toBeTruthy();
  });
  
  it('should render navbar when authenticated', () => {
    const navbar = fixture.nativeElement.querySelector('app-navbar');
    expect(navbar).toBeTruthy();
  });
});
```

## Deployment

### Deploying the Backend to Azure App Service

1. Publish the API project from Visual Studio:
   - Right-click on the API project
   - Select "Publish"
   - Choose "Azure App Service"
   - Follow the wizard to create or select an existing App Service

2. Configure Azure App Service settings:
   - Set up connection strings
   - Configure authentication with Azure AD B2C
   - Set up CORS to allow requests from the frontend

3. Deploy using Azure DevOps Pipeline:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetSdkVersion: '6.x'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET SDK $(dotnetSdkVersion)'
  inputs:
    version: '$(dotnetSdkVersion)'

- task: DotNetCoreCLI@2
  displayName: 'Restore project dependencies'
  inputs:
    command: 'restore'
    projects: 'SriCharge.API/SriCharge.API.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build the project'
  inputs:
    command: 'build'
    projects: 'SriCharge.API/SriCharge.API.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  displayName: 'Publish the project'
  inputs:
    command: 'publish'
    projects: 'SriCharge.API/SriCharge.API.csproj'
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    publishWebProjects: false
    zipAfterPublish: true

- task: AzureWebApp@1
  displayName: 'Deploy to Azure Web App'
  inputs:
    azureSubscription: 'Your-Azure-Subscription'
    appType: 'webApp'
    appName: 'sricharge-api'
    package: '$(Build.ArtifactStagingDirectory)/*.zip'
    deploymentMethod: 'auto'
```

### Deploying the Frontend to Azure Static Web Apps

1. Build the Angular application:

```bash
ng build --prod
```

2. Deploy using Azure Static Web Apps:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '14.x'
  displayName: 'Install Node.js'

- script: |
    npm install
    npm run build -- --prod
  displayName: 'npm install and build'
  workingDirectory: 'SriCharge.Web'

- task: AzureStaticWebApp@0
  inputs:
    app_location: 'SriCharge.Web/dist/sricharge-web'
    api_location: ''
    output_location: ''
    azure_static_web_apps_api_token: '$(deployment_token)'
```

3. Configure Azure Static Web Apps:
   - Set up routes for SPA navigation
   - Configure custom domain
   - Set up HTTPS

## Common Tasks

### Adding a New Feature

1. **Plan the feature**:
   - Define the user stories
   - Design the UI/UX
   - Plan the API endpoints
   - Identify the data models

2. **Implement the backend**:
   - Create the entity model
   - Create the repository
   - Create the service
   - Create the controller
   - Write unit tests

3. **Implement the frontend**:
   - Create the Angular components
   - Create the service for API communication
   - Implement the UI following the design system
   - Add routing
   - Write unit tests

4. **Test the feature**:
   - Test on different devices and screen sizes
   - Verify accessibility
   - Test with different user roles
   - Test error handling

5. **Deploy the feature**:
   - Deploy the backend changes
   - Deploy the frontend changes
   - Monitor for any issues

### Implementing Role-Based Features

1. **Define the role requirements**:
   - Identify which roles can access the feature
   - Define the permissions for each role

2. **Update the backend**:
   - Add role-based authorization to controllers
   - Implement permission checks in services

3. **Update the frontend**:
   - Use the RoleContext to conditionally show UI elements
   - Add role guards to routes
   - Implement role-specific UI variations

### Optimizing Performance

1. **Backend optimizations**:
   - Use async/await for all I/O operations
   - Implement caching for frequently accessed data
   - Optimize database queries
   - Use pagination for large data sets

2. **Frontend optimizations**:
   - Lazy load feature modules
   - Use OnPush change detection
   - Optimize bundle size
   - Implement virtual scrolling for long lists
   - Use memoization for expensive computations

3. **Network optimizations**:
   - Minimize API calls
   - Implement request caching
   - Use compression
   - Optimize asset loading

### Troubleshooting Common Issues

1. **Authentication issues**:
   - Check Azure AD B2C configuration
   - Verify token validation parameters
   - Check CORS configuration
   - Inspect network requests for token issues

2. **API connectivity issues**:
   - Check API URL configuration
   - Verify CORS settings
   - Check network requests in browser dev tools
   - Verify API is running and accessible

3. **UI rendering issues**:
   - Check browser console for errors
   - Verify CSS is loading correctly
   - Test on different browsers and devices
   - Check for responsive design issues

4. **Performance issues**:
   - Use browser dev tools to identify bottlenecks
   - Check for excessive API calls
   - Look for memory leaks
   - Optimize rendering performance

## Conclusion

This developer guide provides a comprehensive overview of the SriCharge application architecture, development practices, and common tasks. By following these guidelines, you can maintain consistency, ensure high quality, and efficiently implement new features.

For more detailed information, refer to the following resources:

- UI/UX Guidelines directory for design system documentation
- API documentation for backend endpoints
- Component examples in the codebase
- Azure documentation for cloud services
