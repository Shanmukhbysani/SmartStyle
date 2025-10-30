# 🔐 Security & Environment Variables Guide

## 🔑 Understanding NEXT_PUBLIC_* Variables

### **Are They Secret?**
**NO!** `NEXT_PUBLIC_*` variables are **intentionally public**:



**Why is this safe?**
1. Firebase config is **designed to be public**
2. Security is enforced by **Firebase Security Rules** (firestore.rules, storage.rules)
3. Only authenticated users can access protected data
4. Rate limiting and abuse protection in Firebase Console

### **What IS Secret?**
These should NEVER be committed or exposed:
```bash
FIREBASE_SERVICE_ACCOUNT_KEY  # Server-side admin credentials
GOOGLE_GENAI_API_KEY          # Not prefixed with NEXT_PUBLIC_
GROQ_API_KEY                  # Not prefixed with NEXT_PUBLIC_
```

---

## 📋 Best Practices

### **1. Local Development**
```bash
# Copy the template
cp .env.template .env.local

# Fill in your actual values
# Edit .env.local with your real API keys
```

### **2. Git Repository**
```bash
# Before committing, verify .env is not staged
git status

# If .env appears, it's a problem! Check .gitignore
```

### **3. Production Deployment**

#### **Vercel:**
1. Go to Project Settings → Environment Variables
2. Add each `NEXT_PUBLIC_*` variable
3. Deploy

#### **Firebase Hosting:**
```bash
# Set environment variables
firebase functions:config:set \
  firebase.api_key="YOUR_KEY" \
  firebase.project_id="YOUR_PROJECT"

# Deploy
firebase deploy
```

#### **Other Platforms:**
- Add environment variables in platform dashboard
- Ensure `NEXT_PUBLIC_*` prefix for client-side vars

---

## 🛡️ Firebase Security

### **Security is Enforced By:**

1. **Firestore Rules** (`firestore.rules`):
```javascript
// Users can only read/write their own data
match /users/{userId} {
  allow read, write: if request.auth.uid == userId;
}
```

2. **Storage Rules** (`storage.rules`):
```javascript
// Only authenticated users can upload
// Max 20MB file size
match /users/{userId}/{allPaths=**} {
  allow write: if request.auth.uid == userId 
               && request.resource.size < 20 * 1024 * 1024;
}
```

3. **Firebase Console:**
- App Check (prevents abuse)
- Rate limiting
- Budget alerts
- Usage monitoring

---

## ⚠️ What to NEVER Commit

### **❌ NEVER commit these files:**
```
.env
.env.local
.env.*.local
firebase-service-account*.json
*-private-key.json
.firebase/ (cache directory)
```

### **❌ NEVER hardcode in source code:**


---

## 🔍 Security Audit Checklist

Run this before every commit:

```bash
# 1. Check what's being committed
git status
git diff --cached

# 2. Search for potential secrets
grep -r "AIzaSy" .env .env.local .env.example

# 3. Verify .gitignore is working
git check-ignore .env .env.local

# 4. Check Next.js config
grep -i "AIzaSy" next.config.js  # Should return nothing!
```

---

## 🚨 If You Accidentally Committed Secrets

### **Immediate Actions:**

1. **Rotate all exposed credentials immediately**
   - Firebase: Generate new API keys
   - Google AI: Create new API keys  
   - Delete old keys

2. **Remove from Git history:**
```bash
# Use git-filter-branch or BFG Repo Cleaner
# Then force push (if repository is private)
```

3. **Update Firebase Security Rules**

4. **Enable App Check in Firebase**

---

## 📚 Additional Resources

- [Next.js Environment Variables](https://nextjs.org/docs/basic-features/environment-variables)
- [Firebase Security Rules](https://firebase.google.com/docs/rules)
- [Firebase App Check](https://firebase.google.com/docs/app-check)
- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)

---

## ✅ Summary

**Your current setup is SECURE because:**
1. ✅ `.env` files are in `.gitignore`
2. ✅ `next.config.js` has NO hardcoded secrets
3. ✅ Firebase Security Rules are configured
4. ✅ `NEXT_PUBLIC_*` vars are safe to expose (by design)
5. ✅ Server-side secrets are never committed

**Remember:** Firebase client config is **meant to be public**. Security comes from Firebase Security Rules, not hiding the config.
