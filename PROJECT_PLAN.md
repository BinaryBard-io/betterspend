# Project Plan: BetterSpend - A Modern Procurement System

**Goal:** Develop an MVP for BetterSpend, a comprehensive procurement system, using the T3 Stack.

**Technology Stack:**
*   **Framework:** Next.js (App Router)
*   **Language:** TypeScript
*   **Database:** PostgreSQL (via Prisma)
*   **ORM:** Prisma
*   **Authentication:** NextAuth.js
*   **API Layer:** tRPC
*   **Styling:** Tailwind CSS

---

## Phase 1: Foundation & Infrastructure (Days 1-3)

**Objective:** Set up the core T3 App, database connection, and foundational authentication.

1.  **Initialize T3 App:**
    *   Execute `pnpm create t3-app@latest`.
    *   Select `Next.js`, `TypeScript`, `Prisma`, `NextAuth.js`, `tRPC`, `Tailwind CSS`.
    *   Choose `pnpm` as the package manager.
    *   Confirm App Router.
2.  **Database Setup (PostgreSQL):**
    *   **Prisma Configuration:** Configure `prisma/schema.prisma` for PostgreSQL.
    *   **Environment Variables:** Set `DATABASE_URL` in `.env`.
    *   **Initial Migration:**
        *   `npx prisma migrate dev --name init` to create the initial database schema based on NextAuth.js's default models.
        *   `npx prisma db push` (for development, to sync schema).
3.  **NextAuth.js Configuration:**
    *   **Providers:** Configure at least one authentication provider (e.g., Google, Discord, or Credentials for initial testing) in `src/server/auth.ts`.
    *   **Database Adapter:** Ensure Prisma Adapter is correctly configured in `src/server/auth.ts` to store user sessions and accounts.
    *   **API Route:** Verify `src/app/api/auth/[...nextauth]/route.ts` is correctly set up.
4.  **tRPC Core Setup:**
    *   Verify `src/server/api/trpc.ts` (context, middleware).
    *   Verify `src/server/api/root.ts` (empty router).
    *   Verify `src/app/api/trpc/[trpc]/route.ts` (API handler).
    *   Verify client setup in `src/trpc/react.tsx` and `src/trpc/server.ts`.
5.  **Basic Layout & Styling:**
    *   Configure `tailwind.config.ts` for any custom branding or theme.
    *   Ensure `src/styles/globals.css` imports Tailwind directives.
    *   Create a simple `src/app/layout.tsx` with basic header/footer for navigation and user status (login/logout).

---

## Phase 2: Core Data Models & API Endpoints (Days 4-8)

**Objective:** Define Prisma models for core entities and expose basic CRUD operations via tRPC.

1.  **Prisma Schema Definition (`prisma/schema.prisma`):**
    *   **User Enhancements:** Add `role` field to the `User` model (e.g., `role Role @default(REQUESTOR)`). Define `enum Role { REQUESTOR APPROVER ADMIN SUPER_ADMIN }`.
    *   **Vendor Model:**
        ```prisma
        model Vendor {
          id          String    @id @default(cuid())
          name        String    @unique
          contactInfo String?
          address     String?
          products    Product[]
          createdAt   DateTime  @default(now())
          updatedAt   DateTime  @updatedAt
        }
        ```
    *   **Product Model:**
        ```prisma
        model Product {
          id          String     @id @default(cuid())
          name        String     @unique
          description String?
          price       Float
          vendorId    String
          vendor      Vendor     @relation(fields: [vendorId], references: [id])
          // Link to RequisitionItem later
          createdAt   DateTime   @default(now())
          updatedAt   DateTime   @updatedAt
        }
        ```
    *   **Budget Model:**
        ```prisma
        model Budget {
          id        String    @id @default(cuid())
          name      String    @unique
          amount    Float
          spent     Float     @default(0)
          remaining Float     @default(0) // Calculated or updated
          // Link to Departments/Users if applicable
          createdAt DateTime  @default(now())
          updatedAt DateTime  @updatedAt
        }
        ```
    *   **Requisition & RequisitionItem Models:**
        ```prisma
        enum RequisitionStatus {
          DRAFT
          PENDING_APPROVAL
          APPROVED
          REJECTED
          PURCHASED
          CANCELLED
        }

        model Requisition {
          id            String              @id @default(cuid())
          title         String
          description   String?
          requestorId   String
          requestor     User                @relation(fields: [requestorId], references: [id])
          status        RequisitionStatus   @default(DRAFT)
          totalAmount   Float               @default(0) // Calculated
          budgetRef     String? // Link to Budget.id later
          approvalSteps ApprovalStep[]
          items         RequisitionItem[]
          createdAt     DateTime            @default(now())
          updatedAt     DateTime            @updatedAt
        }

        model RequisitionItem {
          id            String      @id @default(cuid())
          requisitionId String
          requisition   Requisition @relation(fields: [requisitionId], references: [id])
          productId     String? // Optional if custom item
          product       Product?    @relation(fields: [productId], references: [id])
          customItem    String? // For items not in Product catalog
          quantity      Int
          unitPrice     Float
          totalPrice    Float       // quantity * unitPrice
          createdAt     DateTime    @default(now())
          updatedAt     DateTime    @updatedAt
        }
        ```
    *   **ApprovalStep Model (Basic for now):**
        ```prisma
        enum ApprovalStatus {
          PENDING
          APPROVED
          REJECTED
        }

        model ApprovalStep {
          id            String         @id @default(cuid())
          requisitionId String
          requisition   Requisition    @relation(fields: [requisitionId], references: [id])
          approverId    String         // Who is supposed to approve
          approver      User           @relation(fields: [approverId], references: [id])
          status        ApprovalStatus @default(PENDING)
          notes         String?
          approvedAt    DateTime?
          order         Int            // For sequential approvals
          createdAt     DateTime       @default(now())
          updatedAt     DateTime       @updatedAt
        }
        ```
2.  **Run Prisma Migrations:**
    *   `npx prisma migrate dev --name add_betterspend_models`
    *   Update `prisma/seed.ts` (if applicable) for initial data.
3.  **tRPC Router & Procedures (`src/server/api/routers/`):**
    *   **`_app.ts`:** (Rename `example.ts` to `_app.ts` if needed, or add new files and combine in `root.ts`).
    *   Create dedicated routers:
        *   `vendor.ts`: `create`, `getAll`, `getById`, `update`, `delete` procedures.
        *   `product.ts`: `create`, `getAll`, `getById`, `update`, `delete` procedures.
        *   `budget.ts`: `create`, `getAll`, `getById`, `update`, `delete` procedures.
        *   `requisition.ts`: `createDraft`, `submitForApproval`, `getById`, `getAllForRequestor`, `getAllForApprover` procedures.
        *   `approval.ts`: `approveStep`, `rejectStep` procedures.
    *   **Input & Output Schemas:** Use Zod for type-safe validation for all tRPC inputs.
    *   **Combine Routers:** Integrate new routers into `src/server/api/root.ts`.

---

## Phase 3: Authentication & Authorization (Days 9-11)

**Objective:** Implement role-based access control (RBAC) across the application.

1.  **User Role Management (Super Admin only):**
    *   **tRPC Procedure:** Add a `user.ts` router with a `updateUserRole` procedure (ensure this is restricted to `SUPER_ADMIN`).
    *   **Frontend Page:** Create an `src/app/admin/users/page.tsx` for `SUPER_ADMIN` to view and update user roles.
2.  **tRPC Middleware for RBAC:**
    *   Create tRPC middleware in `src/server/api/trpc.ts` to check the user's role before executing procedures.
    *   Apply this middleware to relevant procedures (e.g., `ADMIN` for Vendor/Product management, `APPROVER` for approval actions).
    *   Example:
        ```typescript
        // In src/server/api/trpc.ts
        import { TRPCError } from '@trpc/server';
        import { t } from '../trpc'; // Assuming 't' is your trpc instance

        const isAdmin = t.middleware(({ ctx, next }) => {
          if (!ctx.session?.user || ctx.session.user.role !== 'ADMIN') {
            throw new TRPCError({ code: 'UNAUTHORIZED' });
          }
          return next({
            ctx: {
              // infers the `session` as non-nullable
              session: ctx.session,
            },
          });
        });

        export const adminProcedure = t.procedure.use(isAdmin);
        ```
3.  **Client-Side Role Checks:** Implement logic in Next.js components to conditionally render UI elements based on the logged-in user's role.

---

## Phase 4: Requisition Lifecycle (MVP) (Days 12-18)

**Objective:** Implement the core requisition creation, submission, viewing, and approval/rejection flow.

1.  **Create Requisition (Requestor):**
    *   **Frontend Page:** `src/app/requisitions/create/page.tsx`.
    *   **UI Components:** Form fields for title, description, and dynamic addition of `RequisitionItem`s (selecting existing `Product`s or adding custom items).
    *   **tRPC Integration:** Call `requisition.createDraft` and `requisition.addItem` procedures.
2.  **View Requisitions (Requestor/Approver/Admin):**
    *   **Frontend Page:**
        *   `src/app/requisitions/mine/page.tsx` (Requestor: their own).
        *   `src/app/requisitions/pending/page.tsx` (Approver: requisitions awaiting their approval).
        *   `src/app/requisitions/[id]/page.tsx` (Detailed view for any role with permissions).
    *   **UI Components:** Tables/lists to display requisitions, showing status, total amount, requestor, etc.
    *   **tRPC Integration:** Use `requisition.getAllForRequestor`, `requisition.getAllForApprover`, `requisition.getById` procedures.
3.  **Submit for Approval (Requestor):**
    *   **tRPC Procedure:** `requisition.submitForApproval` (updates status and creates an initial `ApprovalStep` for a default approver).
    *   **UI:** A "Submit" button on the requisition detail/create page.
4.  **Approve/Reject Requisition (Approver):**
    *   **tRPC Procedures:** `approval.approveStep`, `approval.rejectStep`. These procedures should update the `ApprovalStep` status and potentially the `Requisition` status if all steps are complete or rejected.
    *   **UI:** Buttons on the `requisitions/[id]/page.tsx` for `APPROVER` role to interact with.
    *   **Basic Approval Flow:** For MVP, assume a single, fixed approver for simplicity. (The configurable flow comes later).
5.  **Status Updates:** Ensure the UI dynamically reflects changes in `RequisitionStatus` and `ApprovalStatus`.

---

## Phase 5: Master Data Management (MVP) (Days 19-24)

**Objective:** Enable administrators to manage Vendors, Products, and Budgets.

1.  **Vendor Management (Admin):**
    *   **Frontend Page:** `src/app/admin/vendors/page.tsx` (list, create, edit, delete).
    *   **UI Components:** Forms for `Vendor` details, table for listing.
    *   **tRPC Integration:** Use `vendor.getAll`, `vendor.create`, `vendor.update`, `vendor.delete` procedures.
2.  **Product Management (Admin):**
    *   **Frontend Page:** `src/app/admin/products/page.tsx`.
    *   **UI Components:** Forms for `Product` details (including selecting a `Vendor`), table for listing.
    *   **tRPC Integration:** Use `product.getAll`, `product.create`, `product.update`, `product.delete` procedures.
3.  **Budget Management (Admin):**
    *   **Frontend Page:** `src/app/admin/budgets/page.tsx`.
    *   **UI Components:** Forms for `Budget` details, table for listing.
    *   **tRPC Integration:** Use `budget.getAll`, `budget.create`, `budget.update` procedures.
    *   **Budget Association:** Implement basic linking of requisitions to a budget. Update `Budget.spent` when a requisition is `APPROVED`.

---

## Phase 6: Frontend Polish & User Experience (Days 25-28)

**Objective:** Enhance the UI with Tailwind CSS and ensure a smooth user experience.

1.  **Component Library:** Create reusable UI components (buttons, input fields, tables, modals) using Tailwind CSS. Place them in `src/app/_components`.
2.  **Navigation:** Implement intuitive navigation for different user roles.
3.  **Responsiveness:** Ensure the application is responsive across different screen sizes using Tailwind's utility classes.
4.  **Notifications/Toasts:** Add simple user feedback for actions (e.g., "Requisition created!", "Approval successful").
5.  **Error Handling:** Display user-friendly error messages from tRPC operations.

---

## Phase 7: Deployment Preparation & Testing (Days 29-30)

**Objective:** Prepare the application for deployment and conduct initial testing.

1.  **Environment Variables:** Review `.env` and `.env.example` for all necessary production variables.
2.  **Testing:**
    *   **Manual Testing:** Test all MVP features across different user roles.
    *   **Unit/Integration Tests:** (Optional for MVP, but good to start planning).
3.  **Documentation:** Update `README.md` with setup instructions and basic usage.

---

## Future Considerations (Post-MVP)

*   **Configurable Approval Flows:** Develop a UI and backend logic to define conditional, multi-step approval workflows based on amount, department, etc. This would heavily expand on the `ApprovalStep` model.
*   **Reporting & Analytics:** Build dedicated dashboards and reports for spend analysis, budget utilization, vendor performance, etc.
*   **Notifications:** Email or in-app notifications for approvals, rejections, status changes.
*   **Attachments:** Allow users to attach documents to requisitions.
*   **Audit Trails:** Log all significant actions for compliance.
*   **Internationalization (i18n):** Support multiple languages.