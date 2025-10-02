# HarborList Admin Setup Guide

This guide will help you set up admin accounts and access the admin panel for the HarborList platform.

## 🌐 **IMPORTANT: Custom Domains Requirement**

**HarborList is designed to work with custom domains by default.** This is not optional for production deployments.

### **Why Custom Domains Are Required**

1. **🔒 Security**: Cloudflare provides DDoS protection, SSL termination, and security features
2. **🚀 Performance**: Global CDN with edge caching for better user experience
3. **🛡️ Origin Protection**: API Gateway and S3 are protected behind Cloudflare
4. **📊 Analytics**: Cloudflare provides detailed traffic analytics and monitoring
5. **🔧 Flexibility**: Easy SSL management, custom headers, and routing rules

### **When to Use Custom Domains (Recommended)**

✅ **Production deployments**
✅ **Staging environments** 
✅ **Demo environments**
✅ **Any public-facing deployment**

### **When You Can Skip Custom Domains**

❌ **Local development only** (use `useCustomDomains=false`)
❌ **Internal testing** (temporary setups)
❌ **CI/CD pipeline testing** (automated tests)

⚠️ **Warning**: Without custom domains, you'll lose security features, performance benefits, and the frontend may not work correctly due to CORS and routing configurations.

## 🚀 Quick Start

### 1. Prerequisites

Before setting up admin accounts, ensure you have:

- ✅ **Custom domain configured** (see Domain Setup section below)
- ✅ **Cloudflare account** with domain management access
- ✅ **AWS account** with appropriate permissions
- ✅ **Deployed HarborList infrastructure** to AWS
- ✅ **DynamoDB tables** created and accessible
- ✅ **AWS credentials** configured locally or in your environment
- ✅ **Node.js and npm** installed for running setup scripts

## 🌐 Domain Setup (Required)

### **Step 1: Configure Your Domains**

HarborList requires two domains:
- **Frontend Domain**: `harborlist.com` (or `dev.harborlist.com` for dev)
- **API Domain**: `api.harborlist.com` (or `api-dev.harborlist.com` for dev)

### **Step 2: Cloudflare Configuration**

1. **Add your domain to Cloudflare**
2. **Update nameservers** at your domain registrar
3. **Generate Origin Certificate** in Cloudflare SSL/TLS → Origin Server
4. **Import certificate to AWS ACM** (Certificate ARN is in the infrastructure code)

### **Step 3: Deploy Infrastructure**

```bash
# Deploy with custom domains (default)
cd infrastructure
npm run deploy:prod

# Deploy for development
npm run deploy:dev

# Deploy for staging  
npm run deploy:staging
```

### **Step 4: Configure Cloudflare DNS**

After deployment, configure these DNS records in Cloudflare:

```
# Frontend (Cloudflare Tunnel handles this automatically)
harborlist.com → CNAME → [tunnel-id].cfargotunnel.com

# API Gateway
api.harborlist.com → CNAME → [api-gateway-domain-from-outputs]
```

### **Alternative: Deploy Without Custom Domains (Development Only)**

⚠️ **Only for local development and testing**

```bash
cd infrastructure
cdk deploy --context useCustomDomains=false
```

This will:
- ✅ Use default API Gateway URLs
- ✅ Skip Cloudflare integration
- ❌ No DDoS protection
- ❌ No global CDN
- ❌ Manual SSL certificate management
- ❌ Potential CORS issues

### 2. Environment Setup

Set the required environment variables:

```bash
# Required for admin user creation
export USERS_TABLE=boat-users
export AWS_REGION=us-east-1

# Optional: If using custom table names
export AUDIT_LOGS_TABLE=boat-audit-logs
export SESSIONS_TABLE=boat-sessions
export LOGIN_ATTEMPTS_TABLE=boat-login-attempts
```

### 2. Deploy Infrastructure First

**⚠️ CRITICAL**: You must deploy the infrastructure before creating admin users.

```bash
cd infrastructure

# Production deployment (with custom domains)
npm run deploy:prod

# Development deployment (without custom domains)
npm run deploy:dev:no-domains
```

### 3. Create Your First Admin User

**After successful infrastructure deployment**, navigate to the backend directory and run the admin creation script:

```bash
cd backend

# Install dependencies if not already done
npm install

# Create a super admin (recommended for first admin)
npm run create-admin -- --email admin@yourcompany.com --name "Super Admin" --role super_admin

# The script will output a generated password - SAVE IT SECURELY!
```

#### Example Output:
```
✅ Admin user created successfully!
📧 Email: admin@yourcompany.com
👤 Name: Super Admin
🔑 Role: super_admin
🛡️  Permissions: user_management, content_moderation, financial_access, system_config, analytics_view, audit_log_view
🔐 Generated Password: Kx9#mP2$vL8@qR5!

⚠️  IMPORTANT: Save this password securely! It won't be shown again.

📋 Next Steps:
1. Log in to the admin panel at: /admin/login
2. Set up MFA (required for admin accounts)
3. Change the password after first login
```

## 🔐 Admin Account Types & Permissions

### Super Admin (`super_admin`)
- **Full system access** - can perform all administrative functions
- **Permissions**: All available permissions
- **Use case**: Platform owners, technical administrators

### Admin (`admin`)
- **Standard administrative access** - most common admin role
- **Permissions**: 
  - User Management
  - Content Moderation
  - Analytics View
  - Audit Log View
- **Use case**: General administrators, customer service managers

### Moderator (`moderator`)
- **Content-focused access** - limited to content management
- **Permissions**:
  - Content Moderation
  - Audit Log View
- **Use case**: Content moderators, community managers

### Support (`support`)
- **Limited support access** - read-only for most functions
- **Permissions**:
  - Audit Log View
- **Use case**: Customer support representatives

## 🛠️ Advanced Admin Creation

### Create Admin with Custom Password

```bash
npm run create-admin -- \
  --email moderator@yourcompany.com \
  --name "Content Moderator" \
  --role moderator \
  --password "YourSecurePassword123!"
```

### Create Admin with Custom Permissions

```bash
npm run create-admin -- \
  --email support@yourcompany.com \
  --name "Support User" \
  --role support \
  --permissions "user_management,audit_log_view"
```

### Available Permissions

- `user_management` - Manage user accounts, suspend/activate users
- `content_moderation` - Moderate listings, handle reports
- `financial_access` - View financial data, process refunds
- `system_config` - Modify platform settings, configurations
- `analytics_view` - Access analytics and reporting
- `audit_log_view` - View audit logs and system activity

## 🌐 Accessing the Admin Panel

### 1. Admin Login URL

The admin login URL depends on your deployment type:

#### **Production (Custom Domains)**
```
https://harborlist.com/admin/login
```

#### **Staging (Custom Domains)**
```
https://staging.harborlist.com/admin/login
```

#### **Development (Custom Domains)**
```
https://dev.harborlist.com/admin/login
```

#### **Development (No Custom Domains)**
```
https://harborlist.com/admin/login
```

#### **Local Development**
```
http://localhost:3000/admin/login
```

**💡 Tip**: Get the exact URL from your CDK deployment outputs:

```bash
aws cloudformation describe-stacks \
  --stack-name BoatListingStack-prod \
  --query 'Stacks[0].Outputs[?OutputKey==`FrontendUrl`].OutputValue' \
  --output text
```

### 2. First Login Process

1. **Navigate to Admin Login**: Go to `/admin/login`
2. **Enter Credentials**: Use the email and generated password
3. **Set Up MFA**: Admin accounts require Multi-Factor Authentication
   - Scan the QR code with an authenticator app (Google Authenticator, Authy, etc.)
   - Enter the 6-digit code to verify
4. **Change Password**: Update your password after first login
5. **Access Admin Dashboard**: You'll be redirected to `/admin`

### 3. MFA Setup (Required)

All admin accounts must have MFA enabled:

1. After first login, go to your profile settings
2. Click "Set up MFA"
3. Scan the QR code with your authenticator app
4. Enter the verification code
5. Save your backup codes securely

## 🔧 Troubleshooting

### Common Issues

#### "User already exists" Error
```bash
# Check if user exists in database
aws dynamodb query \
  --table-name boat-users \
  --index-name email-index \
  --key-condition-expression "email = :email" \
  --expression-attribute-values '{":email":{"S":"admin@example.com"}}'
```

#### "Table not found" Error
- Verify your `USERS_TABLE` environment variable
- Ensure the DynamoDB table exists in your AWS account
- Check your AWS credentials and region

#### "Access Denied" Error
- Verify your AWS credentials have DynamoDB permissions
- Ensure the IAM role/user has `dynamodb:PutItem` and `dynamodb:Query` permissions

#### Can't Access Admin Panel
- Verify the frontend is deployed and accessible
- Check that the API Gateway endpoints are working
- Ensure CORS is properly configured

### Reset Admin Password

If you need to reset an admin password:

```bash
# Create a new admin with the same email (will update existing)
npm run create-admin -- \
  --email existing-admin@yourcompany.com \
  --name "Admin User" \
  --role super_admin \
  --password "NewSecurePassword123!"
```

### Disable MFA (Emergency Only)

⚠️ **Only use in emergencies** - Admin accounts should always have MFA enabled.

You can temporarily disable MFA by directly updating the DynamoDB record:

```bash
aws dynamodb update-item \
  --table-name boat-users \
  --key '{"id":{"S":"admin-user-id"}}' \
  --update-expression "SET mfaEnabled = :false" \
  --expression-attribute-values '{":false":{"BOOL":false}}'
```

## 📊 Admin Panel Features

Once logged in, admins have access to:

### 🏠 Dashboard
- Platform overview and key metrics
- Recent activity and alerts
- Quick access to common tasks

### 👥 User Management
- View and manage user accounts
- Suspend/activate users
- View user activity and details

### 🛡️ Content Moderation
- Review flagged listings
- Approve/reject content
- Manage content policies

### 📈 Analytics
- User growth and engagement metrics
- Listing performance data
- Geographic distribution

### ⚙️ System Monitoring
- System health and performance
- Error tracking and alerts
- Infrastructure metrics

### 📋 Audit Logs
- Complete audit trail of admin actions
- Search and filter capabilities
- Export functionality for compliance

### 🎛️ Platform Settings
- Configure platform-wide settings
- Manage feature flags
- Update policies and terms

### 💬 Support Dashboard
- Manage support tickets
- Create announcements
- Handle user communications

## 🔒 Security Best Practices

### For Admin Accounts

1. **Use Strong Passwords**: Minimum 12 characters with mixed case, numbers, and symbols
2. **Enable MFA**: Always required for admin accounts
3. **Regular Password Changes**: Update passwords every 90 days
4. **Limit Admin Accounts**: Only create admin accounts for users who need them
5. **Use Appropriate Roles**: Assign the minimum permissions needed
6. **Monitor Activity**: Regularly review audit logs for admin actions

### For Platform Security

1. **Regular Backups**: Ensure DynamoDB tables are backed up
2. **Access Logging**: Monitor all admin panel access
3. **IP Restrictions**: Consider IP whitelisting for admin access
4. **Session Management**: Configure appropriate session timeouts
5. **Regular Updates**: Keep all dependencies and infrastructure updated

## 📞 Support

If you encounter issues with admin setup:

1. Check the troubleshooting section above
2. Review the application logs in CloudWatch
3. Verify your AWS infrastructure is properly deployed
4. Contact your development team for assistance

## 🔄 Maintenance

### Regular Tasks

- **Weekly**: Review audit logs for unusual activity
- **Monthly**: Update admin passwords and review permissions
- **Quarterly**: Audit admin accounts and remove unused ones
- **As needed**: Update MFA backup codes

### Monitoring

Set up CloudWatch alarms for:
- Failed admin login attempts
- Unusual admin activity patterns
- System errors in admin functions
- Database connection issues

---

**⚠️ Important Security Note**: Always treat admin credentials as highly sensitive. Never share passwords, always use MFA, and regularly audit admin access.