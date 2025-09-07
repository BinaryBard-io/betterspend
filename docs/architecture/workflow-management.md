# BetterSpend Workflow Management

## Overview
This document describes the workflow management system in BetterSpend, including requisition lifecycle, approval processes, and status management.

## Requisition Lifecycle

### Status Flow
```
DRAFT → PENDING_APPROVAL → APPROVED → PURCHASED
  ↓           ↓              ↓
CANCELLED  REJECTED      CANCELLED
```

### Status Definitions

#### DRAFT
- Initial state when requisition is created
- Can be modified by the requestor
- Not visible to approvers
- Can be cancelled or submitted for approval

#### PENDING_APPROVAL
- Requisition has been submitted for approval
- Cannot be modified by requestor
- Visible to designated approvers
- Awaiting approval decision

#### APPROVED
- All approval steps completed successfully
- Ready for purchase
- Cannot be modified
- Can be marked as purchased or cancelled

#### REJECTED
- At least one approval step was rejected
- Cannot be modified
- Requestor can create new requisition based on feedback
- Final state

#### PURCHASED
- Items have been purchased
- Final state
- Used for reporting and tracking

#### CANCELLED
- Requisition cancelled at any stage
- Final state
- Cannot be reactivated

## Approval Workflow

### Basic Approval Process
1. **Submission**: Requestor submits requisition
2. **Approval Assignment**: System assigns approvers based on rules
3. **Review**: Approvers review requisition details
4. **Decision**: Approvers approve or reject with notes
5. **Completion**: Workflow completes based on decisions

### Approval Rules

#### Amount-Based Rules
- Requisitions under $1,000: Single approver
- Requisitions $1,000-$10,000: Two approvers
- Requisitions over $10,000: Three approvers

#### Department-Based Rules
- IT purchases: IT Manager approval required
- Office supplies: Office Manager approval required
- Equipment: Department Head approval required

#### Sequential vs Parallel Approval
- **Sequential**: Approvers must approve in order
- **Parallel**: All approvers can approve simultaneously

### Approval Step Management

#### Step Creation
```typescript
interface ApprovalStep {
  id: string;
  requisitionId: string;
  approverId: string;
  status: ApprovalStatus;
  notes?: string;
  order: number;
  approvedAt?: Date;
  rejectedAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}
```

#### Step Status
- **PENDING**: Awaiting approval
- **APPROVED**: Step approved
- **REJECTED**: Step rejected

#### Step Processing
1. Create approval steps based on rules
2. Assign approvers to steps
3. Notify approvers of pending approvals
4. Process approval decisions
5. Update requisition status based on results

## Workflow Engine

### Rule Engine
```typescript
interface ApprovalRule {
  id: string;
  name: string;
  conditions: RuleCondition[];
  actions: RuleAction[];
  priority: number;
  isActive: boolean;
}

interface RuleCondition {
  field: string;
  operator: 'equals' | 'greater_than' | 'less_than' | 'contains';
  value: any;
}

interface RuleAction {
  type: 'assign_approver' | 'send_notification' | 'update_status';
  parameters: Record<string, any>;
}
```

### Rule Evaluation
1. Load active rules for requisition
2. Evaluate conditions against requisition data
3. Execute actions for matching rules
4. Create approval steps based on actions
5. Send notifications to assigned approvers

### Dynamic Workflow
- Rules can be modified without code changes
- New approval paths can be added
- Department-specific workflows
- Emergency approval processes

## Notification System

### Notification Types
- **Approval Required**: New requisition needs approval
- **Approval Decision**: Approval decision made
- **Status Change**: Requisition status updated
- **Reminder**: Pending approval reminder
- **Escalation**: Overdue approval escalation

### Notification Channels
- **In-App**: Real-time notifications in application
- **Email**: Email notifications to users
- **SMS**: Text messages for urgent approvals
- **Slack**: Integration with team communication

### Notification Rules
- Immediate notifications for new approvals
- Daily reminders for pending approvals
- Escalation after 3 days of no action
- Final reminder before auto-rejection

## Status Management

### Status Transitions
```typescript
const statusTransitions = {
  DRAFT: ['PENDING_APPROVAL', 'CANCELLED'],
  PENDING_APPROVAL: ['APPROVED', 'REJECTED', 'CANCELLED'],
  APPROVED: ['PURCHASED', 'CANCELLED'],
  REJECTED: ['DRAFT'], // Create new requisition
  PURCHASED: [], // Final state
  CANCELLED: [] // Final state
};
```

### Status Validation
- Validate transition is allowed
- Check user permissions for transition
- Verify business rules are met
- Update related data (budgets, notifications)

### Status History
- Track all status changes
- Record who made the change
- Store reason for change
- Maintain audit trail

## Business Rules

### Budget Rules
- Check budget availability before approval
- Reserve budget when approved
- Release budget when rejected/cancelled
- Update budget when purchased

### Validation Rules
- Required fields must be completed
- Amounts must be positive
- Quantities must be reasonable
- Vendors must be active

### Permission Rules
- Requestors can only modify own requisitions
- Approvers can only approve assigned requisitions
- Admins can override approval decisions
- Super admins have full access

## Error Handling

### Validation Errors
- Field validation failures
- Business rule violations
- Permission denials
- Data integrity issues

### Workflow Errors
- Missing approvers
- Invalid approval sequences
- Notification failures
- Status transition errors

### Recovery Procedures
- Retry failed operations
- Manual intervention for stuck workflows
- Data consistency checks
- Audit trail maintenance

## Performance Considerations

### Optimization Strategies
- Cache approval rules
- Batch notification processing
- Index database queries
- Optimize status updates

### Scalability
- Handle high volume of requisitions
- Support multiple concurrent approvals
- Scale notification system
- Optimize database performance

## Monitoring and Analytics

### Workflow Metrics
- Average approval time
- Approval rates by department
- Bottleneck identification
- User performance metrics

### System Health
- Workflow completion rates
- Error rates and types
- Performance metrics
- User satisfaction scores

### Reporting
- Approval trend analysis
- Department comparison
- User activity reports
- System performance reports

## Configuration Management

### Workflow Configuration
- Approval rules management
- Notification settings
- Status definitions
- Business rule parameters

### Environment-Specific Settings
- Development: Simplified workflows
- Staging: Production-like workflows
- Production: Full workflow complexity

### Change Management
- Version control for configurations
- Rollback capabilities
- Change approval process
- Impact assessment

## Testing

### Unit Tests
- Rule engine logic
- Status transition validation
- Notification generation
- Business rule enforcement

### Integration Tests
- End-to-end workflow testing
- Database interaction testing
- Notification system testing
- API endpoint testing

### Performance Tests
- High-volume workflow testing
- Concurrent approval testing
- Database performance testing
- Notification system load testing

## Maintenance

### Regular Tasks
- Review and update approval rules
- Monitor workflow performance
- Clean up old notifications
- Update business rules

### Troubleshooting
- Identify stuck workflows
- Resolve approval conflicts
- Fix notification issues
- Recover from system errors

### Documentation Updates
- Keep workflow documentation current
- Update user guides
- Maintain API documentation
- Document configuration changes
