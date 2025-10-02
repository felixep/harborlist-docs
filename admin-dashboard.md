# 🛡️ Admin Dashboard Frontend

The admin dashboard provides a secure, role-based interface for platform administration with comprehensive user management, content moderation, and system monitoring capabilities.

## 🏗 **Architecture Overview**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Admin User    │───▶│  Admin Routes    │───▶│  Admin Layout   │
└─────────────────┘    │  (/admin/*)      │    │   + Sidebar     │
         │              └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Admin Auth      │───▶│ Permission Check │───▶│ Admin Features  │
│ JWT + Roles     │    │ Role-Based       │    │ User/Content    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## ✨ **Key Features**

### Security & Authentication
- **🔐 JWT-Based Auth**: Separate admin authentication system
- **👥 Role-Based Access**: Super Admin, Admin, Moderator, Support roles
- **🛡️ Permission System**: Granular permission-based feature access
- **🔒 Protected Routes**: Route-level security with role verification
- **⏱️ Session Management**: Configurable session timeouts

### User Interface
- **📱 Responsive Design**: Mobile-first admin interface
- **🎨 Modern UI**: Clean, professional dashboard design
- **🧭 Intuitive Navigation**: Role-based sidebar navigation
- **📊 Dashboard Overview**: Key metrics and system status
- **🔄 Real-time Updates**: Live data and notifications

### Administrative Features
- **👤 User Management**: View, edit, suspend user accounts
- **📝 Content Moderation**: Review and moderate listings
- **📈 System Monitoring**: Performance and health metrics
- **📊 Analytics Dashboard**: Platform usage statistics
- **⚙️ Platform Settings**: System configuration management

## 📁 **Project Structure**

```
frontend/src/
├── components/admin/           # Admin-specific components
│   ├── AdminLayout.tsx            # Main admin layout wrapper
│   ├── AdminHeader.tsx            # Admin navigation header
│   ├── Sidebar.tsx                # Role-based sidebar navigation
│   ├── AdminProtectedRoute.tsx    # Admin route protection
│   ├── AdminAuthProvider.tsx      # Admin authentication context
│   ├── index.ts                   # Component exports
│   └── __tests__/                 # Component tests
│       ├── AdminLayout.test.tsx
│       ├── AdminHeader.test.tsx
│       ├── Sidebar.test.tsx
│       └── AdminProtectedRoute.test.tsx
├── pages/admin/                # Admin page components
│   ├── AdminDashboard.tsx         # Main dashboard overview
│   ├── AdminLogin.tsx             # Admin login page
│   ├── UserManagement.tsx         # User administration
│   ├── ListingModeration.tsx      # Content moderation
│   ├── SystemMonitoring.tsx       # System health monitoring
│   ├── Analytics.tsx              # Platform analytics
│   ├── PlatformSettings.tsx       # System configuration
│   └── index.ts                   # Page exports
├── hooks/                      # Admin-specific hooks
│   └── useAdminAuth.ts            # Admin authentication hook
└── App.tsx                     # Updated with admin routes
```

## 🔐 **Authentication System**

### Admin Roles & Permissions

```typescript
enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
  MODERATOR = 'moderator',
  SUPPORT = 'support'
}

enum AdminPermission {
  USER_MANAGEMENT = 'user_management',
  CONTENT_MODERATION = 'content_moderation',
  FINANCIAL_ACCESS = 'financial_access',
  SYSTEM_CONFIG = 'system_config',
  ANALYTICS_VIEW = 'analytics_view',
  AUDIT_LOG_VIEW = 'audit_log_view'
}
```

### Role Hierarchy
- **Super Admin**: Full system access, all permissions
- **Admin**: User management, content moderation, analytics
- **Moderator**: Content moderation, limited user actions
- **Support**: User support, limited read access

### AdminAuthProvider Implementation

```typescript
// components/admin/AdminAuthProvider.tsx
export const AdminAuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [adminUser, setAdminUser] = useState<AdminUser | null>(null);
  const [loading, setLoading] = useState(true);

  const hasPermission = (permission: AdminPermission): boolean => {
    if (!adminUser) return false;
    if (adminUser.role === UserRole.SUPER_ADMIN) return true;
    return adminUser.permissions?.includes(permission) || false;
  };

  const hasRole = (role: UserRole): boolean => {
    if (!adminUser) return false;
    return adminUser.role === role;
  };

  // ... authentication logic
};
```

## 🧩 **Core Components**

### AdminLayout Component

**Features:**
- Responsive sidebar navigation
- Mobile-friendly collapsible menu
- Header with user info and logout
- Loading states and error handling

```typescript
// components/admin/AdminLayout.tsx
const AdminLayout: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  const { loading } = useAdminAuth();

  if (loading) {
    return <LoadingSpinner message="Loading admin dashboard..." />;
  }

  return (
    <div className="h-screen flex overflow-hidden bg-gray-100">
      <Sidebar isOpen={isMobileMenuOpen} onClose={() => setIsMobileMenuOpen(false)} />
      <div className="flex flex-col flex-1 overflow-hidden">
        <AdminHeader onMenuToggle={() => setIsMobileMenuOpen(!isMobileMenuOpen)} />
        <main className="flex-1 relative overflow-y-auto">
          {children}
        </main>
      </div>
    </div>
  );
};
```

### Sidebar Navigation

**Features:**
- Role-based menu filtering
- Active state management
- Permission-based visibility
- Mobile overlay support

```typescript
// components/admin/Sidebar.tsx
const menuItems: MenuItem[] = [
  {
    name: 'Dashboard',
    path: '/admin',
    icon: <DashboardIcon />,
  },
  {
    name: 'User Management',
    path: '/admin/users',
    icon: <UsersIcon />,
    permission: AdminPermission.USER_MANAGEMENT,
  },
  {
    name: 'Content Moderation',
    path: '/admin/moderation',
    icon: <ModerationIcon />,
    permission: AdminPermission.CONTENT_MODERATION,
  },
  // ... more menu items
];

const isMenuItemVisible = (item: MenuItem): boolean => {
  if (adminUser?.role === UserRole.SUPER_ADMIN) return true;
  if (item.roles && !item.roles.some(role => hasRole(role))) return false;
  if (item.permission && !hasPermission(item.permission)) return false;
  return true;
};
```

### AdminProtectedRoute

**Features:**
- Route-level authentication
- Permission and role checking
- Graceful access denied handling
- Loading state management

```typescript
// components/admin/AdminProtectedRoute.tsx
const AdminProtectedRoute: React.FC<{
  children: React.ReactNode;
  requiredPermission?: AdminPermission;
  requiredRole?: UserRole;
}> = ({ children, requiredPermission, requiredRole }) => {
  const { isAuthenticated, hasPermission, hasRole, loading } = useAdminAuth();

  if (loading) return <LoadingSpinner />;
  if (!isAuthenticated) return <Navigate to="/admin/login" />;
  if (requiredRole && !hasRole(requiredRole)) return <AccessDenied />;
  if (requiredPermission && !hasPermission(requiredPermission)) return <AccessDenied />;

  return <>{children}</>;
};
```

## 🛣 **Routing Structure**

### Admin Route Configuration

```typescript
// App.tsx - Admin Routes
<Route path="/admin/login" element={<AdminLogin />} />
<Route
  path="/admin/*"
  element={
    <AdminProtectedRoute>
      <AdminLayout>
        <Routes>
          <Route index element={<AdminDashboard />} />
          <Route 
            path="users" 
            element={
              <AdminProtectedRoute requiredPermission={AdminPermission.USER_MANAGEMENT}>
                <UserManagement />
              </AdminProtectedRoute>
            } 
          />
          <Route 
            path="moderation" 
            element={
              <AdminProtectedRoute requiredPermission={AdminPermission.CONTENT_MODERATION}>
                <ListingModeration />
              </AdminProtectedRoute>
            } 
          />
          <Route path="monitoring" element={<SystemMonitoring />} />
          <Route 
            path="analytics" 
            element={
              <AdminProtectedRoute requiredPermission={AdminPermission.ANALYTICS_VIEW}>
                <Analytics />
              </AdminProtectedRoute>
            } 
          />
          <Route 
            path="settings" 
            element={
              <AdminProtectedRoute requiredPermission={AdminPermission.SYSTEM_CONFIG}>
                <PlatformSettings />
              </AdminProtectedRoute>
            } 
          />
        </Routes>
      </AdminLayout>
    </AdminProtectedRoute>
  }
/>
```

### Route Protection Levels

1. **Public Routes**: `/admin/login`
2. **Authenticated Routes**: `/admin/*` (requires admin login)
3. **Permission Routes**: Specific features require permissions
4. **Role Routes**: Some features require specific roles

## 📊 **Admin Pages**

### AdminDashboard
- **Overview**: Key platform metrics and statistics
- **Quick Actions**: Common administrative tasks
- **System Status**: Health indicators and alerts
- **Recent Activity**: Latest user and content activity

### UserManagement
- **User List**: Searchable, filterable user directory
- **User Details**: Comprehensive user profiles
- **Account Actions**: Suspend, activate, delete accounts
- **Role Management**: Assign roles and permissions

### ListingModeration
- **Moderation Queue**: Pending content reviews
- **Listing Actions**: Approve, reject, flag listings
- **Bulk Operations**: Mass moderation tools
- **Moderation History**: Audit trail of actions

### SystemMonitoring
- **Performance Metrics**: System health indicators
- **Error Tracking**: Application error monitoring
- **Usage Statistics**: Platform utilization data
- **Infrastructure Status**: Server and service health

### Analytics
- **User Analytics**: Registration, activity, retention
- **Content Analytics**: Listing creation, views, conversions
- **Financial Analytics**: Revenue and transaction data
- **Custom Reports**: Configurable analytics dashboards

### PlatformSettings
- **System Configuration**: Platform-wide settings
- **Feature Flags**: Enable/disable features
- **Content Policies**: Moderation rules and guidelines
- **Integration Settings**: Third-party service configuration

## 🎨 **Design System**

### Admin Theme
```css
/* Admin-specific color palette */
:root {
  --admin-primary: #1f2937;    /* Dark gray */
  --admin-secondary: #374151;  /* Medium gray */
  --admin-accent: #3b82f6;     /* Blue */
  --admin-success: #10b981;    /* Green */
  --admin-warning: #f59e0b;    /* Amber */
  --admin-danger: #ef4444;     /* Red */
}
```

### Component Styling
- **Sidebar**: Dark theme with blue accents
- **Header**: Clean white background with user info
- **Content**: Light background with card-based layout
- **Forms**: Consistent input styling and validation states

### Responsive Breakpoints
- **Mobile**: < 768px (collapsed sidebar, mobile header)
- **Tablet**: 768px - 1024px (collapsible sidebar)
- **Desktop**: > 1024px (persistent sidebar)

## 🧪 **Testing Strategy**

### Component Testing
```typescript
// __tests__/AdminHeader.test.tsx
describe('AdminHeader', () => {
  it('renders admin header with user info', () => {
    renderWithContext(<AdminHeader onMenuToggle={vi.fn()} />);
    
    expect(screen.getByText('HarborList Admin')).toBeInTheDocument();
    expect(screen.getByText('Test Admin')).toBeInTheDocument();
    expect(screen.getByText('ADMIN')).toBeInTheDocument();
  });

  it('calls logout when sign out is clicked', () => {
    renderWithContext(<AdminHeader onMenuToggle={vi.fn()} />);
    
    fireEvent.click(screen.getByRole('button', { name: /A/i }));
    fireEvent.click(screen.getByText('Sign out'));
    
    expect(mockAuthContext.logout).toHaveBeenCalledOnce();
  });
});
```

### Test Coverage
- **Authentication Flow**: Login, logout, token validation
- **Permission Checking**: Role and permission-based access
- **Navigation**: Sidebar menu visibility and routing
- **Responsive Behavior**: Mobile menu and layout adaptation

### Testing Tools
- **Vitest**: Test runner and assertion library
- **React Testing Library**: Component testing utilities
- **Mock Service Worker**: API mocking for tests
- **User Event**: User interaction simulation

## 🔧 **Development Workflow**

### Local Development
```bash
# Start development server
npm run dev

# Run admin-specific tests
npm test -- --run src/components/admin

# Type checking
npm run type-check

# Build for production
npm run build
```

### Environment Configuration
```bash
# .env.local
VITE_API_URL=https://api.harborlist.com
VITE_ADMIN_API_URL=https://api.harborlist.com/admin
```

## 🚀 **Deployment & Security**

### Security Considerations
- **JWT Validation**: Server-side token verification
- **Role Verification**: Backend permission checking
- **HTTPS Only**: Secure communication channels
- **Session Management**: Automatic token refresh and expiry
- **CSRF Protection**: Cross-site request forgery prevention

### Performance Optimizations
- **Code Splitting**: Admin routes loaded separately
- **Lazy Loading**: Components loaded on demand
- **Caching**: Efficient data caching strategies
- **Bundle Optimization**: Minimal admin bundle size

### Monitoring & Logging
- **Error Tracking**: Admin-specific error monitoring
- **Audit Logging**: Administrative action tracking
- **Performance Monitoring**: Admin interface performance
- **Security Logging**: Authentication and authorization events

## 🔮 **Future Enhancements**

### Planned Features
- **Real-time Notifications**: Live admin alerts and updates
- **Advanced Analytics**: Custom dashboard creation
- **Bulk Operations**: Mass user and content management
- **API Management**: Admin API key and rate limit management
- **Audit Trail**: Comprehensive action history and reporting

### Technical Improvements
- **PWA Support**: Offline admin capabilities
- **Dark Mode**: Admin interface theme options
- **Keyboard Shortcuts**: Power user navigation
- **Export/Import**: Data management utilities
- **Multi-language**: Internationalization support

---

**🛡️ The admin dashboard provides a secure, scalable, and user-friendly interface for comprehensive platform administration.**