# Azure Backend Setup for SriCharge

This document provides detailed instructions for setting up the Azure backend services for the SriCharge application. Azure is recommended for this project as it integrates seamlessly with the .NET backend and provides all the necessary services for authentication, database, storage, and hosting.

## Azure Services Overview

For the SriCharge application, we'll use the following Azure services:

1. **Azure App Service** - For hosting the .NET Core API
2. **Azure SQL Database** - For storing application data
3. **Azure Active Directory B2C** - For user authentication and role management
4. **Azure Blob Storage** - For storing images and other assets
5. **Azure Notification Hubs** - For push notifications to verifiers

## Prerequisites

- Azure subscription
- Azure CLI installed locally
- Visual Studio 2022 or Visual Studio Code with Azure extensions
- .NET 7.0 SDK or later

## Step 1: Create Azure Resources

### Azure SQL Database

1. Log in to the Azure Portal (https://portal.azure.com)
2. Create a new SQL Database:
   - Click "Create a resource" > "Databases" > "SQL Database"
   - Fill in the basic details:
     - Subscription: Your subscription
     - Resource group: Create new (e.g., "sricharge-rg")
     - Database name: "sricharge-db"
     - Server: Create new
       - Server name: "sricharge-server"
       - Server admin login: Create a secure username
       - Password: Create a secure password
     - Compute + storage: Configure as needed (Basic tier is sufficient for development)
   - Click "Review + create" and then "Create"

3. Once created, go to the database and note the connection string from the "Connection strings" section

### Azure App Service

1. In the Azure Portal, create a new App Service:
   - Click "Create a resource" > "Web App"
   - Fill in the basic details:
     - Subscription: Your subscription
     - Resource group: Use the same as above
     - Name: "sricharge-api"
     - Publish: Code
     - Runtime stack: .NET 7 (or your version)
     - Operating System: Windows
     - Region: Choose a region close to your users
     - App Service Plan: Create new
       - Name: "sricharge-plan"
       - Pricing tier: Choose as needed (B1 is sufficient for development)
   - Click "Review + create" and then "Create"

### Azure Active Directory B2C

1. In the Azure Portal, create a new Azure AD B2C tenant:
   - Click "Create a resource" > "Identity" > "Azure Active Directory B2C"
   - Select "Create a new Azure AD B2C Tenant"
   - Fill in the organization name, domain name, and other details
   - Click "Create"

2. Once created, register your application:
   - Go to your new B2C tenant
   - Navigate to "App registrations"
   - Click "New registration"
   - Name: "SriCharge"
   - Supported account types: "Accounts in any identity provider or organizational directory"
   - Redirect URI: Web, https://sricharge-api.azurewebsites.net/.auth/login/aad/callback
   - Click "Register"

3. Configure user flows:
   - In your B2C tenant, go to "User flows"
   - Create flows for:
     - Sign up and sign in
     - Profile editing
     - Password reset

### Azure Blob Storage

1. In the Azure Portal, create a new Storage account:
   - Click "Create a resource" > "Storage" > "Storage account"
   - Fill in the basic details:
     - Subscription: Your subscription
     - Resource group: Use the same as above
     - Storage account name: "srichargeassets"
     - Region: Choose the same region as your App Service
     - Performance: Standard
     - Redundancy: Locally-redundant storage (LRS)
   - Click "Review + create" and then "Create"

2. Once created, create containers for:
   - "profile-images" - For user profile photos
   - "station-images" - For charging station photos
   - "ads" - For advertisement banners

### Azure Notification Hubs

1. In the Azure Portal, create a new Notification Hub:
   - Click "Create a resource" > "Mobile" > "Notification Hubs"
   - Fill in the basic details:
     - Subscription: Your subscription
     - Resource group: Use the same as above
     - Namespace: "sricharge-notifications"
     - Location: Choose the same region as your App Service
     - Pricing tier: Free
   - Click "Create"

2. Once created, add a new Notification Hub:
   - Name: "sricharge-hub"
   - Click "Create"

## Step 2: Configure Connection Strings and App Settings

1. Go to your App Service in the Azure Portal
2. Navigate to "Configuration" > "Application settings"
3. Add the following settings:
   - `ConnectionStrings:DefaultConnection`: Your SQL Database connection string
   - `AzureAd:Instance`: https://login.microsoftonline.com/
   - `AzureAd:Domain`: Your B2C domain
   - `AzureAd:TenantId`: Your B2C tenant ID
   - `AzureAd:ClientId`: Your B2C application ID
   - `AzureAd:CallbackPath`: /signin-oidc
   - `BlobStorage:ConnectionString`: Your Blob Storage connection string
   - `NotificationHub:ConnectionString`: Your Notification Hub connection string
   - `NotificationHub:HubName`: sricharge-hub

## Step 3: Update .NET Project for Azure Integration

### Configure Entity Framework for Azure SQL

1. Open your .NET project
2. Update the `ApplicationDbContext.cs` file to use SQL Server:

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users { get; set; }
    public DbSet<Station> Stations { get; set; }
    public DbSet<Review> Reviews { get; set; }
    public DbSet<PinRequest> PinRequests { get; set; }
    public DbSet<Ad> Ads { get; set; }
    public DbSet<StationVerifier> StationVerifiers { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure relationships and constraints
        modelBuilder.Entity<Review>()
            .HasOne(r => r.User)
            .WithMany(u => u.Reviews)
            .HasForeignKey(r => r.UserId);

        modelBuilder.Entity<Review>()
            .HasOne(r => r.Station)
            .WithMany(s => s.Reviews)
            .HasForeignKey(r => r.StationId);

        modelBuilder.Entity<Station>()
            .HasOne(s => s.Owner)
            .WithMany(u => u.OwnedStations)
            .HasForeignKey(s => s.OwnerId)
            .OnDelete(DeleteBehavior.Restrict);

        modelBuilder.Entity<PinRequest>()
            .HasOne(p => p.User)
            .WithMany(u => u.PinRequests)
            .HasForeignKey(p => p.UserId);

        modelBuilder.Entity<StationVerifier>()
            .HasOne(sv => sv.User)
            .WithMany(u => u.VerifiedStations)
            .HasForeignKey(sv => sv.UserId);

        modelBuilder.Entity<StationVerifier>()
            .HasOne(sv => sv.Station)
            .WithMany(s => s.Verifiers)
            .HasForeignKey(sv => sv.StationId);
    }
}
```

2. Update the `Startup.cs` or `Program.cs` file to configure services:

```csharp
// Program.cs for .NET 7+
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Configure Azure SQL Database
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Configure Azure AD B2C Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

// Configure Azure Blob Storage
builder.Services.AddSingleton<IBlobStorageService>(provider =>
{
    var connectionString = builder.Configuration["BlobStorage:ConnectionString"];
    return new BlobStorageService(connectionString);
});

// Configure Azure Notification Hubs
builder.Services.AddSingleton<INotificationService>(provider =>
{
    var connectionString = builder.Configuration["NotificationHub:ConnectionString"];
    var hubName = builder.Configuration["NotificationHub:HubName"];
    return new NotificationService(connectionString, hubName);
});

// Add CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAngularApp", policy =>
    {
        policy.WithOrigins("https://sricharge.azurewebsites.net")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseCors("AllowAngularApp");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Implement Azure Blob Storage Service

Create a service to handle file uploads to Azure Blob Storage:

```csharp
public interface IBlobStorageService
{
    Task<string> UploadFileAsync(string containerName, string fileName, Stream fileStream);
    Task<bool> DeleteFileAsync(string containerName, string fileName);
    string GetFileUrl(string containerName, string fileName);
}

public class BlobStorageService : IBlobStorageService
{
    private readonly BlobServiceClient _blobServiceClient;

    public BlobStorageService(string connectionString)
    {
        _blobServiceClient = new BlobServiceClient(connectionString);
    }

    public async Task<string> UploadFileAsync(string containerName, string fileName, Stream fileStream)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        await containerClient.CreateIfNotExistsAsync();

        var blobClient = containerClient.GetBlobClient(fileName);
        await blobClient.UploadAsync(fileStream, true);

        return blobClient.Uri.ToString();
    }

    public async Task<bool> DeleteFileAsync(string containerName, string fileName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobClient = containerClient.GetBlobClient(fileName);
        
        return await blobClient.DeleteIfExistsAsync();
    }

    public string GetFileUrl(string containerName, string fileName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobClient = containerClient.GetBlobClient(fileName);
        
        return blobClient.Uri.ToString();
    }
}
```

### Implement Azure Notification Service

Create a service to send push notifications using Azure Notification Hubs:

```csharp
public interface INotificationService
{
    Task<bool> SendNotificationAsync(string userId, string title, string message, Dictionary<string, string> data = null);
    Task<bool> SendNotificationToAllAsync(string title, string message, Dictionary<string, string> data = null);
}

public class NotificationService : INotificationService
{
    private readonly NotificationHubClient _hub;

    public NotificationService(string connectionString, string hubName)
    {
        _hub = NotificationHubClient.CreateClientFromConnectionString(connectionString, hubName);
    }

    public async Task<bool> SendNotificationAsync(string userId, string title, string message, Dictionary<string, string> data = null)
    {
        try
        {
            var notification = new Dictionary<string, string>
            {
                { "title", title },
                { "body", message }
            };

            if (data != null)
            {
                foreach (var item in data)
                {
                    notification.Add(item.Key, item.Value);
                }
            }

            // Send to specific user (using tags)
            await _hub.SendFcmNativeNotificationAsync(JsonConvert.SerializeObject(notification), $"userId:{userId}");
            return true;
        }
        catch
        {
            return false;
        }
    }

    public async Task<bool> SendNotificationToAllAsync(string title, string message, Dictionary<string, string> data = null)
    {
        try
        {
            var notification = new Dictionary<string, string>
            {
                { "title", title },
                { "body", message }
            };

            if (data != null)
            {
                foreach (var item in data)
                {
                    notification.Add(item.Key, item.Value);
                }
            }

            // Send to all devices
            await _hub.SendFcmNativeNotificationAsync(JsonConvert.SerializeObject(notification));
            return true;
        }
        catch
        {
            return false;
        }
    }
}
```

## Step 4: Update Angular Project for Azure Integration

### Configure Authentication with Azure AD B2C

1. Install the required packages:

```bash
npm install @azure/msal-browser @azure/msal-angular
```

2. Create an authentication service:

```typescript
// src/app/core/auth/azure-ad.service.ts
import { Injectable } from '@angular/core';
import { MsalService, MsalBroadcastService, MSAL_GUARD_CONFIG, MsalGuardConfiguration } from '@azure/msal-angular';
import { AuthenticationResult, InteractionType, PopupRequest, RedirectRequest } from '@azure/msal-browser';
import { Observable, of, from } from 'rxjs';
import { map, switchMap, catchError } from 'rxjs/operators';
import { environment } from '../../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class AzureAdService {
  constructor(
    private msalService: MsalService,
    private msalBroadcastService: MsalBroadcastService
  ) {}

  login(): Observable<AuthenticationResult> {
    return from(this.msalService.loginPopup({
      scopes: environment.azureAd.scopes
    }));
  }

  logout(): void {
    this.msalService.logout();
  }

  getToken(): Observable<string> {
    return from(this.msalService.acquireTokenSilent({
      scopes: environment.azureAd.scopes
    })).pipe(
      map((result: AuthenticationResult) => result.accessToken),
      catchError(() => {
        return from(this.msalService.acquireTokenPopup({
          scopes: environment.azureAd.scopes
        })).pipe(
          map((result: AuthenticationResult) => result.accessToken)
        );
      })
    );
  }

  isAuthenticated(): boolean {
    return this.msalService.instance.getAllAccounts().length > 0;
  }

  getUserInfo(): Observable<any> {
    const accounts = this.msalService.instance.getAllAccounts();
    if (accounts.length === 0) {
      return of(null);
    }

    const account = accounts[0];
    return of({
      id: account.homeAccountId,
      name: account.name,
      email: account.username,
      // You'll need to get the role from the API
      role: 'User'
    });
  }
}
```

3. Configure MSAL in your app module:

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { MsalModule, MsalService, MsalGuard, MsalInterceptor } from '@azure/msal-angular';
import { InteractionType, PublicClientApplication } from '@azure/msal-browser';
import { HTTP_INTERCEPTORS, HttpClientModule } from '@angular/common/http';
import { environment } from '../environments/environment';

const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

@NgModule({
  declarations: [
    AppComponent,
    // other components
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    MsalModule.forRoot(new PublicClientApplication({
      auth: {
        clientId: environment.azureAd.clientId,
        authority: environment.azureAd.authority,
        redirectUri: environment.azureAd.redirectUri,
      },
      cache: {
        cacheLocation: 'localStorage',
        storeAuthStateInCookie: isIE,
      }
    }), {
      interactionType: InteractionType.Redirect,
      authRequest: {
        scopes: environment.azureAd.scopes
      }
    }, {
      interactionType: InteractionType.Redirect,
      protectedResourceMap: new Map([
        [environment.apiUrl, environment.azureAd.scopes]
      ])
    }),
    // other modules
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    },
    MsalGuard,
    // other providers
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

4. Update your environment files:

```typescript
// src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'https://sricharge-api.azurewebsites.net/api',
  azureAd: {
    clientId: 'your-client-id',
    authority: 'https://login.microsoftonline.com/your-tenant-id',
    redirectUri: 'http://localhost:4200',
    scopes: ['api://sricharge-api/user_impersonation']
  }
};
```

### Configure Azure Blob Storage for File Uploads

Create a service to handle file uploads to Azure Blob Storage:

```typescript
// src/app/core/services/blob-storage.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class BlobStorageService {
  private apiUrl = `${environment.apiUrl}/storage`;

  constructor(private http: HttpClient) {}

  uploadFile(containerName: string, file: File): Observable<string> {
    const formData = new FormData();
    formData.append('file', file);

    return this.http.post<string>(`${this.apiUrl}/upload/${containerName}`, formData);
  }

  deleteFile(containerName: string, fileName: string): Observable<boolean> {
    return this.http.delete<boolean>(`${this.apiUrl}/delete/${containerName}/${fileName}`);
  }

  getFileUrl(containerName: string, fileName: string): string {
    return `${environment.storageUrl}/${containerName}/${fileName}`;
  }
}
```

## Step 5: Deploy to Azure

### Deploy .NET API to Azure App Service

1. In Visual Studio:
   - Right-click on your API project
   - Select "Publish"
   - Choose "Azure" as the target
   - Select "Azure App Service" (Windows or Linux)
   - Select your existing App Service
   - Click "Publish"

2. Or using the Azure CLI:
   ```bash
   dotnet publish -c Release
   cd bin/Release/net7.0/publish
   zip -r publish.zip .
   az webapp deployment source config-zip --resource-group sricharge-rg --name sricharge-api --src publish.zip
   ```

### Deploy Angular App to Azure Static Web Apps

1. Create a Static Web App in Azure:
   - In the Azure Portal, click "Create a resource"
   - Search for "Static Web App" and select it
   - Fill in the basic details:
     - Subscription: Your subscription
     - Resource group: Use the same as above
     - Name: "sricharge-web"
     - Hosting plan: Free
     - Region: Choose a region close to your users
   - Click "Review + create" and then "Create"

2. Deploy using GitHub Actions:
   - Connect your GitHub repository
   - Configure the build settings:
     - App location: "/"
     - Api location: "api"
     - Output location: "dist/sricharge-web"
   - GitHub will automatically create a workflow file

3. Or deploy manually:
   ```bash
   ng build --prod
   az storage blob upload-batch -d '$web' -s ./dist/sricharge-web --account-name srichargeassets
   ```

## Step 6: Configure CORS and Security

1. In your App Service, go to "CORS" and add your Angular app's URL
2. Ensure all sensitive API endpoints are protected with authentication
3. Configure Azure AD B2C policies for password complexity, MFA, etc.

## Step 7: Set Up CI/CD Pipelines

1. For the .NET API:
   - Create an Azure DevOps pipeline or GitHub Actions workflow
   - Configure it to build and deploy to your App Service

2. For the Angular app:
   - Create an Azure DevOps pipeline or GitHub Actions workflow
   - Configure it to build and deploy to your Static Web App

## Conclusion

This setup provides a robust, scalable, and secure backend for the SriCharge application using Azure services. The integration between .NET and Azure is seamless, making it an ideal choice for developers with .NET experience.

For local development, you can use:
- Azure SQL Database Emulator or LocalDB
- Azurite for local Azure Storage emulation
- Azure AD B2C can be tested against the actual tenant

Remember to keep your connection strings and sensitive information in user secrets during development and in Azure Key Vault for production.
