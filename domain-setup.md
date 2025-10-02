# 🌐 HarborList Domain Setup Guide

This guide explains how to configure custom domains for HarborList, why they're required, and when you might skip them.

## 🎯 **Why Custom Domains Are Essential**

HarborList is architected as a **production-ready platform** with enterprise-grade security and performance. Custom domains aren't just a nice-to-have—they're fundamental to the architecture.

### **🔒 Security Benefits**

| Feature | With Custom Domains | Without Custom Domains |
|---------|-------------------|----------------------|
| **DDoS Protection** | ✅ Cloudflare's global network | ❌ Direct exposure to attacks |
| **SSL/TLS Management** | ✅ Automatic certificate renewal | ❌ Manual ACM certificate management |
| **Origin Protection** | ✅ AWS resources hidden behind Cloudflare | ❌ Direct access to AWS endpoints |
| **Rate Limiting** | ✅ Cloudflare's advanced rate limiting | ❌ Basic API Gateway limits only |
| **Bot Protection** | ✅ Cloudflare's bot management | ❌ No bot protection |
| **WAF (Web Application Firewall)** | ✅ Cloudflare's security rules | ❌ No WAF protection |

### **🚀 Performance Benefits**

| Feature | With Custom Domains | Without Custom Domains |
|---------|-------------------|----------------------|
| **Global CDN** | ✅ 200+ edge locations worldwide | ❌ Single AWS region |
| **Edge Caching** | ✅ Static assets cached globally | ❌ No edge caching |
| **Compression** | ✅ Automatic Brotli/Gzip compression | ❌ No compression |
| **HTTP/2 & HTTP/3** | ✅ Latest protocols supported | ❌ Limited protocol support |
| **Smart Routing** | ✅ Optimal path to origin | ❌ Direct routing only |

### **📊 Operational Benefits**

| Feature | With Custom Domains | Without Custom Domains |
|---------|-------------------|----------------------|
| **Analytics** | ✅ Detailed traffic analytics | ❌ Limited CloudWatch metrics |
| **Monitoring** | ✅ Real-time performance metrics | ❌ Basic AWS monitoring |
| **Alerting** | ✅ Proactive issue detection | ❌ Reactive monitoring only |
| **Debugging** | ✅ Comprehensive request logs | ❌ Limited debugging info |

## 🏗️ **Architecture Overview**

```
Internet → Cloudflare → AWS Infrastructure
    ↓         ↓              ↓
  Users   Security &     API Gateway
         Performance      Lambda
                         DynamoDB
                         S3
```

### **With Custom Domains (Recommended)**
```
harborlist.com → Cloudflare → S3 (Frontend)
api.harborlist.com → Cloudflare → API Gateway → Lambda Functions
```

### **Without Custom Domains (Development Only)**
```
[random-id].s3-website.amazonaws.com → S3 (Frontend)
[random-id].execute-api.amazonaws.com → API Gateway → Lambda Functions
```

## 🛠️ **Domain Configuration Steps**

### **Prerequisites**

- ✅ Domain registered with any registrar
- ✅ Cloudflare account (free tier is sufficient)
- ✅ AWS account with appropriate permissions
- ✅ Access to domain's DNS settings

### **Step 1: Add Domain to Cloudflare**

1. **Log in to Cloudflare Dashboard**
2. **Click "Add a Site"**
3. **Enter your domain** (e.g., `harborlist.com`)
4. **Select plan** (Free plan works fine)
5. **Cloudflare will scan existing DNS records**

### **Step 2: Update Nameservers**

1. **Copy nameservers** from Cloudflare (e.g., `alice.ns.cloudflare.com`)
2. **Go to your domain registrar** (GoDaddy, Namecheap, etc.)
3. **Update nameservers** to point to Cloudflare
4. **Wait for propagation** (can take up to 24 hours)

### **Step 3: Generate Origin Certificate**

1. **In Cloudflare Dashboard** → SSL/TLS → Origin Server
2. **Click "Create Certificate"**
3. **Select "Let Cloudflare generate a private key and a CSR"**
4. **Add hostnames**:
   ```
   *.harborlist.com
   harborlist.com
   ```
5. **Choose validity** (15 years recommended)
6. **Download certificate** and **private key**

### **Step 4: Import Certificate to AWS ACM**

```bash
# Import the certificate to AWS Certificate Manager
aws acm import-certificate \
  --certificate fileb://certificate.pem \
  --private-key fileb://private-key.pem \
  --region us-east-1
```

**Note the Certificate ARN** - you'll need to update it in the infrastructure code.

### **Step 5: Update Infrastructure Configuration**

Update the certificate ARN in `infrastructure/lib/boat-listing-stack.ts`:

```typescript
certificate = certificatemanager.Certificate.fromCertificateArn(
  this, 'CloudflareOriginCertificate', 
  'arn:aws:acm:us-east-1:YOUR-ACCOUNT:certificate/YOUR-CERT-ID'
);
```

### **Step 6: Deploy Infrastructure**

```bash
cd infrastructure

# For production
npm run deploy:prod

# For development
npm run deploy:dev

# For staging
npm run deploy:staging
```

### **Step 7: Configure DNS Records**

After deployment, add these DNS records in Cloudflare:

#### **Production Environment**
```
Type: CNAME
Name: api
Target: [API Gateway Custom Domain from CDK outputs]
Proxy: ✅ Proxied

Type: CNAME  
Name: @
Target: [Cloudflare Tunnel ID].cfargotunnel.com
Proxy: ✅ Proxied
```

#### **Development Environment**
```
Type: CNAME
Name: api-dev
Target: [API Gateway Custom Domain from CDK outputs]
Proxy: ✅ Proxied

Type: CNAME
Name: dev
Target: [Cloudflare Tunnel ID].cfargotunnel.com
Proxy: ✅ Proxied
```

## 🧪 **Development Without Custom Domains**

For **local development and testing only**, you can deploy without custom domains:

### **When to Use This Approach**

✅ **Local development** on your machine
✅ **Automated testing** in CI/CD pipelines
✅ **Quick prototyping** and experimentation
✅ **Learning and tutorials**

❌ **Never use for production**
❌ **Not suitable for demos**
❌ **Not for staging environments**

### **How to Deploy Without Custom Domains**

```bash
cd infrastructure

# Deploy without custom domains
cdk deploy --context useCustomDomains=false

# Or set environment variable
export USE_CUSTOM_DOMAINS=false
cdk deploy
```

### **Limitations Without Custom Domains**

1. **🔒 Security Risks**
   - No DDoS protection
   - Direct exposure of AWS resources
   - Manual SSL certificate management

2. **🐌 Performance Issues**
   - No global CDN
   - No edge caching
   - Slower load times globally

3. **🔧 Operational Challenges**
   - Limited monitoring and analytics
   - Harder to debug issues
   - No advanced routing capabilities

4. **💻 Development Issues**
   - CORS configuration complexity
   - Different URLs for different environments
   - Frontend routing may not work correctly

### **Frontend Configuration for Development**

When deploying without custom domains, update your frontend environment:

```bash
# frontend/.env.local
VITE_API_URL=https://[random-id].execute-api.us-east-1.amazonaws.com/prod
VITE_ENVIRONMENT=development
```

## 🚨 **Troubleshooting**

### **Common Issues**

#### **"Certificate not found" Error**
```bash
# Check if certificate exists
aws acm list-certificates --region us-east-1

# Import certificate if missing
aws acm import-certificate \
  --certificate fileb://certificate.pem \
  --private-key fileb://private-key.pem \
  --region us-east-1
```

#### **DNS Not Resolving**
```bash
# Check DNS propagation
dig api.harborlist.com
nslookup api.harborlist.com

# Check Cloudflare DNS settings
# Ensure records are "Proxied" (orange cloud)
```

#### **SSL Certificate Errors**
- Verify certificate includes all required domains
- Check certificate expiration date
- Ensure certificate is in the correct AWS region (us-east-1)

#### **CORS Errors**
- Verify API Gateway CORS configuration
- Check Cloudflare security settings
- Ensure frontend is using correct API URL

### **Verification Commands**

```bash
# Test API endpoint
curl -I https://api.harborlist.com/health

# Test frontend
curl -I https://harborlist.com

# Check certificate
openssl s_client -connect api.harborlist.com:443 -servername api.harborlist.com
```

## 📋 **Environment-Specific Configurations**

### **Production**
```
Frontend: harborlist.com
API: api.harborlist.com
Certificate: *.harborlist.com
```

### **Staging**
```
Frontend: staging.harborlist.com  
API: api-staging.harborlist.com
Certificate: *.harborlist.com
```

### **Development**
```
Frontend: dev.harborlist.com
API: api-dev.harborlist.com  
Certificate: *.harborlist.com
```

## 🎯 **Best Practices**

1. **🔒 Always use custom domains for production**
2. **📊 Monitor Cloudflare analytics regularly**
3. **🔄 Set up automatic certificate renewal**
4. **🛡️ Enable Cloudflare security features**
5. **📈 Use staging environment for testing**
6. **🔍 Monitor DNS propagation after changes**
7. **💾 Backup DNS configurations**
8. **🚨 Set up alerting for domain issues**

## 📞 **Support**

If you encounter issues with domain setup:

1. **Check Cloudflare Status**: https://www.cloudflarestatus.com/
2. **Verify AWS Service Health**: https://status.aws.amazon.com/
3. **Review DNS propagation**: https://dnschecker.org/
4. **Test SSL certificates**: https://www.ssllabs.com/ssltest/

---

**💡 Remember**: Custom domains are not just about having a pretty URL—they're essential for security, performance, and operational excellence in production environments.