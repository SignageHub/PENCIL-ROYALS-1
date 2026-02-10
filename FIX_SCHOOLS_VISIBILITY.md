# Fix Guide: Schools Not Visible to Other Users

## Problem Summary
Users see schools they registered on their browser, but other users on other devices cannot see those schools. This is caused by:
1. Missing RLS (Row Level Security) policy for public school access
2. Missing domain configuration in Supabase

---

## SOLUTION 1: Fix RLS Policies (Required)

### Step 1: Open Supabase SQL Editor
1. Go to https://supabase.com
2. Sign in to your project
3. Click **SQL Editor** (left sidebar)
4. Click **New Query**

### Step 2: Run the Fix SQL
Copy and paste this SQL code:

```sql
-- Add policy for public to read APPROVED schools
DROP POLICY IF EXISTS "public_read_approved_schools" ON schools;

CREATE POLICY "public_read_approved_schools" ON schools
FOR SELECT USING (approved = true);
```

Then click **Run** (Ctrl+Enter)

**What this does:** Allows anyone (even unauthenticated users) to see schools that have `approved = true`.

### Step 3: Verify It Works
Run this test query:

```sql
-- This should return schools if policy is working
SELECT id, name, location, approved FROM schools WHERE approved = true LIMIT 5;
```

---

## SOLUTION 2: Configure Your Domain (Required for Custom Domain)

⚠️ **IMPORTANT:** If you're using a custom domain like `pencilroyal.com`, you MUST add it to Supabase's allowed domains, otherwise Cross-Origin (CORS) errors will block requests.

### Steps:
1. Go to **Supabase Dashboard** → Your Project
2. Click **Settings** → **API** (left sidebar)
3. Scroll down to **URL Configuration** section
4. Look for **Redirect URLs** or **CORS origins**
5. Add your domain(s):
   - `http://pencilroyal.com`
   - `https://pencilroyal.com`
   - `http://www.pencilroyal.com`
   - `https://www.pencilroyal.com`

6. Click **Save**

### If Still Getting CORS Errors:
Also add these in **Settings** → **API** → **API Settings**:
- Under **Allow CORS** or **CORS Configuration**
- Add your full domain URL

---

## SOLUTION 3: Verify Code is Correct (Already Fixed)

Your `schools.html` is correctly set up to:
- Query the `schools` table from Supabase (not localStorage)
- Filter by `approved = true`
- Display results to all visitors

✅ No code changes needed here.

---

## Testing Steps

### Test 1: Single Browser - Different Tabs
1. Open `schools.html` in Tab 1
2. Register a new school and check if it appears
3. Open an **Incognito/Private Tab** and go to `schools.html`
4. **Expected:** You should see the newly registered school (if approved)

### Test 2: Different Devices/Users
1. Device 1: Register a school
2. Device 2: Go to `schools.html`
3. **Expected:** School from Device 1 should be visible

### Test 3: Check Browser Console for Errors
1. Open `schools.html`
2. Press **F12** (Developer Tools)
3. Go to **Console** tab
4. Look for any red error messages
5. If you see CORS errors → You need to add domain to Supabase
6. If you see permission errors → RLS policy wasn't applied correctly

---

## Common Issues & Fixes

### Issue: "No Schools Yet" message appears
**Cause:** Either no schools exist, or RLS policy isn't working
**Fix:** 
1. Verify RLS policy was created (run the test query above)
2. Check if any schools exist in database
3. Check if schools have `approved = true`

**Debug Query:**
```sql
-- See ALL schools in database (ignoring RLS)
SELECT * FROM schools;
```

### Issue: CORS Error in Console
**Cause:** Domain not added to Supabase
**Fix:** Add your domain to Supabase settings (Step 5 above)

### Issue: Still can't see schools on other devices
**Cause:** Schools might not be marked as approved
**Fix:** 
```sql
-- Mark all schools as approved (for testing)
UPDATE schools SET approved = true;
```

---

## Summary Checklist

- [ ] Run the FIX_PUBLIC_SCHOOLS_ACCESS.sql in Supabase
- [ ] Test with incognito tab (should see approved schools)
- [ ] Add your domain to Supabase allowed domains
- [ ] Clear browser cache (Ctrl+Shift+Delete)
- [ ] Test on another device
- [ ] Check browser console for errors

---

## Need Help?

If schools still don't appear:
1. Check Supabase **Console** → **Logs** for database errors
2. Verify `schools` table has data
3. Verify `approved` column exists and has `true` values
4. Check RLS policies are enabled on schools table
