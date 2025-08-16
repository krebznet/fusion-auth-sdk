# FusionAuth SDK Helper Resources

> **Making FusionAuth integration less painful, one REST call at a time**

FusionAuth integration can be notoriously difficult to get working correctly through their official client SDKs. Many small configuration details can cause hard-to-debug problems that will have you questioning your life choices. This repository provides battle-tested helper resources to make your life easier when integrating with FusionAuth from Node.js, Java, and Kubernetes deployments.

## 🚨 Why This Exists

The official FusionAuth SDKs are... let's say "quirky." Common issues include:
- Inconsistent error handling across language SDKs
- Poorly documented configuration requirements
- Breaking changes between versions
- Complex setup for seemingly simple use cases
- API endpoints that expect different HTTP methods than documented

This repo cuts through the noise with practical, working examples.

## 📦 What's Included

### 1. TypeScript FusionConnector Package
A clean TypeScript wrapper that uses regular REST endpoints instead of the official SDK.

**Features:**
- ✅ User registration and authentication
- ✅ Token validation and refresh
- ✅ Comprehensive error handling
- ✅ Full test suite included
- ✅ Environment variable configuration
- ✅ Verbose logging for debugging

### 2. Java Spring Boot Resources
Enterprise-ready Java integration that's actually easier than the Node.js version.

**Features:**
- ✅ Spring Boot starter for quick setup
- ✅ Auth service designed for microservice clusters
- ✅ Drop-in authentication components
- ✅ Production-ready configuration

### 3. Kubernetes Helm Chart
Complete Kubernetes deployment with all the fixings.

**Features:**
- ✅ PostgreSQL database setup
- ✅ Init container for database seeding
- ✅ Initial admin user creation
- ✅ Let's Encrypt SSL with cert-manager integration
- ✅ Production-ready configuration

> **Note:** The Helm chart assumes you're using Let's Encrypt and cert-manager. It's not for everyone, but provides a solid reference implementation.

## 🚀 Quick Start

### Prerequisites

Before you begin, you'll need to set up in your FusionAuth instance:
- ✅ **Application** - Your app configuration
- ✅ **Tenant** - Multi-tenant setup (or use default)
- ✅ **API Key** - With appropriate permissions (NOT the master key!)

### TypeScript Integration

```bash
# Install dependencies
npm install

# Copy environment template
cp .env.example .env

# Configure your FusionAuth credentials
# Edit .env with your FusionAuth URL, API key, application ID, and tenant ID

# Run the test suite
npm run test:fusion
```

**Environment Variables:**
```env
FUSION_AUTH_URL=https://your-fusionauth-instance.com
FUSION_AUTH_API_KEY=your-application-api-key
FUSION_AUTH_CLIENT_ID=your-application-id
FUSION_AUTH_TENANT_ID=your-tenant-id
```

**Basic Usage:**
```typescript
import { FusionConnector, FusionConnectorConfig } from './fusion-connector';

const config: FusionConnectorConfig = {
  apiKey: process.env.FUSION_AUTH_API_KEY!,
  fusionUrl: process.env.FUSION_AUTH_URL!,
  applicationId: process.env.FUSION_AUTH_CLIENT_ID!,
  tenantId: process.env.FUSION_AUTH_TENANT_ID!,
};

const connector = new FusionConnector(config);

// Register a user
const user = await connector.registerUser({
  email: 'user@example.com',
  firstName: 'John',
  lastName: 'Doe',
  password: 'SecurePassword123!',
  useTenantSpecificUsername: true
});

// Authenticate
const authResult = await connector.authUser('user@example.com', 'SecurePassword123!');

// Validate token
const isValid = await connector.validateToken(authResult.token);
```

### Java Spring Boot Integration

```java
// Add the auth starter to your Spring Boot service
@SpringBootApplication
@EnableFusionAuth
public class YourApplication {
    public static void main(String[] args) {
        SpringApplication.run(YourApplication.class, args);
    }
}

// Use the auth service in your controllers
@RestController
public class AuthController {
    @Autowired
    private FusionAuthService authService;
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        return authService.authenticate(request.getEmail(), request.getPassword());
    }
}
```

### Kubernetes Deployment

```bash
# Add the Helm repository (if using external chart)
helm repo add fusionauth-helper ./helm

# Install with your configuration
helm install fusionauth fusionauth-helper/fusionauth \
  --set fusionauth.host=auth.yourdomain.com \
  --set postgres.password=your-secure-password \
  --set fusionauth.adminPassword=your-admin-password

# Verify deployment
kubectl get pods -l app=fusionauth
```

## 🔧 Configuration Details

### API Key Setup (IMPORTANT!)

**DO NOT use the master API key for your application!**

1. Go to **Settings → API Keys** in FusionAuth Admin UI
2. Click **Add** (+ button)
3. Configure:
   - **Name**: Descriptive name (e.g., "MyApp Production")
   - **Tenant**: Select your specific tenant (not "All Tenants")
   - **Endpoints**: Only select what you need:
     - `/api/login` - for authentication
     - `/api/user` - for user management
     - `/api/user/registration` - for registrations
     - `/api/jwt/validate` - for token validation
     - `/api/jwt/refresh` - for token refresh

### Common Gotchas

1. **Token Validation 405 Error**
   - Use `GET /api/jwt/validate` with token in Authorization header
   - NOT `POST` with token in request body

2. **CORS Issues**
   - Configure CORS settings in FusionAuth Admin UI
   - Add your domain to allowed origins

3. **Tenant ID Required**
   - Always include `X-FusionAuth-TenantId` header for multi-tenant setups
   - Even if using the default tenant

4. **Password Complexity**
   - Check your tenant's password validation rules
   - Default rules might be stricter than expected

## 🧪 Testing

### TypeScript Tests
```bash
# Run all tests
npm test

# Run FusionAuth integration tests specifically
npm run test:fusion

# Run tests with verbose logging
DEBUG=true npm run test:fusion
```

### Java Tests
```bash
# Run Spring Boot tests
./mvnw test

# Run integration tests
./mvnw test -Dtest=FusionAuthIntegrationTest
```

## 🏗 Project Structure

```
├── typescript/
│   ├── src/
│   │   ├── fusion-connector.ts      # Main connector class
│   │   ├── fusion-connector.test.ts # Comprehensive tests
│   │   └── types.ts                 # TypeScript interfaces
│   ├── package.json
│   └── .env.example
├── java/
