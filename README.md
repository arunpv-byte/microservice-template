# Microservice Template - .NET 8

Enterprise-grade microservice template following Clean Architecture and Vertical Slice patterns with dynamic third-party service authentication.

## Architecture

- **Clean Architecture**: API → Application → Domain
- **Vertical Slice**: Features organized by use case
- **CQRS**: Commands and Queries with MediatR
- **Domain-Driven Design**: Rich domain models with behavior
- **Dynamic Authentication**: Configuration-driven third-party service auth
- **Factory Pattern**: Centralized external service resolution

## Technology Stack

- .NET 8 (LTS)
- Dapper (Data Access)
- MediatR (CQRS)
- FluentValidation (Validation)
- Serilog (Logging)
- xUnit, Moq, FluentAssertions (Testing)
- SQL Server (Database)
- Docker (Containerization)

## Project Structure

```
microservice-template/
├── src/
│   ├── microservice-template.API          # Presentation Layer
│   ├── microservice-template.Application  # Application Layer
│   ├── microservice-template.Domain       # Domain Layer
│   └── microservice-template.Infrastructure # Infrastructure Layer
└── tests/
    ├── microservice-template.UnitTests
    └── microservice-template.IntegrationTests
```

## Getting Started

### Prerequisites

- .NET 8 SDK
- SQL Server (or Docker)
- Docker Desktop (optional)

### Running Locally

1. **Update Connection String**
   
   Edit `appsettings.json` in the API project with your SQL Server connection string.

2. **Run the Application**

   ```bash
   dotnet restore
   dotnet build
   dotnet run --project src/microservice-template.API
   ```

3. **Access Swagger**
   
   Navigate to: `https://localhost:5001/swagger`

### Running with Docker

```bash
docker-compose up -d
```

Access the API at: `http://localhost:5000`

## API Endpoints

### Cardholders

- `POST /api/cardholders` - Create a new cardholder
- `GET /api/cardholders/{userId}` - Get cardholder by NymCard user ID
- `PUT /api/cardholders/{userId}` - Update cardholder information

### Cards

- `GET /api/cards/{accountId}/balance` - Get card available balance

### Health Check

- `GET /health` - Application health status

### Authentication

- **Swagger UI**: JWT Bearer token authentication enabled
- **Third-Party APIs**: Dynamic authentication (ApiKey, Bearer, Basic)

## Testing

### Run Unit Tests

```bash
dotnet test tests/microservice-template.UnitTests
```

### Run All Tests

```bash
dotnet test
```

## Configuration

### Connection Strings

Configure in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MicroserviceDb;..."
  }
}
```

### Third-Party Services

Configure dynamic authentication in `appsettings.json`:

```json
{
  "ThirdPartyServices": {
    "NymCard": {
      "BaseUrl": "https://api.sand.platform.nymcard.com",
      "AuthType": "ApiKey",
      "ApiKey": "<NymCardApiKey>"
    },
    "Stripe": {
      "BaseUrl": "https://api.stripe.com",
      "AuthType": "Bearer",
      "Token": "<StripeApiKey>"
    },
    "PayPal": {
      "BaseUrl": "https://api.paypal.com",
      "AuthType": "Basic",
      "Username": "<PayPalClientId>",
      "Password": "<PayPalClientSecret>"
    }
  }
}
```

**Supported Auth Types:**
- `ApiKey` - Adds `X-API-Key` header
- `Bearer` - Adds `Authorization: Bearer {token}` header
- `Basic` - Adds `Authorization: Basic {credentials}` header

### Logging

Serilog is configured for structured logging. Adjust levels in `appsettings.json`.

## Key Features

### Dynamic Third-Party Authentication

- Configuration-driven authentication for external services
- Support for multiple auth types (ApiKey, Bearer, Basic)
- Factory pattern for service resolution
- Environment-specific configurations
- No code changes needed for new services

### Exception Handling

- Global exception middleware
- Domain-specific exceptions
- Correlation ID tracking
- Structured error responses

### Validation

- FluentValidation for request validation
- Fail-fast validation pipeline
- Meaningful error messages

### Logging

- Structured logging with Serilog
- Request/response logging
- Performance tracking
- Correlation ID support

### Health Checks

- Database connectivity check
- Ready for Kubernetes probes

### API Documentation

- Swagger UI with JWT Bearer authentication
- Interactive API testing
- Comprehensive endpoint documentation

## Customization Guide

### Adding a New Feature

1. Create feature folder in `Application/Features/`
2. Add Command/Query, Handler, and Response
3. Use `IThirdPartyServiceFactory` for external services
4. Add validator in `Application/Validators/`
5. Add controller endpoint in `API/Controllers/`
6. Add unit tests in `UnitTests/Application/`

### Adding a New Third-Party Service

1. Define service interface in `Application/Interfaces/`
2. Implement service client extending `ResilientHttpClient`
3. Add service configuration in `appsettings.json`:
   ```json
   "ThirdPartyServices": {
     "ServiceName": {
       "BaseUrl": "https://api.service.com",
       "AuthType": "Bearer",
       "Token": "<ServiceToken>"
     }
   }
   ```
4. Register service in `DependencyInjection.cs`:
   ```csharp
   services.AddHttpClient<IServiceName, ServiceClient>(client =>
   {
       ConfigureServiceAuth(client, configuration, "ServiceName");
   });
   ```
5. Use via factory in handlers:
   ```csharp
   var service = _serviceFactory.GetService<IServiceName>();
   ```

### Adding a New Entity

1. Create entity in `Domain/Entities/`
2. Add repository interface in `Application/Interfaces/`
3. Implement repository in `Infrastructure/Repositories/`
4. Add domain tests in `UnitTests/Domain/`

## Production Considerations

- Configure proper connection strings
- Set up application secrets management for third-party API keys
- Configure logging sinks (e.g., Application Insights)
- Set up health checks monitoring
- Configure CORS policies
- Add authentication/authorization
- Set up CI/CD pipelines
- Configure rate limiting
- Add API versioning
- Implement circuit breakers for external services
- Monitor third-party service dependencies
- Set up alerting for authentication failures

## License

This template is provided as-is for enterprise use.
