# Troubleshooting Loading Issues - Verification System

## Quick Diagnosis Steps

When pages are stuck on "loading", follow these steps:

### 1. **Check Browser Console for Errors** (CRITICAL!)
1. Open the page that's loading (admin panel, verification page, etc.)
2. Press **F12** to open Developer Tools
3. Go to the **Console** tab
4. Look for RED ERROR messages
5. Share these errors with debugging info below

### 2. **Look for Console Logs**
The fixed code now has console logs starting with emojis:
- 🔄 = Starting process
- ✅ = Success
- ❌ = Error
- ⚠️ = Warning
- 📍 = Redirection
- 📋 = Loading data

**Example console output should look like:**
```
🔄 Initializing verification page...
👤 Getting current user...
✅ User authenticated: school@example.com
🏫 Loading school info...
✅ School loaded: School Name
📋 Loading verification status...
✅ Verification page ready!
```

---

## Common Issues & Solutions

### **Issue 1: "Loading..." stuck forever on school-verification.html**

**Likely causes:**
- Supabase not loaded
- User not authenticated  
- School not found in database
- RLS policies blocking queries

**Debug steps:**
1. Open **F12 Console**
2. Look for errors starting with ❌
3. Check if you see:
   - 🔄 Initializing verification page...
   - 👤 Getting current user... ✅
   - 🏫 Loading school info... (stuck here = problem)

**If stuck at "Loading school info":**
- Make sure you completed signup first (created school in database)
- Make sure `PERMISSIONS_AND_RLS.sql` was run in Supabase
- Check that RLS policies are applied

**Fix:**
```javascript
// In browser console, run:
console.log('Supabase loaded:', !!window.supabase);
```

---

### **Issue 2: Admin panel redirects to login page after waiting**

**Likely causes:**
- `protectAdminPage()` is failing
- User doesn't have 'admin' role in `user_roles` table
- `getCurrentUser()` returns null

**Debug steps:**
1. Open **F12 Console**
2. Look for:
   - 🔐 Protecting admin page...
   - User: [email] ✅
   - Is admin: ❌ (means NOT admin)

**If "Is admin: false":**
1. Go to **Supabase Dashboard** → **SQL Editor**
2. Run this query:
   ```sql
   SELECT ur.user_id, ur.role, au.email, ur.id
   FROM user_roles ur
   LEFT JOIN auth.users au ON ur.user_id = au.id
   WHERE au.email = 'your-admin-email';
   ```
3. Check if:
   - ✅ Row exists (admin is in the table)
   - ✅ Role is 'admin' (not 'school' or 'user')

**If not found:**
- Admin user needs to be manually added to `user_roles` table with role='admin'
- Use Supabase console to add:
   ```sql
   INSERT INTO user_roles (user_id, role)
   SELECT id, 'admin'
   FROM auth.users
   WHERE email = 'your-admin-email@example.com'
   ON CONFLICT DO NOTHING;
   ```

---

### **Issue 3: School verification page loads but shows error message**

**Possible errors:**
- "School not found"
- "Error: Failed to load"

**Debug steps:**
1. Check **F12 Console** for ❌ errors
2. Look for which step failed:
   - Authentication OK? (👤 ✅ shows email)
   - School found? (🏫 ✅ shows school name)

**If "School not found":**
- The school record doesn't exist in `schools` table
- **Solution**: Complete signup and setup-profile again

**If database query error:**
- RLS policies might be blocking access
- Run in Supabase SQL Editor:
   ```sql
   -- Check if verification table exists and is accessible
   SELECT * FROM school_verification LIMIT 5;
   ```
- If error: RLS policies are too strict

---

### **Issue 4: Admin verification panel loads but shows "No Verifications"**

**Likely causes:**
- Filter is set to wrong status
- No verifications exist yet
- Query isn't fetching properly

**Debug steps:**
1. Check **F12 Console**:
   - Should see: "✅ Loaded verifications: [number]"
   - If showing 0: no records exist
2. Check **filter dropdown** - make sure it's set to "All" or "Pending"
3. Run in Supabase SQL Editor:
   ```sql
   SELECT COUNT(*) as total FROM school_verification;
   SELECT * FROM school_verification LIMIT 10;
   ```

---

## Complete Debugging Checklist

Before reporting issues, verify:

- [ ] **Supabase connected?**
  ```javascript
  // In console:
  window.supabase ? 'YES' : 'NO'
  ```

- [ ] **User authenticated?**
  ```javascript
  // In console:
  window.supabase.auth.getUser().then(r => console.log(r.data.user?.email))
  ```

- [ ] **Admin role exists?**
  - Go to Supabase → user_roles table
  - Check if your email has role='admin'

- [ ] **School record exists?**
  - Go to Supabase → schools table
  - Check if school is there with your user_id

- [ ] **Verification table exists?**
  - Go to Supabase → school_verification table
  - Check if structure matches SQL file

- [ ] **RLS policies applied?**
  - Go to Supabase → Authentication → Policies
  - Check all tables have policies listed

---

## Console Log Reference

### School Verification Page (school-verification.html)
```
🔄 Initializing verification page...
👤 Getting current user...
   [shows user email or error]
✅ User authenticated: school@example.com
🏫 Loading school info...
   [shows school name or error]
✅ School loaded: My School Name
📋 Loading verification status...
✅ Verification page ready!
```

### Admin Verification Panel (admin-school-verification.html)
```
🔄 Initializing admin verification panel...
🔐 Checking admin access...
✅ Admin access confirmed
✅ User authenticated: admin@example.com
📋 Loading verifications...
✅ Loaded verifications: 5
```

### Login Flow with Verification Check
```
[User logs in]
✅ Admin detected, redirecting to admin panel
OR
✅ School verified, redirecting to dashboard
OR
📍 School not verified, redirecting to verification page
OR
⏳ School in pending approval, staying on verification page
```

---

## Step-by-Step Fix When Stuck

### For School Verification Page:
1. Clear browser cache (Ctrl+Shift+Delete)
2. Hard refresh page (Ctrl+Shift+R)
3. Open F12 Console
4. Look for first ❌ error
5. Go to that issue section above
6. Follow the fix

### For Admin Panel:
1. Hard refresh (Ctrl+Shift+R)
2. Open F12 Console
3. Check if you see "Is admin: false"
4. If yes, run the SQL INSERT command in Supabase
5. Refresh page again
6. Should now work

### For General Problems:
1. Copy the FULL console output (with all red errors)
2. Run all queries in Supabase SQL Editor to verify:
   - user_roles table has your user with right role
   - schools table has your school
   - school_verification table exists
3. Ensure PERMISSIONS_AND_RLS.sql was fully executed

---

## Database Verification Queries

Run these in **Supabase SQL Editor** to verify everything:

```sql
-- 1. Check if user_roles table exists and has data
SELECT COUNT(*) FROM user_roles;
SELECT * FROM user_roles LIMIT 5;

-- 2. Check if school_verification table exists
SELECT COUNT(*) FROM school_verification;
SELECT * FROM school_verification LIMIT 5;

-- 3. Check RLS policies are enabled
SELECT tablename, ((tablename='schools'::regclass)::int + (tablename='students'::regclass)::int) FROM pg_tables WHERE schemaname='public';

-- 4. Check a specific user's roles (replace email)
SELECT ur.*, au.email, s.name as school_name
FROM user_roles ur
LEFT JOIN auth.users au ON ur.user_id = au.id
LEFT JOIN schools s ON ur.school_id = s.id
WHERE au.email = 'your-email@example.com';

-- 5. Check verifications for a school (replace school_id)
SELECT * FROM school_verification WHERE school_id = 'your-school-uuid';
```

---

## Still Stuck?

Share in console output (F12):
1. All ❌ errors shown
2. The emoji sequence of logs
3. Result of queries above
4. What page you're on
5. Whether this is a NEW school or existing

This will help pinpoint the exact issue!
