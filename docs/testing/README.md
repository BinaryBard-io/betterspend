# BetterSpend Testing Framework

## Overview
This document outlines the comprehensive testing strategy for the BetterSpend procurement system, covering unit tests, integration tests, end-to-end tests, and quality assurance processes.

## Testing Philosophy

### Test Pyramid
- **Unit Tests (70%)**: Fast, isolated tests for individual functions and components
- **Integration Tests (20%)**: Tests for API endpoints and database interactions
- **E2E Tests (10%)**: Full user workflow tests

### Testing Principles
- Test behavior, not implementation
- Write tests before fixing bugs
- Maintain high test coverage
- Keep tests fast and reliable
- Use descriptive test names

## Testing Stack

### Core Testing Tools
- **Jest**: JavaScript testing framework
- **React Testing Library**: Component testing
- **Playwright**: End-to-end testing
- **Prisma Test Environment**: Database testing
- **MSW**: API mocking

### Additional Tools
- **ESLint**: Code quality and testing rules
- **Prettier**: Code formatting
- **Husky**: Git hooks for testing
- **Coverage**: Test coverage reporting

## Test Structure

### Directory Organization
```
tests/
├── unit/                 # Unit tests
│   ├── components/      # React component tests
│   ├── utils/          # Utility function tests
│   └── hooks/          # Custom hook tests
├── integration/         # Integration tests
│   ├── api/            # API endpoint tests
│   ├── database/       # Database operation tests
│   └── auth/           # Authentication tests
├── e2e/                # End-to-end tests
│   ├── workflows/      # User workflow tests
│   ├── pages/          # Page-specific tests
│   └── fixtures/       # Test data and fixtures
├── mocks/              # Mock data and functions
└── setup/              # Test configuration
```

## Unit Testing

### Component Testing
```typescript
// Example component test
import { render, screen, fireEvent } from '@testing-library/react';
import { RequisitionForm } from '../components/RequisitionForm';

describe('RequisitionForm', () => {
  it('should render form fields correctly', () => {
    render(<RequisitionForm />);
    
    expect(screen.getByLabelText(/title/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/description/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });
  
  it('should validate required fields', async () => {
    render(<RequisitionForm />);
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(await screen.findByText(/title is required/i)).toBeInTheDocument();
  });
});
```

### Utility Function Testing
```typescript
// Example utility test
import { calculateTotal, formatCurrency } from '../utils/calculations';

describe('calculations', () => {
  describe('calculateTotal', () => {
    it('should calculate total correctly', () => {
      const items = [
        { quantity: 2, unitPrice: 10 },
        { quantity: 3, unitPrice: 15 }
      ];
      
      expect(calculateTotal(items)).toBe(65);
    });
    
    it('should handle empty array', () => {
      expect(calculateTotal([])).toBe(0);
    });
  });
  
  describe('formatCurrency', () => {
    it('should format currency correctly', () => {
      expect(formatCurrency(1234.56)).toBe('$1,234.56');
    });
  });
});
```

### Custom Hook Testing
```typescript
// Example hook test
import { renderHook, act } from '@testing-library/react';
import { useRequisitions } from '../hooks/useRequisitions';

describe('useRequisitions', () => {
  it('should fetch requisitions on mount', async () => {
    const { result } = renderHook(() => useRequisitions());
    
    expect(result.current.loading).toBe(true);
    
    await act(async () => {
      await new Promise(resolve => setTimeout(resolve, 100));
    });
    
    expect(result.current.loading).toBe(false);
    expect(result.current.requisitions).toHaveLength(0);
  });
});
```

## Integration Testing

### API Endpoint Testing
```typescript
// Example API test
import { createMocks } from 'node-mocks-http';
import handler from '../pages/api/requisitions';

describe('/api/requisitions', () => {
  it('should create requisition with valid data', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: {
        title: 'Test Requisition',
        description: 'Test Description',
        items: [
          { productId: '1', quantity: 2, unitPrice: 10 }
        ]
      }
    });
    
    await handler(req, res);
    
    expect(res._getStatusCode()).toBe(201);
    expect(JSON.parse(res._getData())).toMatchObject({
      title: 'Test Requisition',
      status: 'DRAFT'
    });
  });
  
  it('should return 401 for unauthenticated requests', async () => {
    const { req, res } = createMocks({
      method: 'GET'
    });
    
    await handler(req, res);
    
    expect(res._getStatusCode()).toBe(401);
  });
});
```

### Database Testing
```typescript
// Example database test
import { PrismaClient } from '@prisma/client';
import { createTestUser, createTestVendor } from '../test-utils';

describe('Database Operations', () => {
  let prisma: PrismaClient;
  
  beforeAll(async () => {
    prisma = new PrismaClient({
      datasources: {
        db: {
          url: process.env.TEST_DATABASE_URL
        }
      }
    });
  });
  
  afterAll(async () => {
    await prisma.$disconnect();
  });
  
  beforeEach(async () => {
    await prisma.requisition.deleteMany();
    await prisma.user.deleteMany();
  });
  
  it('should create requisition with items', async () => {
    const user = await createTestUser(prisma);
    const vendor = await createTestVendor(prisma);
    
    const requisition = await prisma.requisition.create({
      data: {
        title: 'Test Requisition',
        requestorId: user.id,
        items: {
          create: [
            {
              customItem: 'Test Item',
              quantity: 2,
              unitPrice: 10,
              totalPrice: 20
            }
          ]
        }
      },
      include: { items: true }
    });
    
    expect(requisition.title).toBe('Test Requisition');
    expect(requisition.items).toHaveLength(1);
    expect(requisition.totalAmount).toBe(20);
  });
});
```

### tRPC Testing
```typescript
// Example tRPC test
import { createTRPCMsw } from 'msw-trpc';
import { appRouter } from '../server/api/root';

const trpcMsw = createTRPCMsw(appRouter);

describe('tRPC Procedures', () => {
  it('should create vendor', async () => {
    const caller = appRouter.createCaller({
      session: { user: { id: '1', role: 'ADMIN' } }
    });
    
    const vendor = await caller.vendor.create({
      name: 'Test Vendor',
      contactInfo: 'test@vendor.com'
    });
    
    expect(vendor.name).toBe('Test Vendor');
    expect(vendor.id).toBeDefined();
  });
});
```

## End-to-End Testing

### User Workflow Testing
```typescript
// Example E2E test
import { test, expect } from '@playwright/test';

test.describe('Requisition Workflow', () => {
  test('should create and submit requisition', async ({ page }) => {
    // Login as requestor
    await page.goto('/login');
    await page.fill('[data-testid=email]', 'requestor@test.com');
    await page.fill('[data-testid=password]', 'password');
    await page.click('[data-testid=login-button]');
    
    // Navigate to create requisition
    await page.click('[data-testid=create-requisition]');
    
    // Fill requisition form
    await page.fill('[data-testid=title]', 'Test Requisition');
    await page.fill('[data-testid=description]', 'Test Description');
    
    // Add item
    await page.click('[data-testid=add-item]');
    await page.fill('[data-testid=item-description]', 'Test Item');
    await page.fill('[data-testid=quantity]', '2');
    await page.fill('[data-testid=unit-price]', '10');
    
    // Submit requisition
    await page.click('[data-testid=submit-requisition]');
    
    // Verify success
    await expect(page.locator('[data-testid=success-message]')).toBeVisible();
    await expect(page.locator('text=Test Requisition')).toBeVisible();
  });
});
```

### Page Testing
```typescript
// Example page test
test.describe('Vendor Management', () => {
  test('should display vendor list', async ({ page }) => {
    await page.goto('/admin/vendors');
    
    await expect(page.locator('[data-testid=vendor-list]')).toBeVisible();
    await expect(page.locator('[data-testid=add-vendor-button]')).toBeVisible();
  });
  
  test('should create new vendor', async ({ page }) => {
    await page.goto('/admin/vendors');
    
    await page.click('[data-testid=add-vendor-button]');
    await page.fill('[data-testid=vendor-name]', 'New Vendor');
    await page.fill('[data-testid=vendor-email]', 'vendor@test.com');
    await page.click('[data-testid=save-vendor]');
    
    await expect(page.locator('text=New Vendor')).toBeVisible();
  });
});
```

## Test Data Management

### Fixtures
```typescript
// Example test fixtures
export const testUsers = {
  requestor: {
    id: '1',
    email: 'requestor@test.com',
    name: 'Test Requestor',
    role: 'REQUESTOR'
  },
  approver: {
    id: '2',
    email: 'approver@test.com',
    name: 'Test Approver',
    role: 'APPROVER'
  },
  admin: {
    id: '3',
    email: 'admin@test.com',
    name: 'Test Admin',
    role: 'ADMIN'
  }
};

export const testVendors = [
  {
    id: '1',
    name: 'Test Vendor 1',
    contactInfo: 'vendor1@test.com'
  },
  {
    id: '2',
    name: 'Test Vendor 2',
    contactInfo: 'vendor2@test.com'
  }
];
```

### Mock Data
```typescript
// Example mock data
export const mockRequisitions = [
  {
    id: '1',
    title: 'Test Requisition 1',
    status: 'DRAFT',
    totalAmount: 100,
    requestorId: '1',
    items: [
      {
        id: '1',
        customItem: 'Test Item',
        quantity: 2,
        unitPrice: 50,
        totalPrice: 100
      }
    ]
  }
];
```

## Test Configuration

### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup/jest.setup.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@tests/(.*)$': '<rootDir>/tests/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{js,jsx,ts,tsx}'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Quality Assurance

### Code Coverage
- Minimum 80% coverage for all metrics
- Coverage reports generated on every build
- Coverage badges in README
- Coverage trends tracked over time

### Performance Testing
```typescript
// Example performance test
import { performance } from 'perf_hooks';

describe('Performance Tests', () => {
  it('should load requisitions within 500ms', async () => {
    const start = performance.now();
    
    const requisitions = await getRequisitions();
    
    const end = performance.now();
    const duration = end - start;
    
    expect(duration).toBeLessThan(500);
    expect(requisitions).toBeDefined();
  });
});
```

### Accessibility Testing
```typescript
// Example accessibility test
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Accessibility Tests', () => {
  it('should not have accessibility violations', async () => {
    const { container } = render(<RequisitionForm />);
    const results = await axe(container);
    
    expect(results).toHaveNoViolations();
  });
});
```

## Continuous Integration

### GitHub Actions
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'pnpm'
    
    - name: Install dependencies
      run: pnpm install
    
    - name: Run unit tests
      run: pnpm test:unit
    
    - name: Run integration tests
      run: pnpm test:integration
    
    - name: Run E2E tests
      run: pnpm test:e2e
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## Test Maintenance

### Regular Tasks
- Update tests when features change
- Remove obsolete tests
- Refactor test code for maintainability
- Review test coverage regularly

### Best Practices
- Keep tests independent
- Use descriptive test names
- Test edge cases and error conditions
- Mock external dependencies
- Maintain test data consistency

### Troubleshooting
- Debug failing tests locally
- Check test environment setup
- Verify mock data accuracy
- Review test isolation issues
