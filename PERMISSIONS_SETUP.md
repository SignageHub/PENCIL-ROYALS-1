# SETUP GUIDE: Role-Based Permissions & Admin Access

## Overview
This guide explains how to set up the new role-based permission system for your Pencil Royals application.

## What Changed
- **Old System**: Hardcoded admin email addresses in JavaScript files (INSECURE)
- **New System**: Role-based access control with Supabase Row Level Security (RLS) policies (SECURE)

## Step 1: Run the Permissions SQL Script

1. Go to your Supabase Dashboard
2. Navigate to **SQL Editor**
3. Click **New Query**
4. Copy ALL the SQL code from `supabase/PERMISSIONS_AND_RLS.sql`
5. Paste it into the SQL Editor
6. Click **Run**

This will:
- Create a `user_roles` table to store user roles
- Create helper functions for permission checking
- Enable RLS (Row Level Security) on all tables
- Create policies for admin and school access

## Step 2: Create Admin Users

After running the SQL script, you need to create admin roles for your admin users.

### Option A: Set Admin Role for Existing Users

1. In Supabase, go to **SQL Editor** → **New Query**
2. For each admin email, run:

```sql
INSERT INTO user_roles (user_id, role)
SELECT id, 'admin'
FROM auth.users
WHERE email = 'admin@pencilroyal.com'
ON CONFLICT (user_id) DO NOTHING;
```

Replace `'admin@pencilroyal.com'` with your actual admin email.

**Example for multiple admins:**
```sql
INSERT INTO user_roles (user_id, role)
SELECT id, 'admin'
FROM auth.users
WHERE email IN ('admin@pencilroyal.com', 'ss@gmail.com', 'admin@test.com')
ON CONFLICT (user_id) DO NOTHING;
```

### Option B: Add Admin Role When Creating New User

Ask your admin users to:
1. Sign up at your application
2. Then you run the query above for their email

## Step 3: Test the System

### Test Admin Access
1. Login as an admin user
2. You should be redirected to `/admin-panel.html`
3. Try creating/editing competitions and updating marks

### Test School Access
1. Login as a school user
2. You should be redirected to `/profile.html`
3. Schools can:
   - Add/edit/delete their own students ✓
   - View their own students ✓
4. Schools CANNOT:
   - Edit other schools' students ✗
   - Update national finals ✗
   - Access admin panel ✗

### Test Admin Permissions
- Admins can:
  - ✓ View all schools
  - ✓ Update marks/scores via the admin panel
  - ✓ Select finalists for national competitions
  - ✓ Manage all students across schools
  - ✓ Create and manage competitions

## Step 4: Set School Role for Users (Optional)

If you want to assign school-specific roles (not just admin):

```sql
UPDATE user_roles
SET role = 'school', school_id = (SELECT id FROM schools WHERE user_id = $1)
WHERE user_id = $1;
```

Or insert:
```sql
INSERT INTO user_roles (user_id, role, school_id)
SELECT id, 'school', (SELECT id FROM schools WHERE user_id = id)
FROM auth.users
WHERE email = 'school@example.com'
ON CONFLICT (user_id) DO NOTHING;
```

## Troubleshooting

### "Access Denied" when trying to access admin panel
- **Check**: Is your user role set to 'admin'? Run:
  ```sql
  SELECT ur.user_id, ur.role, au.email
  FROM user_roles ur
  LEFT JOIN auth.users au ON ur.user_id = au.id
  WHERE au.email = 'YOUR_EMAIL';
  ```
- **Fix**: If no result, add the admin role using the commands in Step 2

### Schools can't add/edit/delete students
- **Check**: Is their role set to 'school'? Run:
  ```sql
  SELECT ur.user_id, ur.role, ur.school_id, au.email, s.name
  FROM user_roles ur
  LEFT JOIN auth.users au ON ur.user_id = au.id
  LEFT JOIN schools s ON ur.school_id = s.id
  WHERE au.email = 'SCHOOL_EMAIL';
  ```
- **Fix**: Update their role to 'school' and make sure `school_id` is set

### "No permission" errors when updating marks
- **Check**: Verify you're logged in as an admin
- **Check**: Row Level Security (RLS) policies are enabled
- Run this to verify:
  ```sql
  SELECT tablename, rowsecurity FROM pg_tables 
  WHERE schemaname = 'public' AND tablename IN ('students', 'competitions', 'competition_scores');
  ```
  Should show `rowsecurity | true` for all tables

## Key Functions Added

The system provides these helper functions in auth.js:

```javascript
// Get user's role from database
await getUserRole(userId)  // Returns: 'admin', 'school', or 'user'

// Check if user is admin
await isUserAdmin(userId)  // Returns: true/false

// Check if user is a school
await isUserSchool(userId)  // Returns: true/false

// Get user's school ID
await getUserSchoolId(userId)  // Returns: school UUID or null

// Protect admin pages
await protectAdminPage()  // Redirects non-admins away
```

## Database Functions

The system creates these SQL functions:

```sql
get_user_role(user_id)          -- Get user's role
get_user_school(user_id)        -- Get user's school
is_admin(user_id)               -- Check if admin
owns_school(user_id, school_id) -- Check if owns school
```

These are used by Row Level Security policies to enforce permissions at the database level.

## Security Notes

✓ **More Secure**: Permissions are enforced at the DATABASE level via RLS, not just in JavaScript
✓ **Scalable**: Easy to add/remove admins without code changes
✓ **Verifiable**: Check permissions in database, not hidden in code
✓ **Flexible**: Easy to add more roles (volunteer, moderator, etc.) later

## File Changes

These files were updated:
- `assets/js/auth.js` - Added role-checking functions
- `admin-panel.html` - Uses database roles instead of hardcoded emails
- `admin-intercompitions.html` - Uses database roles for permission check
- `admin-school-details.html` - Uses database roles for permission check

New files:
- `supabase/PERMISSIONS_AND_RLS.sql` - All RLS policies and helper functions
