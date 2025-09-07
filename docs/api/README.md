# BetterSpend API Documentation

This directory contains comprehensive API documentation for the BetterSpend system, built with tRPC for end-to-end type safety.

## API Overview

The BetterSpend API is built using tRPC, providing:
- End-to-end type safety
- Automatic API documentation
- Input validation with Zod
- Error handling and logging
- Authentication and authorization

## API Structure

### Authentication Endpoints
- User registration and login
- Session management
- Role-based access control
- Password reset functionality

### Core Business Endpoints

#### Vendor Management
- Create, read, update, delete vendors
- Vendor search and filtering
- Contact information management

#### Product Management
- Product catalog operations
- Vendor-product associations
- Price and description management

#### Budget Management
- Budget creation and tracking
- Spending calculations
- Budget allocation and monitoring

#### Requisition Management
- Draft creation and management
- Item addition and modification
- Submission and approval workflows
- Status tracking and updates

#### Approval System
- Approval workflow management
- Status updates and notifications
- Audit trail maintenance
- Rejection handling

## Authentication & Authorization

### Authentication Methods
- NextAuth.js integration
- Multiple provider support (Google, Discord, etc.)
- Session-based authentication
- Secure cookie management

### Authorization Levels
- **REQUESTOR**: Can create and manage own requisitions
- **APPROVER**: Can approve/reject requisitions
- **ADMIN**: Can manage vendors, products, and budgets
- **SUPER_ADMIN**: Full system access including user management

### Security Features
- CSRF protection
- Input validation and sanitization
- Rate limiting
- Audit logging

## Error Handling

### Error Types
- **Validation Errors**: Invalid input data
- **Authentication Errors**: Invalid credentials or expired sessions
- **Authorization Errors**: Insufficient permissions
- **Business Logic Errors**: Workflow violations
- **System Errors**: Database or server issues

### Error Response Format
```typescript
{
  error: {
    code: string;
    message: string;
    details?: any;
  }
}
```

## Rate Limiting

API endpoints are rate-limited to prevent abuse:
- Authentication endpoints: 5 requests per minute
- CRUD operations: 100 requests per minute
- Search endpoints: 50 requests per minute

## API Versioning

The API uses semantic versioning:
- Major version changes for breaking changes
- Minor version changes for new features
- Patch version changes for bug fixes

## Testing

### Test Environment
- Separate test database
- Mock authentication
- Automated test suite
- Integration testing

### Test Coverage
- Unit tests for business logic
- Integration tests for API endpoints
- E2E tests for complete workflows
- Performance testing

## Documentation Standards

### Endpoint Documentation
Each endpoint should include:
- Purpose and functionality
- Input parameters and validation
- Output format and examples
- Error conditions
- Authentication requirements
- Rate limiting information

### Code Examples
- TypeScript client examples
- cURL command examples
- Error handling examples
- Authentication examples

## Contributing to API Documentation

1. Update endpoint documentation when adding new features
2. Include code examples for new endpoints
3. Document any breaking changes
4. Update error handling documentation
5. Test all examples before publishing
