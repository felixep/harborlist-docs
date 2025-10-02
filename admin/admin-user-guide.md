# HarborList Admin Dashboard User Guide

## Table of Contents

1. [Getting Started](#getting-started)
2. [Dashboard Overview](#dashboard-overview)
3. [User Management](#user-management)
4. [Content Moderation](#content-moderation)
5. [System Monitoring](#system-monitoring)
6. [Analytics & Reporting](#analytics--reporting)
7. [Platform Settings](#platform-settings)
8. [Support Management](#support-management)
9. [Audit Logs](#audit-logs)
10. [Security Best Practices](#security-best-practices)
11. [Troubleshooting](#troubleshooting)

## Getting Started

### Accessing the Admin Dashboard

1. Navigate to `https://admin.harbotlist.com`
2. Enter your admin credentials
3. Complete two-factor authentication if enabled
4. You'll be redirected to the main dashboard

### First Login Setup

Upon your first login, you'll need to:

1. **Update Your Profile**: Click on your name in the top-right corner and update your profile information
2. **Enable Two-Factor Authentication**: Go to Security Settings and enable 2FA for enhanced security
3. **Review Permissions**: Familiarize yourself with your role permissions and available features

### Navigation Overview

The admin dashboard uses a sidebar navigation with the following main sections:

- **Dashboard**: Overview of key metrics and system health
- **Users**: User management and account operations
- **Moderation**: Content review and listing management
- **Analytics**: Business intelligence and reporting
- **System**: Monitoring and health checks
- **Settings**: Platform configuration and preferences
- **Support**: Customer service and communication tools
- **Audit**: Activity logs and compliance tracking

## Dashboard Overview

The main dashboard provides a comprehensive view of your platform's health and performance.

### Key Metrics Cards

The dashboard displays several metric cards showing:

- **Total Users**: Current registered user count with growth percentage
- **Active Listings**: Number of live boat listings
- **Pending Moderation**: Items requiring review
- **Revenue Today**: Daily revenue with trend indicator
- **System Health**: Overall platform status
- **Response Time**: Average API response time

### Quick Actions

From the dashboard, you can quickly:

- View recent user registrations
- Check flagged content requiring attention
- Monitor system alerts
- Access frequently used admin functions

### Real-time Updates

The dashboard automatically refreshes every 30 seconds to show the latest data. You can also manually refresh using the refresh button in the top-right corner.

## User Management

### Viewing Users

1. Navigate to **Users** in the sidebar
2. The user table displays:
   - Name and email
   - Account status (Active, Suspended, Banned)
   - Registration date
   - Last login
   - Number of listings
   - Verification status

### Searching and Filtering Users

**Search Options:**
- Search by name, email, or user ID
- Use the search bar at the top of the user table
- Results update in real-time as you type

**Filter Options:**
- **Status**: Active, Suspended, Banned, All
- **Role**: User, Premium User, All
- **Verification**: Verified, Unverified, All
- **Registration Date**: Last 7 days, Last 30 days, Custom range

### User Actions

#### Viewing User Details

1. Click the "View" button next to any user
2. The user detail modal shows:
   - Complete profile information
   - Listing history
   - Activity timeline
   - Account statistics
   - Contact information

#### Suspending a User

1. Click the "Actions" dropdown next to the user
2. Select "Suspend User"
3. Fill out the suspension form:
   - **Reason**: Select from predefined reasons or choose "Other"
   - **Duration**: Temporary (specify days) or Permanent
   - **Notes**: Additional details about the suspension
   - **Notify User**: Check to send email notification
4. Click "Confirm Suspension"

#### Reactivating a User

1. Find the suspended user in the user list
2. Click "Actions" → "Reactivate User"
3. Provide a reason for reactivation
4. Click "Confirm Reactivation"

#### Deleting a User Account

⚠️ **Warning**: This action is irreversible and should only be used in extreme cases.

1. Click "Actions" → "Delete User"
2. Type "DELETE" to confirm
3. Provide a detailed reason
4. Click "Permanently Delete User"

### Bulk Operations

For managing multiple users simultaneously:

1. Select users using the checkboxes
2. Choose a bulk action from the dropdown:
   - Send notification
   - Export user data
   - Apply status change
3. Confirm the bulk operation

## Content Moderation

### Moderation Queue

The moderation queue shows all flagged content requiring review:

1. Navigate to **Moderation** in the sidebar
2. View flagged listings organized by:
   - Priority (High, Medium, Low)
   - Flag reason
   - Date reported
   - Reporter type (User, System, Admin)

### Reviewing Flagged Content

#### Opening a Listing for Review

1. Click "Review" next to any flagged listing
2. The listing detail modal displays:
   - Complete listing information
   - All images and descriptions
   - Flag details and reporter information
   - Seller information
   - Listing statistics

#### Making Moderation Decisions

**Approve Listing:**
1. Click "Approve" in the listing detail modal
2. Add review notes (optional)
3. Choose whether to notify the user
4. Click "Confirm Approval"

**Reject Listing:**
1. Click "Reject" in the listing detail modal
2. Select rejection reason:
   - Inappropriate content
   - Misleading information
   - Prohibited items
   - Copyright violation
   - Other (specify)
3. Add detailed notes explaining the rejection
4. Choose notification settings
5. Click "Confirm Rejection"

**Request Changes:**
1. Click "Request Changes"
2. Select what needs to be changed:
   - Title
   - Description
   - Images
   - Price
   - Specifications
3. Provide specific instructions
4. Set a deadline for changes
5. Click "Send Change Request"

### Bulk Moderation

For processing multiple items:

1. Select listings using checkboxes
2. Choose bulk action:
   - Approve all selected
   - Reject all selected
   - Assign to moderator
3. Provide bulk action notes
4. Confirm the operation

### Moderation Statistics

Track your moderation performance:

- **Items Reviewed Today**: Number of decisions made
- **Average Review Time**: Time spent per item
- **Approval Rate**: Percentage of approved items
- **Pending Queue Size**: Items awaiting review

## System Monitoring

### Health Dashboard

Monitor system health in real-time:

1. Navigate to **System** → **Health**
2. View service status indicators:
   - **API Services**: Response time and error rates
   - **Database**: Connection status and performance
   - **Cache**: Hit rates and memory usage
   - **Storage**: Disk usage and availability

### Performance Metrics

Track key performance indicators:

- **Response Times**: API endpoint performance
- **Error Rates**: System error frequency
- **Uptime**: Service availability percentage
- **Resource Usage**: CPU, memory, and disk utilization

### Alerts and Notifications

**Setting Up Alerts:**
1. Go to **System** → **Alerts**
2. Configure thresholds for:
   - Response time limits
   - Error rate thresholds
   - Resource usage limits
3. Set notification preferences:
   - Email alerts
   - SMS notifications
   - Dashboard notifications

**Managing Active Alerts:**
- View current alerts in the alerts panel
- Acknowledge alerts to stop notifications
- Add notes to alert investigations
- Mark alerts as resolved

### System Logs

Access detailed system logs:

1. Navigate to **System** → **Logs**
2. Filter logs by:
   - Service type
   - Log level (Error, Warning, Info, Debug)
   - Time range
   - Search terms
3. Export logs for analysis

## Analytics & Reporting

### Business Analytics

Access comprehensive business intelligence:

1. Navigate to **Analytics**
2. Select date range for analysis
3. View key metrics:
   - User growth trends
   - Listing creation patterns
   - Revenue analytics
   - Geographic distribution

### Custom Reports

Create custom reports:

1. Click "Create Report"
2. Select data sources:
   - User data
   - Listing data
   - Transaction data
   - System metrics
3. Choose visualization type:
   - Charts and graphs
   - Tables
   - Maps
4. Set filters and parameters
5. Generate and save report

### Data Export

Export data for external analysis:

1. Select the data you want to export
2. Choose export format:
   - CSV for spreadsheet analysis
   - JSON for technical analysis
   - PDF for presentations
3. Configure export parameters
4. Download the exported file

### Scheduled Reports

Set up automated reporting:

1. Create a report template
2. Set schedule:
   - Daily, weekly, or monthly
   - Specific days and times
3. Configure recipients
4. Enable email delivery

## Platform Settings

### General Settings

Configure basic platform settings:

1. Navigate to **Settings** → **General**
2. Update:
   - Platform name and branding
   - Contact information
   - Time zone and locale
   - Default currency

### Feature Flags

Enable or disable platform features:

1. Go to **Settings** → **Features**
2. Toggle features on/off:
   - New user registration
   - Listing creation
   - Payment processing
   - Search functionality
3. Set feature rollout percentages for gradual deployment

### Content Policies

Manage platform policies:

1. Navigate to **Settings** → **Policies**
2. Update:
   - Terms of service
   - Privacy policy
   - Community guidelines
   - Listing policies
3. Set policy effective dates
4. Configure user notification settings

### Notification Settings

Configure system notifications:

1. Go to **Settings** → **Notifications**
2. Set up email templates:
   - Welcome emails
   - Suspension notifications
   - Moderation decisions
   - System alerts
3. Configure notification triggers
4. Test email delivery

## Support Management

### Support Ticket System

Manage customer support requests:

1. Navigate to **Support**
2. View ticket queue organized by:
   - Priority (Urgent, High, Normal, Low)
   - Status (Open, In Progress, Resolved, Closed)
   - Category (Technical, Billing, General)
   - Assignment

### Responding to Tickets

1. Click on a ticket to open it
2. Review ticket history and customer information
3. Add response using the rich text editor
4. Attach files if necessary
5. Update ticket status and priority
6. Assign to team member if needed
7. Click "Send Response"

### Ticket Management

**Escalating Tickets:**
1. Open the ticket
2. Click "Escalate"
3. Select escalation reason
4. Assign to senior team member
5. Add escalation notes

**Closing Tickets:**
1. Ensure issue is resolved
2. Add final resolution notes
3. Select closure reason
4. Click "Close Ticket"
5. Customer receives closure notification

### Knowledge Base Management

Maintain self-service resources:

1. Go to **Support** → **Knowledge Base**
2. Create and edit articles:
   - FAQ entries
   - How-to guides
   - Troubleshooting steps
3. Organize articles by category
4. Track article usage and effectiveness

### Announcements

Communicate with users:

1. Navigate to **Support** → **Announcements**
2. Create new announcement:
   - Title and content
   - Target audience (All users, Specific groups)
   - Delivery method (Email, In-app, Both)
   - Schedule delivery time
3. Preview and send announcement

## Audit Logs

### Viewing Audit Logs

Track all administrative actions:

1. Navigate to **Audit**
2. View comprehensive log of:
   - User management actions
   - Content moderation decisions
   - System configuration changes
   - Login/logout events
   - Data access events

### Filtering Audit Logs

Filter logs by:

- **Admin User**: Actions by specific administrators
- **Action Type**: Login, user suspension, content approval, etc.
- **Resource**: Users, listings, settings, etc.
- **Date Range**: Specific time periods
- **IP Address**: Actions from specific locations

### Exporting Audit Data

For compliance and analysis:

1. Set desired filters
2. Click "Export Audit Logs"
3. Choose format (CSV, JSON, PDF)
4. Select date range
5. Download exported file

### Audit Log Retention

Understand data retention policies:

- **Standard Logs**: Retained for 2 years
- **Security Events**: Retained for 7 years
- **Financial Transactions**: Retained for 10 years
- **Compliance Events**: Retained permanently

## Security Best Practices

### Account Security

**Strong Passwords:**
- Use at least 12 characters
- Include uppercase, lowercase, numbers, and symbols
- Avoid common words or personal information
- Use a unique password for the admin account

**Two-Factor Authentication:**
- Enable 2FA immediately upon first login
- Use an authenticator app (Google Authenticator, Authy)
- Keep backup codes in a secure location
- Never share 2FA codes

### Session Management

**Secure Sessions:**
- Always log out when finished
- Don't leave admin sessions unattended
- Use private/incognito browsing on shared computers
- Monitor active sessions in your profile

**Access Controls:**
- Only access admin functions from secure networks
- Avoid public Wi-Fi for admin tasks
- Use VPN when accessing remotely
- Report suspicious login attempts immediately

### Data Protection

**Sensitive Information:**
- Never share user personal data unnecessarily
- Use secure channels for sensitive communications
- Follow data minimization principles
- Respect user privacy rights

**Compliance:**
- Follow GDPR guidelines for EU users
- Respect data retention policies
- Handle data deletion requests promptly
- Document compliance activities

## Troubleshooting

### Common Issues

**Login Problems:**

*Issue*: Cannot log in with correct credentials
*Solution*:
1. Check if Caps Lock is enabled
2. Clear browser cache and cookies
3. Try incognito/private browsing mode
4. Contact system administrator if issue persists

*Issue*: Two-factor authentication not working
*Solution*:
1. Ensure device time is synchronized
2. Try backup codes if available
3. Contact administrator to reset 2FA

**Performance Issues:**

*Issue*: Dashboard loading slowly
*Solution*:
1. Check internet connection
2. Clear browser cache
3. Disable browser extensions temporarily
4. Try different browser

*Issue*: Search results not loading
*Solution*:
1. Refresh the page
2. Check search filters
3. Try simpler search terms
4. Report to technical team if persistent

**Data Issues:**

*Issue*: Metrics showing incorrect data
*Solution*:
1. Check selected date range
2. Verify applied filters
3. Refresh dashboard
4. Contact technical support

*Issue*: Export not working
*Solution*:
1. Check file size limits
2. Try smaller date range
3. Ensure popup blockers are disabled
4. Try different export format

### Getting Help

**Internal Support:**
- Contact your system administrator
- Check internal documentation
- Consult with other admin team members

**Technical Support:**
- Email: admin-support@harbotlist.com
- Phone: 1-800-HARBOR-1
- Emergency line: 1-800-HARBOR-911

**Documentation:**
- API Documentation: `/docs/api`
- Technical Guides: `/docs/technical`
- Video Tutorials: `/docs/videos`

### Reporting Issues

When reporting issues, include:

1. **Description**: What you were trying to do
2. **Steps**: Exact steps to reproduce the issue
3. **Expected Result**: What should have happened
4. **Actual Result**: What actually happened
5. **Browser**: Browser type and version
6. **Screenshots**: Visual evidence of the issue
7. **Error Messages**: Any error messages displayed

### Emergency Procedures

**Security Incidents:**
1. Immediately log out of all sessions
2. Contact security team: security@harbotlist.com
3. Document the incident
4. Do not attempt to investigate on your own

**System Outages:**
1. Check system status page
2. Contact technical team
3. Communicate with stakeholders
4. Document impact and resolution

**Data Breaches:**
1. Immediately contact security team
2. Do not access potentially compromised data
3. Follow incident response procedures
4. Cooperate with investigation

---

## Quick Reference

### Keyboard Shortcuts

- `Ctrl/Cmd + K`: Global search
- `Ctrl/Cmd + R`: Refresh current page
- `Esc`: Close modal dialogs
- `Tab`: Navigate between form fields
- `Enter`: Submit forms

### Status Indicators

- 🟢 **Green**: Healthy/Active/Approved
- 🟡 **Yellow**: Warning/Pending/Under Review
- 🔴 **Red**: Error/Suspended/Rejected
- ⚪ **Gray**: Inactive/Disabled/Unknown

### Contact Information

- **General Support**: support@harbotlist.com
- **Technical Issues**: tech@harbotlist.com
- **Security Concerns**: security@harbotlist.com
- **Emergency**: 1-800-HARBOR-911

---

*Last Updated: September 28, 2024*
*Version: 1.0*