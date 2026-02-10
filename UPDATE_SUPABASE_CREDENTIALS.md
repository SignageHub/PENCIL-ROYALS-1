# Update Your Supabase Credentials

## Where to Find Your Real Supabase Credentials

Follow these steps to get your actual Supabase project credentials:

### 1. **Go to Supabase Dashboard**
- Open: https://app.supabase.com
- Login with your account

### 2. **Select Your Project**
- Click on your project name

### 3. **Get Project URL & API Key**
- Click on **Settings** (bottom left)
- Click on **API**
- You'll see:
  - **Project URL** (looks like: `https://xxx.supabase.co`)
  - **public Anon key** (long string starting with `eyJ...`)

### 4. **Copy Your Credentials**

Your screen should show something like:

```
Project URL: https://xxxxxxxxxxx.supabase.co
public anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Update Your Project File

### **File to Update:**
`pencil-royal/assets/js/supabase-client.js`

### **Find Lines 1-2:**
```javascript
const SUPABASE_URL = 'https://tjooofnjwwtgageayezr.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

### **Replace with YOUR credentials:**
```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_URL_HERE';
const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY_HERE';
```

---

## Example:

**Before (Current Demo):**
```javascript
const SUPABASE_URL = 'https://tjooofnjwwtgageayezr.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InRqb29vZm5qd3d0Z2FnZWF5ZXpyIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0NTI1NDIsImV4cCI6MjA4NjAyODU0Mn0.Pg8ldP8qI6e70WNGNzdnAbMmgAlL1rb6w41sGjXFc9Y';
```

**After (Your Real Credentials):**
```javascript
const SUPABASE_URL = 'https://abcdef123456.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyNEWJHSDaksjdakDJDAkjdakljdakldjaldo...';
```

---

## After Updating:

1. **Save the file** (Ctrl+S)
2. **Hard refresh your browser** (Ctrl+Shift+Delete then clear cache, or Ctrl+F5)
3. **Reload the page**
4. **Open F12 Console** to confirm no errors

---

## Verify It Works:

In the browser console (F12), you should see:
```
✓ Supabase initialized successfully
```

No more `"Identifier 'supabase' has already been declared"` error!

