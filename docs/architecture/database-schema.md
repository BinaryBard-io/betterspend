# BetterSpend Database Schema

## Overview
This document describes the complete database schema for the BetterSpend procurement system, including all tables, relationships, and constraints.

## Database Technology
- **Database**: PostgreSQL
- **ORM**: Prisma
- **Migration Tool**: Prisma Migrate

## Core Tables

### Users Table
Stores user information and authentication data.

```prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  role          Role      @default(REQUESTOR)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  // Relations
  accounts      Account[]
  sessions      Session[]
  requisitions  Requisition[]
  approvalSteps ApprovalStep[]

  @@map("users")
}

enum Role {
  REQUESTOR
  APPROVER
  ADMIN
  SUPER_ADMIN
}
```

### Vendors Table
Stores vendor information and contact details.

```prisma
model Vendor {
  id          String    @id @default(cuid())
  name        String    @unique
  contactInfo String?
  address     String?
  phone       String?
  email       String?
  website     String?
  isActive    Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  // Relations
  products    Product[]

  @@map("vendors")
}
```

### Products Table
Stores product catalog information.

```prisma
model Product {
  id          String     @id @default(cuid())
  name        String     @unique
  description String?
  price       Float
  category    String?
  sku         String?    @unique
  isActive    Boolean    @default(true)
  vendorId    String
  vendor      Vendor     @relation(fields: [vendorId], references: [id])
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  // Relations
  requisitionItems RequisitionItem[]

  @@map("products")
}
```

### Budgets Table
Stores budget information and tracking.

```prisma
model Budget {
  id          String    @id @default(cuid())
  name        String    @unique
  description String?
  amount      Float
  spent       Float     @default(0)
  remaining   Float     @default(0)
  startDate   DateTime?
  endDate     DateTime?
  isActive    Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  // Relations
  requisitions Requisition[]

  @@map("budgets")
}
```

### Requisitions Table
Stores purchase requisition information.

```prisma
model Requisition {
  id            String              @id @default(cuid())
  title         String
  description   String?
  requestorId   String
  requestor     User                @relation(fields: [requestorId], references: [id])
  status        RequisitionStatus   @default(DRAFT)
  totalAmount   Float               @default(0)
  budgetId      String?
  budget        Budget?             @relation(fields: [budgetId], references: [id])
  submittedAt   DateTime?
  approvedAt    DateTime?
  rejectedAt    DateTime?
  createdAt     DateTime            @default(now())
  updatedAt     DateTime            @updatedAt

  // Relations
  approvalSteps ApprovalStep[]
  items         RequisitionItem[]

  @@map("requisitions")
}

enum RequisitionStatus {
  DRAFT
  PENDING_APPROVAL
  APPROVED
  REJECTED
  PURCHASED
  CANCELLED
}
```

### Requisition Items Table
Stores individual items within requisitions.

```prisma
model RequisitionItem {
  id            String      @id @default(cuid())
  requisitionId String
  requisition   Requisition @relation(fields: [requisitionId], references: [id])
  productId     String?
  product       Product?    @relation(fields: [productId], references: [id])
  customItem    String?
  description   String?
  quantity      Int
  unitPrice     Float
  totalPrice    Float
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt

  @@map("requisition_items")
}
```

### Approval Steps Table
Stores approval workflow information.

```prisma
model ApprovalStep {
  id            String         @id @default(cuid())
  requisitionId String
  requisition   Requisition    @relation(fields: [requisitionId], references: [id])
  approverId    String
  approver      User           @relation(fields: [approverId], references: [id])
  status        ApprovalStatus @default(PENDING)
  notes         String?
  approvedAt    DateTime?
  rejectedAt    DateTime?
  order         Int
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt

  @@map("approval_steps")
}

enum ApprovalStatus {
  PENDING
  APPROVED
  REJECTED
}
```

## Authentication Tables (NextAuth.js)

### Accounts Table
Stores OAuth account information.

```prisma
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}
```

### Sessions Table
Stores user session information.

```prisma
model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}
```

### Verification Tokens Table
Stores email verification tokens.

```prisma
model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
  @@map("verification_tokens")
}
```

## Relationships

### User Relationships
- **One-to-Many**: User → Requisitions (as requestor)
- **One-to-Many**: User → ApprovalSteps (as approver)
- **One-to-Many**: User → Accounts (OAuth)
- **One-to-Many**: User → Sessions

### Vendor Relationships
- **One-to-Many**: Vendor → Products

### Product Relationships
- **Many-to-One**: Product → Vendor
- **One-to-Many**: Product → RequisitionItems

### Budget Relationships
- **One-to-Many**: Budget → Requisitions

### Requisition Relationships
- **Many-to-One**: Requisition → User (requestor)
- **Many-to-One**: Requisition → Budget
- **One-to-Many**: Requisition → RequisitionItems
- **One-to-Many**: Requisition → ApprovalSteps

### Requisition Item Relationships
- **Many-to-One**: RequisitionItem → Requisition
- **Many-to-One**: RequisitionItem → Product (optional)

### Approval Step Relationships
- **Many-to-One**: ApprovalStep → Requisition
- **Many-to-One**: ApprovalStep → User (approver)

## Indexes and Constraints

### Primary Keys
All tables use `cuid()` for primary keys, providing:
- Collision resistance
- URL-safe strings
- Sortable by creation time

### Unique Constraints
- User email addresses
- Vendor names
- Product names and SKUs
- Budget names
- Account provider combinations
- Session tokens

### Foreign Key Constraints
All foreign key relationships include:
- Referential integrity
- Cascade deletes where appropriate
- Proper indexing for performance

### Indexes
Recommended indexes for performance:
- User email (unique)
- Vendor name (unique)
- Product name and SKU (unique)
- Requisition status
- Approval step status
- Date fields for reporting

## Data Validation

### Business Rules
- Requisition total amount must equal sum of item totals
- Approval steps must be in sequential order
- Budget remaining amount cannot be negative
- Product prices must be positive
- Quantities must be positive integers

### Database Constraints
- NOT NULL constraints on required fields
- CHECK constraints for positive values
- UNIQUE constraints for business keys
- FOREIGN KEY constraints for referential integrity

## Migration Strategy

### Initial Migration
1. Create authentication tables (NextAuth.js)
2. Create core business tables
3. Add indexes and constraints
4. Seed initial data

### Schema Evolution
1. Use Prisma Migrate for schema changes
2. Test migrations on development database
3. Backup production before migration
4. Monitor performance after migration

### Data Seeding
- Initial admin user creation
- Default budget categories
- Sample vendors and products
- Test data for development

## Performance Considerations

### Query Optimization
- Proper indexing strategy
- Efficient relationship loading
- Pagination for large datasets
- Caching for frequently accessed data

### Database Maintenance
- Regular VACUUM and ANALYZE
- Index maintenance
- Connection pooling
- Query performance monitoring

## Security Considerations

### Data Protection
- Encrypted sensitive data
- Audit trail for changes
- Access control at database level
- Regular security updates

### Backup Strategy
- Daily automated backups
- Point-in-time recovery
- Cross-region replication
- Backup verification procedures
