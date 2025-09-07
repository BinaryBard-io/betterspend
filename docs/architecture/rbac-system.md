# BetterSpend Role-Based Access Control (RBAC) System

## Overview
The BetterSpend system implements a comprehensive Role-Based Access Control (RBAC) system to ensure proper access control and security across all features and data.

## Role Hierarchy

### REQUESTOR
**Default role for new users**
- Create and manage own requisitions
- View own requisition history
- Submit requisitions for approval
- Cancel own draft requisitions

**Permissions:**
- `requisition:create`
- `requisition:read:own`
- `requisition:update:own`
- `requisition:delete:own`
- `requisition:submit:own`

### APPROVER
**Can approve/reject requisitions**
- All REQUESTOR permissions
- Review pending requisitions
- Approve or reject requisitions
- Add approval notes
- View approval history

**Additional Permissions:**
- `requisition:read:all`
- `approval:approve`
- `approval:reject`
- `approval:read:all`

### ADMIN
**Can manage system data**
- All APPROVER permissions
- Manage vendors and products
- Manage budgets
- View system reports
- Manage user roles (except SUPER_ADMIN)

**Additional Permissions:**
- `vendor:create`
- `vendor:read:all`
- `vendor:update:all`
- `vendor:delete:all`
- `product:create`
- `product:read:all`
- `product:update:all`
- `product:delete:all`
- `budget:create`
- `budget:read:all`
- `budget:update:all`
- `budget:delete:all`
- `user:read:all`
- `user:update:role`
- `report:read:all`

### SUPER_ADMIN
**Full system access**
- All ADMIN permissions
- Manage all user roles
- System configuration
- Audit trail access
- Database management

**Additional Permissions:**
- `user:update:role:super_admin`
- `system:configure`
- `audit:read:all`
- `database:manage`

## Permission System

### Permission Structure
```typescript
interface Permission {
  resource: string;    // e.g., 'requisition', 'vendor', 'user'
  action: string;      // e.g., 'create', 'read', 'update', 'delete'
  scope: string;       // e.g., 'own', 'all', 'department'
}
```

### Resource Types
- **requisition**: Purchase requisitions
- **vendor**: Vendor information
- **product**: Product catalog
- **budget**: Budget management
- **user**: User management
- **approval**: Approval workflows
- **report**: Reports and analytics
- **system**: System configuration
- **audit**: Audit trails

### Action Types
- **create**: Create new resources
- **read**: View resources
- **update**: Modify existing resources
- **delete**: Remove resources
- **approve**: Approve workflows
- **reject**: Reject workflows
- **submit**: Submit for approval
- **configure**: System configuration

### Scope Types
- **own**: User's own resources
- **all**: All resources in system
- **department**: Department-specific resources
- **team**: Team-specific resources

## Implementation

### Database Schema
```prisma
enum Role {
  REQUESTOR
  APPROVER
  ADMIN
  SUPER_ADMIN
}

model User {
  id       String @id @default(cuid())
  email    String @unique
  name     String?
  role     Role   @default(REQUESTOR)
  // ... other fields
}
```

### tRPC Middleware
```typescript
// Role-based middleware
const requireRole = (roles: Role[]) => {
  return t.middleware(({ ctx, next }) => {
    if (!ctx.session?.user) {
      throw new TRPCError({ code: 'UNAUTHORIZED' });
    }
    
    if (!roles.includes(ctx.session.user.role)) {
      throw new TRPCError({ code: 'FORBIDDEN' });
    }
    
    return next({
      ctx: {
        session: { ...ctx.session, user: ctx.session.user },
      },
    });
  });
};

// Permission-based middleware
const requirePermission = (resource: string, action: string, scope: string = 'all') => {
  return t.middleware(({ ctx, next }) => {
    if (!ctx.session?.user) {
      throw new TRPCError({ code: 'UNAUTHORIZED' });
    }
    
    const userRole = ctx.session.user.role;
    const hasPermission = checkPermission(userRole, resource, action, scope);
    
    if (!hasPermission) {
      throw new TRPCError({ code: 'FORBIDDEN' });
    }
    
    return next({
      ctx: {
        session: { ...ctx.session, user: ctx.session.user },
      },
    });
  });
};
```

### Protected Procedures
```typescript
// Role-based procedures
export const adminProcedure = t.procedure.use(requireRole(['ADMIN', 'SUPER_ADMIN']));
export const approverProcedure = t.procedure.use(requireRole(['APPROVER', 'ADMIN', 'SUPER_ADMIN']));

// Permission-based procedures
export const createVendorProcedure = t.procedure.use(
  requirePermission('vendor', 'create', 'all')
);

export const readOwnRequisitionsProcedure = t.procedure.use(
  requirePermission('requisition', 'read', 'own')
);
```

## Client-Side Authorization

### Role-Based UI Components
```typescript
// Role-based component rendering
const RequisitionActions = ({ requisition, user }) => {
  return (
    <div>
      {user.role === 'REQUESTOR' && (
        <EditButton requisition={requisition} />
      )}
      {user.role === 'APPROVER' && (
        <ApprovalButtons requisition={requisition} />
      )}
      {user.role === 'ADMIN' && (
        <AdminActions requisition={requisition} />
      )}
    </div>
  );
};
```

### Permission-Based Navigation
```typescript
// Navigation based on permissions
const getNavigationItems = (userRole: Role) => {
  const baseItems = [
    { label: 'My Requisitions', href: '/requisitions/mine', permission: 'requisition:read:own' }
  ];
  
  if (userRole === 'APPROVER' || userRole === 'ADMIN' || userRole === 'SUPER_ADMIN') {
    baseItems.push({
      label: 'Pending Approvals',
      href: '/requisitions/pending',
      permission: 'approval:read:all'
    });
  }
  
  if (userRole === 'ADMIN' || userRole === 'SUPER_ADMIN') {
    baseItems.push(
      { label: 'Vendors', href: '/admin/vendors', permission: 'vendor:read:all' },
      { label: 'Products', href: '/admin/products', permission: 'product:read:all' },
      { label: 'Budgets', href: '/admin/budgets', permission: 'budget:read:all' }
    );
  }
  
  if (userRole === 'SUPER_ADMIN') {
    baseItems.push({
      label: 'Users',
      href: '/admin/users',
      permission: 'user:read:all'
    });
  }
  
  return baseItems.filter(item => hasPermission(userRole, item.permission));
};
```

## Route Protection

### Page-Level Protection
```typescript
// Protected page component
const ProtectedPage = ({ children, requiredRole }: { children: React.ReactNode, requiredRole: Role }) => {
  const { data: session } = useSession();
  
  if (!session) {
    return <LoginPrompt />;
  }
  
  if (!hasRole(session.user.role, requiredRole)) {
    return <AccessDenied />;
  }
  
  return <>{children}</>;
};

// Usage in page components
const AdminVendorsPage = () => {
  return (
    <ProtectedPage requiredRole="ADMIN">
      <VendorManagement />
    </ProtectedPage>
  );
};
```

### API Route Protection
```typescript
// API route protection
export default withAuth(
  async function handler(req: NextApiRequest, res: NextApiResponse) {
    const session = await getServerSession(req, res, authOptions);
    
    if (!session) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (!hasRole(session.user.role, 'ADMIN')) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    // Handle admin-only logic
  }
);
```

## Data Access Control

### Row-Level Security
```typescript
// Filter data based on user role and permissions
const getRequisitionsForUser = async (userId: string, userRole: Role) => {
  const baseQuery = {
    include: {
      requestor: true,
      items: true,
      approvalSteps: true
    }
  };
  
  switch (userRole) {
    case 'REQUESTOR':
      return prisma.requisition.findMany({
        ...baseQuery,
        where: { requestorId: userId }
      });
      
    case 'APPROVER':
      return prisma.requisition.findMany({
        ...baseQuery,
        where: {
          OR: [
            { requestorId: userId },
            { approvalSteps: { some: { approverId: userId, status: 'PENDING' } } }
          ]
        }
      });
      
    case 'ADMIN':
    case 'SUPER_ADMIN':
      return prisma.requisition.findMany(baseQuery);
      
    default:
      throw new Error('Invalid user role');
  }
};
```

### Field-Level Security
```typescript
// Filter sensitive fields based on permissions
const sanitizeUserData = (user: User, requesterRole: Role) => {
  const baseData = {
    id: user.id,
    name: user.name,
    email: user.email
  };
  
  if (requesterRole === 'ADMIN' || requesterRole === 'SUPER_ADMIN') {
    return {
      ...baseData,
      role: user.role,
      createdAt: user.createdAt,
      updatedAt: user.updatedAt
    };
  }
  
  return baseData;
};
```

## Security Best Practices

### Authentication
- Secure session management
- JWT token validation
- Session timeout handling
- Multi-factor authentication support

### Authorization
- Principle of least privilege
- Regular permission audits
- Role-based access reviews
- Temporary permission grants

### Data Protection
- Encrypt sensitive data
- Audit all access attempts
- Implement data retention policies
- Regular security assessments

### Monitoring
- Log all authorization events
- Monitor for privilege escalation
- Alert on suspicious activities
- Regular security reviews

## Testing

### Unit Tests
```typescript
describe('RBAC System', () => {
  it('should allow REQUESTOR to create requisitions', () => {
    const user = { role: 'REQUESTOR' };
    expect(hasPermission(user, 'requisition', 'create', 'own')).toBe(true);
  });
  
  it('should deny REQUESTOR from managing vendors', () => {
    const user = { role: 'REQUESTOR' };
    expect(hasPermission(user, 'vendor', 'create', 'all')).toBe(false);
  });
});
```

### Integration Tests
```typescript
describe('Protected Endpoints', () => {
  it('should allow ADMIN to access vendor management', async () => {
    const session = await createTestSession({ role: 'ADMIN' });
    const response = await request(app)
      .get('/api/vendors')
      .set('Authorization', `Bearer ${session.token}`);
    
    expect(response.status).toBe(200);
  });
});
```

## Maintenance

### Role Management
- Regular role reviews
- Permission audits
- User access reviews
- Role hierarchy updates

### Security Updates
- Regular security patches
- Permission model updates
- Authentication improvements
- Monitoring enhancements
