# 🚀 HarborList Admin Quick Reference

## 🔧 Setup Commands

### Create Super Admin
```bash
cd backend
npm run create-admin -- --email admin@yourcompany.com --name "Super Admin" --role super_admin
```

### Create Moderator
```bash
npm run create-admin -- --email mod@yourcompany.com --name "Content Moderator" --role moderator
```

### Local Development Setup
```bash
cd backend
./scripts/setup-local-admin.sh
```

## 🌐 Access URLs

| Environment | Admin Login URL |
|-------------|----------------|
| **Production** | `https://yourdomain.com/admin/login` |
| **Staging** | `https://staging.yourdomain.com/admin/login` |
| **Local Dev** | `http://localhost:3000/admin/login` |

## 👥 Admin Roles & Permissions

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Super Admin** | All permissions | Platform owners |
| **Admin** | User mgmt, moderation, analytics, audit | General admins |
| **Moderator** | Content moderation, audit logs | Content managers |
| **Support** | Audit logs only | Customer support |

## 🔐 First Login Checklist

- [ ] 1. Navigate to admin login URL
- [ ] 2. Enter email and generated password
- [ ] 3. Set up MFA (scan QR code with authenticator app)
- [ ] 4. Verify MFA with 6-digit code
- [ ] 5. Change password in profile settings
- [ ] 6. Save backup codes securely

## 🛠️ Troubleshooting

### Can't Create Admin
```bash
# Check environment variables
echo $USERS_TABLE
echo $AWS_REGION

# Verify AWS credentials
aws sts get-caller-identity
```

### Can't Access Admin Panel
1. Check if frontend is running
2. Verify API endpoints are accessible
3. Check browser console for errors
4. Ensure CORS is configured

### Forgot Admin Password
```bash
# Reset password by creating admin with same email
npm run create-admin -- --email existing@email.com --name "Admin" --role super_admin --password "NewPassword123!"
```

## 📱 MFA Setup

### Recommended Apps
- **Google Authenticator** (iOS/Android)
- **Authy** (iOS/Android/Desktop)
- **Microsoft Authenticator** (iOS/Android)
- **1Password** (Premium feature)

### Setup Process
1. Scan QR code with authenticator app
2. Enter 6-digit verification code
3. Save backup codes in secure location
4. Test MFA on next login

## 🔒 Security Reminders

- ✅ Always use MFA for admin accounts
- ✅ Use strong, unique passwords
- ✅ Change passwords every 90 days
- ✅ Monitor audit logs regularly
- ✅ Limit admin accounts to necessary personnel
- ❌ Never share admin credentials
- ❌ Don't disable MFA except in emergencies

## 📊 Admin Panel Navigation

```
/admin                    → Dashboard
/admin/users             → User Management
/admin/moderation        → Content Moderation
/admin/analytics         → Analytics & Reports
/admin/monitoring        → System Monitoring
/admin/audit-logs        → Audit Logs
/admin/settings          → Platform Settings
/admin/support           → Support Dashboard
```

## 🆘 Emergency Contacts

| Issue | Contact |
|-------|---------|
| **System Down** | DevOps Team |
| **Security Incident** | Security Team |
| **Admin Access Issues** | Technical Lead |
| **User Complaints** | Customer Support |

---

**💡 Pro Tip**: Bookmark this page and the admin login URL for quick access!