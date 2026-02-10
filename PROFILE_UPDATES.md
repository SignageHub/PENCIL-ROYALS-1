# Profile Page Updates - Pencil Royal

## Overview
The profile.html has been updated to support user-specific profiles with proper access control and improved student management features.

## Key Changes

### 1. **User-Specific URL Routing**
- **Old:** `profile.html` (showed current user's profile)
- **New:** Supports URL parameters to view specific user profiles
  - Query parameter: `profile.html?user=[userid]`
  - Example: `profile.html?user=550e8400-e29b-41d4-a716-446655440000`

### 2. **Registered Students Display**
- ✅ All previously registered students are now displayed in the "Students" tab
- Shows student count by gender (boys/girls)
- Displays student details including name, gender, and age
- Students are listed with edit/delete options (own profile only)

### 3. **Profile Access Control**
- **Own Profile:** Users can edit school information and manage students
- **Other Profiles:** Users can view school info and students but cannot edit
- Visual banner displays when viewing someone else's profile
- Edit/delete buttons are hidden for other users' profiles

### 4. **School Profile Features**
The profile page includes four main sections:

#### Tab 1: Overview
- School statistics (total students, boys count, girls count)
- National competition finalists list

#### Tab 2: Profile Settings
- Edit school name, location, contact person, phone
- Only available when viewing own profile
- Updates save to database with proper validation

#### Tab 3: Students
- Add new students (max 5 boys + 5 girls per school)
- View all registered students
- Edit student information
- Delete students
- Gender split display (👦 Boys | 👧 Girls)

#### Tab 4: Competition Status
- Link to internal competitions page
- Competition status tracking

### 5. **Updated Schools Listing**
The schools.html page now displays available schools with clickable profiles:
- Each school links to its public profile using the user-specific URL
- Shows school name and location
- Hover effects for better UX
- Sorted alphabetically

## How to Use

### View Your Own Profile
1. Login to your account
2. Click "My School" in navigation
3. You'll see your full editable profile

### View Another School's Profile
In schools.html:
1. Go to "Schools" page
2. Click on any school name to view their public profile
3. You can see their students but cannot edit them

Or directly via URL:
- `profile.html?user=[school-user-id]`

### Edit Your Profile
1. Go to your profile (click "My School")
2. Click "Profile Settings" tab
3. Update school information
4. Click "Save Profile"

### Manage Students
1. Go to your profile
2. Click "Students" tab
3. **Add Student:** Fill in the form and click "Add Student"
4. **Edit Student:** Click "Edit" button on the student
5. **Delete Student:** Click "Delete" button on the student

## Technical Details

### Authorization
- All operations check if user is viewing their own profile
- Edit operations only work on own profile
- Database RLS policies enforce server-side security

### Student Limits
- Maximum 5 boys per school
- Maximum 5 girls per school
- Age range: 5-30 years

### Data Validation
- School name is required
- Student name and gender are required
- Age is optional
- All data is validated before sending to database

## Files Modified
1. **profile.html** - Complete rewrite with user-specific routing
2. **assets/js/schools-list.js** - Updated to show clickable school profiles with user IDs

## Error Handling
- Clear error messages for unauthorized actions
- Loading indicators during data fetching
- Success notifications for completed actions
- Fallback messages when no data available

## Notes
- The page maintains backward compatibility with query parameters
- Profile data loads dynamically from Supabase
- All student registrations are preserved
- RLS policies ensure data security

