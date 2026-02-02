# Microservice Template - .NET 8

Enterprise-grade microservice template following Clean Architecture and Vertical Slice patterns with dynamic third-party service authentication and comprehensive enterprise features.

## Architecture

- **Clean Architecture**: API → Application → Domain
- **Vertical Slice**: Features organized by use case
- **CQRS**: Commands and Queries with MediatR
- **Domain-Driven Design**: Rich domain models with behavior
- **Dynamic Authentication**: Configuration-driven third-party service auth
- **Factory Pattern**: Centralized external service resolution
- **Event-Driven Architecture**: Domain events with message publishing
- **Structured Logging**: JSON-formatted logs with correlation IDs

## Technology Stack

- .NET 8 (LTS)
- Dapper (Data Access)
- MediatR (CQRS)
- FluentValidation (Validation)
- Serilog (Structured Logging)
- xUnit, Moq, FluentAssertions (Testing)
- SQL Server (Database)
- Docker (Containerization)
- **Enterprise Features**:
  - API Versioning (Asp.Versioning)
  - Rate Limiting (.NET 8 Built-in)
  - Circuit Breaker (Polly)
  - Distributed Tracing (OpenTelemetry)
  - Caching (Redis/Memory)
  - Message Queue (Event-driven)
  - Structured Logging (JSON format)

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

### Cardholders (v1.0)

- `POST /api/v1/cardholders` - Create a new cardholder
- `GET /api/v1/cardholders/{userId}` - Get cardholder by NymCard user ID
- `PUT /api/v1/cardholders/{userId}` - Update cardholder information

### Health Check

- `GET /health` - Application health status

### Authentication

- **Swagger UI**: JWT Bearer token authentication enabled
- **Third-Party APIs**: Dynamic authentication (ApiKey, Bearer, Basic, OAuth2)
- **Rate Limiting**: 100 requests per minute per endpoint

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
  "ServiceName": "microservice-template",
  "ServiceVersion": "1.0.0",
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=******;...",
    "Redis": "localhost:6379"
  },
  "Caching": {
    "Type": "Memory"
  },
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
    },
    "OAuth2Service": {
      "BaseUrl": "https://api.oauth2service.com",
      "AuthType": "oauth_client_credentials",
      "TokenUrl": "https://auth.oauth2service.com/token",
      "ClientId": "<OAuth2ClientId>",
      "ClientSecret": "<OAuth2ClientSecret>",
      "Scope": "api:read api:write"
    },
    "LegacyService": {
      "BaseUrl": "https://api.legacyservice.com",
      "AuthType": "oauth_password",
      "TokenUrl": "https://auth.legacyservice.com/token",
      "ClientId": "<LegacyClientId>",
      "ClientSecret": "<LegacyClientSecret>",
      "Username": "<ServiceUsername>",
      "Password": "<ServicePassword>",
      "Scope": "full_access"
    }
  }
}
```

**Supported Auth Types:**
- `ApiKey` - Adds `X-API-Key` header
- `Bearer` - Adds `Authorization: Bearer {token}` header
- `Basic` - Adds `Authorization: Basic {credentials}` header
- `oauth_client_credentials` - OAuth 2.0 Client Credentials flow
- `oauth_password` - OAuth 2.0 Resource Owner Password flow

### Structured Logging

Serilog is configured for JSON-formatted structured logging with correlation IDs. All log entries automatically include:
- **CorrelationId**: Unique identifier for request tracking
- **Application**: Service name
- **Environment**: Current environment
- **Version**: Application version
- **MachineName**: Host machine identifier

### AWS Secrets Management

For production environments, sensitive data should be stored in AWS Secrets Manager or Parameter Store:

**AWS Secrets Manager (JSON format):**
```json
{
  "ThirdPartyServices:NymCard:ApiKey": "<NymCardApiKey>",
  "ThirdPartyServices:Stripe:Token": "<StripeApiKey>",
  "ThirdPartyServices:OAuth2Service:ClientSecret": "<OAuth2ClientSecret>",
  "ConnectionStrings:DefaultConnection": "Server=prod-db;Database=MicroserviceDb;..."
}
```

**AWS Parameter Store:**
```
/microservice-template/database/connection-string
/microservice-template/nymcard/api-key
/microservice-template/stripe/secret-key
/microservice-template/oauth2/client-secret
```

**Environment Variables:**
```bash
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=<access-key>
AWS_SECRET_ACCESS_KEY=<secret-key>
# Or use IAM roles in ECS/EKS
```

## Key Features

### API Versioning

- URL-based versioning (`/api/v1/endpoint`)
- Query string versioning (`?version=1.0`)
- Header-based versioning (`X-Version: 1.0`)
- Automatic API explorer integration

### Rate Limiting

- Fixed window rate limiting (100 requests/minute)
- Token bucket limiting for strict endpoints
- Configurable per endpoint
- Automatic 429 responses

### Circuit Breaker & Resilience

- Polly-based circuit breaker pattern
- Exponential backoff retry policy
- Transient fault handling
- Automatic recovery

### Distributed Tracing

- OpenTelemetry integration with correlation IDs
- ASP.NET Core, HTTP, and SQL instrumentation
- New Relic integration via OTLP exporter
- Jaeger exporter support for development
- Request/response tracking across services
- Custom spans and business metrics

### Caching

- Redis distributed caching
- In-memory caching fallback
- Configurable cache providers
- JSON serialization support

### Event-Driven Architecture

- Domain events with MediatR
- Message publishing abstraction
- In-memory publisher (development)
- Ready for external message queues

### Dynamic Third-Party Authentication

- Configuration-driven authentication for external services
- Support for multiple auth types (ApiKey, Bearer, Basic, OAuth2)
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

### Structured Logging

- JSON-formatted logs with Serilog
- Automatic correlation ID enrichment
- Request/response logging middleware
- Performance tracking
- Sensitive data redaction
- Distributed tracing integration

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

- **AWS Secrets Management**: Use AWS Secrets Manager for API keys and connection strings
- **IAM Roles**: Use IAM roles instead of access keys in ECS/EKS environments
- **Parameter Store**: Use AWS Systems Manager Parameter Store for configuration values
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
- Configure structured logging aggregation
- Set up correlation ID tracking across services

## License

This template is provided as-is for enterprise use.
