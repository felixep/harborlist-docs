# Financial Transaction Management Integration Guide

## Overview

This document outlines the integration points and requirements for implementing financial transaction management in the HarborList admin dashboard. The current implementation provides placeholder interfaces that need to be connected to actual payment processors.

## Payment Processor Integration Points

### 1. Stripe Integration

#### Required Configuration
```typescript
interface StripeConfig {
  publishableKey: string;
  secretKey: string;
  webhookSecret: string;
  environment: 'test' | 'live';
}
```

#### API Endpoints to Implement
- `GET /admin/transactions` - Fetch transaction history from Stripe
- `POST /admin/transactions/:id/refund` - Process refunds through Stripe
- `GET /admin/disputes` - Fetch dispute information
- `POST /admin/disputes/:id/respond` - Respond to disputes
- `GET /admin/payouts` - Fetch payout schedules
- `POST /admin/payouts/create` - Create manual payouts

#### Webhook Handlers Required
```typescript
// Handle Stripe webhooks for real-time updates
const webhookHandlers = {
  'payment_intent.succeeded': handlePaymentSuccess,
  'payment_intent.payment_failed': handlePaymentFailure,
  'charge.dispute.created': handleDisputeCreated,
  'payout.created': handlePayoutCreated,
  'payout.paid': handlePayoutCompleted,
  'payout.failed': handlePayoutFailed
};
```

### 2. PayPal Integration

#### Required Configuration
```typescript
interface PayPalConfig {
  clientId: string;
  clientSecret: string;
  environment: 'sandbox' | 'live';
  webhookId: string;
}
```

#### API Integration Points
- PayPal REST API for transaction management
- PayPal Disputes API for dispute handling
- PayPal Payouts API for seller payments

### 3. Database Schema Requirements

#### Transactions Table
```sql
CREATE TABLE transactions (
  id VARCHAR(255) PRIMARY KEY,
  type ENUM('payment', 'refund', 'commission', 'payout'),
  amount DECIMAL(10,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  status ENUM('pending', 'completed', 'failed', 'disputed'),
  user_id VARCHAR(255) NOT NULL,
  listing_id VARCHAR(255),
  payment_method VARCHAR(100),
  processor_transaction_id VARCHAR(255),
  processor_type ENUM('stripe', 'paypal', 'square'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP NULL,
  description TEXT,
  fees DECIMAL(10,2) DEFAULT 0,
  net_amount DECIMAL(10,2) NOT NULL,
  metadata JSON,
  INDEX idx_user_id (user_id),
  INDEX idx_listing_id (listing_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

#### Disputes Table
```sql
CREATE TABLE transaction_disputes (
  id VARCHAR(255) PRIMARY KEY,
  transaction_id VARCHAR(255) NOT NULL,
  dispute_reason TEXT NOT NULL,
  dispute_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  dispute_status ENUM('open', 'under_review', 'resolved', 'escalated'),
  dispute_notes TEXT,
  processor_dispute_id VARCHAR(255),
  resolved_at TIMESTAMP NULL,
  resolved_by VARCHAR(255),
  resolution_notes TEXT,
  FOREIGN KEY (transaction_id) REFERENCES transactions(id),
  INDEX idx_transaction_id (transaction_id),
  INDEX idx_status (dispute_status)
);
```

#### Payouts Table
```sql
CREATE TABLE payouts (
  id VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  scheduled_date DATE NOT NULL,
  status ENUM('scheduled', 'processing', 'completed', 'failed'),
  payment_method VARCHAR(100),
  processor_payout_id VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  processed_at TIMESTAMP NULL,
  failure_reason TEXT,
  metadata JSON,
  INDEX idx_user_id (user_id),
  INDEX idx_status (status),
  INDEX idx_scheduled_date (scheduled_date)
);
```

## Backend API Implementation

### Admin Service Endpoints

#### Financial Summary
```typescript
// GET /admin/financial/summary
export const getFinancialSummary = async (req: Request, res: Response) => {
  try {
    const summary = await FinancialService.getSummary();
    res.json(summary);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch financial summary' });
  }
};
```

#### Transaction Management
```typescript
// GET /admin/financial/transactions
export const getTransactions = async (req: Request, res: Response) => {
  try {
    const { page, limit, status, type, startDate, endDate } = req.query;
    const transactions = await FinancialService.getTransactions({
      page: parseInt(page as string) || 1,
      limit: parseInt(limit as string) || 50,
      status: status as string,
      type: type as string,
      dateRange: startDate && endDate ? { startDate, endDate } : undefined
    });
    res.json(transactions);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch transactions' });
  }
};

// POST /admin/financial/transactions/:id/refund
export const processRefund = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { amount, reason } = req.body;
    const refund = await FinancialService.processRefund(id, amount, reason);
    res.json(refund);
  } catch (error) {
    res.status(500).json({ error: 'Failed to process refund' });
  }
};
```

#### Dispute Management
```typescript
// GET /admin/financial/disputes
export const getDisputes = async (req: Request, res: Response) => {
  try {
    const disputes = await FinancialService.getDisputes();
    res.json(disputes);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch disputes' });
  }
};

// POST /admin/financial/disputes/:id/resolve
export const resolveDispute = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { resolution, notes } = req.body;
    const result = await FinancialService.resolveDispute(id, resolution, notes);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: 'Failed to resolve dispute' });
  }
};
```

## Security Considerations

### 1. PCI Compliance
- Never store credit card information directly
- Use payment processor tokens for recurring payments
- Implement proper data encryption for sensitive financial data
- Regular security audits and compliance checks

### 2. Access Control
- Implement role-based access for financial operations
- Require multi-factor authentication for refund processing
- Audit logging for all financial operations
- IP whitelisting for sensitive financial endpoints

### 3. Data Protection
```typescript
// Example of secure financial data handling
interface SecureTransactionData {
  id: string;
  amount: number;
  currency: string;
  status: string;
  // Sensitive data should be masked or encrypted
  paymentMethod: string; // e.g., "****1234" instead of full card number
  userEmail: string; // Consider masking: "j***@example.com"
}
```

## Testing Strategy

### 1. Unit Tests
- Test all financial calculation functions
- Mock payment processor responses
- Test error handling scenarios
- Validate data transformation logic

### 2. Integration Tests
- Test webhook handling
- Verify payment processor API integration
- Test database transaction integrity
- Validate audit logging

### 3. End-to-End Tests
- Complete payment flow testing
- Refund processing workflow
- Dispute resolution process
- Report generation and export

## Monitoring and Alerting

### 1. Key Metrics to Monitor
- Transaction success/failure rates
- Average processing times
- Dispute rates and resolution times
- Revenue and commission tracking
- Payout processing status

### 2. Alert Conditions
- Failed payment processing above threshold
- Unusual dispute activity
- Payout processing failures
- Revenue anomalies
- Security-related events

### 3. Logging Requirements
```typescript
// Example audit log entry
interface FinancialAuditLog {
  timestamp: string;
  adminId: string;
  action: 'refund_processed' | 'dispute_resolved' | 'payout_created';
  resourceId: string;
  details: {
    amount?: number;
    reason?: string;
    previousStatus?: string;
    newStatus?: string;
  };
  ipAddress: string;
  userAgent: string;
}
```

## Implementation Checklist

### Phase 1: Basic Integration
- [ ] Set up payment processor accounts (Stripe/PayPal)
- [ ] Implement webhook endpoints
- [ ] Create database schema
- [ ] Build basic transaction API endpoints
- [ ] Implement transaction listing and filtering

### Phase 2: Advanced Features
- [ ] Implement refund processing
- [ ] Build dispute management system
- [ ] Create payout scheduling
- [ ] Add financial reporting
- [ ] Implement data export functionality

### Phase 3: Security and Compliance
- [ ] Implement PCI compliance measures
- [ ] Add comprehensive audit logging
- [ ] Set up monitoring and alerting
- [ ] Conduct security testing
- [ ] Implement access controls

### Phase 4: Testing and Deployment
- [ ] Write comprehensive test suite
- [ ] Perform load testing
- [ ] Security penetration testing
- [ ] User acceptance testing
- [ ] Production deployment

## Environment Variables Required

```bash
# Stripe Configuration
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# PayPal Configuration
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...
PAYPAL_WEBHOOK_ID=...

# Database
DATABASE_URL=...

# Security
JWT_SECRET=...
ENCRYPTION_KEY=...

# Monitoring
SENTRY_DSN=...
DATADOG_API_KEY=...
```

## Support and Maintenance

### 1. Regular Tasks
- Monitor transaction processing health
- Review dispute resolution metrics
- Update payment processor configurations
- Maintain PCI compliance documentation

### 2. Troubleshooting Common Issues
- Payment processing failures
- Webhook delivery issues
- Dispute escalation procedures
- Payout processing delays

### 3. Documentation Updates
- Keep integration documentation current
- Update API documentation
- Maintain troubleshooting guides
- Document configuration changes

This integration guide provides the foundation for implementing a complete financial transaction management system. The placeholder components created in this task provide the UI structure that will connect to these backend systems once implemented.