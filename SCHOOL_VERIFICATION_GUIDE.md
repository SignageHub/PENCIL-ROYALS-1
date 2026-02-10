# School Verification System - Documentation

## Overview
The School Verification System is a payment-based verification process that confirms school registrations before they can access the platform. After signup, schools must submit a transaction ID proving payment before gaining access to the dashboard.

---

## System Flow

### 1. **Registration Phase**
- User registers with school name, email, and password on `signup.html`
- School profile setup on `setup-profile.html` (profile picture, location, students)
- After setup completion, user is redirected to **verification page**

### 2. **Verification Phase**
- **Page**: `school-verification.html`
- School sees admin phone number and instructions
- School sends payment to admin's phone number
- School copies transaction ID from payment receipt
- School submits transaction ID in the form
- **Status**: Enters "pending approval" state

### 3. **Admin Approval Phase**
- **Page**: `admin-school-verification.html`
- Admin views all pending & previous verifications
- Admin can:
  - ✅ **Approve**: School is verified, gains access
  - ❌ **Reject**: Send rejection reason, school stays pending
  - **View**: Transaction ID, dates, school details

### 4. **Access Phase**
- ✅ **After Approval**: School can login and access `profile.html` dashboard
- ⏳ **After Rejection**: School can resubmit or contact admin
- 🔒 **Before Verification**: Attempting to access dashboard redirects to verification page

---

## Database Schema

### `school_verification` Table
```sql
CREATE TABLE school_verification (
    id UUID PRIMARY KEY,
    school_id UUID NOT NULL (foreign key to schools)
    user_id UUID NOT NULL (foreign key to auth.users)
    transaction_id TEXT - Transaction ID from payment
    verification_status TEXT - pending/approved/rejected
    admin_notes TEXT - Admin reason for approval/rejection
    verified_at TIMESTAMP - When approved
    created_at TIMESTAMP - When created
    updated_at TIMESTAMP - Last update
);
```

### Status Values
- **pending**: Waiting for admin approval
- **approved**: School is verified and can access platform
- **rejected**: Admin rejected the verification (can resubmit)

---

## File Locations

### Frontend Pages
- **Verification Page**: `/pencil-royal/school-verification.html`
- **Admin Verification Panel**: `/pencil-royal/admin-school-verification.html`

### Backend SQL
- **Database Setup**: `/pencil-royal/supabase/PERMISSIONS_AND_RLS.sql`
- **RLS Policies**: Lines 262-293 (school_verification policies)

### JavaScript Functions
- **Auth Functions**: `/pencil-royal/assets/js/auth.js`
  - `isSchoolVerified(userId)` - Check if school is verified
  - `getSchoolVerificationStatus(userId)` - Get verification details
  - `protectSchoolPage()` - Redirect unverified schools

---

## Configuration

### Admin Phone Number
**File**: `school-verification.html`  
**Line**: Line ~220 in JavaScript section

```javascript
const ADMIN_PHONE = "+1234567890"; // Change this to your admin phone number
```

**Update this to your actual admin phone number before deployment!**

---

## User Journey Examples

### Example 1: Fresh Registration → Approval
```
1. User clicks "Register" → signup.html
2. Fills school info & creates account
3. Completes profile setup (picture, students)
4. ✓ Redirected to school-verification.html
5. Sees admin phone number: +251-9-XX-XX-XXXX
6. Sends money to that number
7. Gets transaction ID: TRN-20260209-12345
8. Enters transaction ID and submits
9. ⏳ Waiting message shown, auto-checks every 5 seconds
10. Admin reviews on admin-school-verification.html
11. Admin clicks "Approve" ✅
12. School gets notification and redirected to dashboard ✓
13. Can now use the platform
```

### Example 2: Rejection & Resubmission
```
1. School submits transaction ID → pending
2. Admin sees it, enters rejection reason: "Invalid transaction ID"
3. Admin clicks "Reject"
4. School sees rejection message with reason
5. School can resubmit with new transaction ID
6. Cycle repeats until approved
```

### Example 3: Login Before Verification
```
1. School registers and sets up profile
2. Closes browser before submitting verification
3. Logs in via login.html
4. System checks: "Is this school verified?"
5. No → Redirected to school-verification.html
6. Can continue verification process
```

---

## Key Features

### ✅ Auto-Verification Check
- Verification page auto-checks for approval every 5 seconds
- No manual refresh needed
- User sees live updates

### 🔒 Security
- RLS policies ensure schools can only view their own verification
- Admins can only approve verifications
- Database-level enforcement via Supabase RLS

### 📱 Mobile Friendly
- Responsive design works on all devices
- Clear instructions for payment submission
- Easy transaction ID input

### 🎯 Admin Controls
- View all pending verifications
- Filter by status (pending, approved, rejected)
- Add notes/comments when approving/rejecting
- See school details, email, phone, submission date
- Bulk view of all verifications

---

## Integration Points

### 1. **Signup Flow** (`setup-profile.html`)
- Line ~523: Redirects to `school-verification.html` after profile setup

### 2. **Login Flow** (`auth.js`)
- `login()` function checks verification status
- Redirects unverified schools to verification page

### 3. **Dashboard Protection** (`profile.html`)
- `protectSchoolPage()` called at initialization
- Redirects unverified schools away

### 4. **Admin Panel** (`admin-panel.html`)
- Navigation link to verification panel added
- Easy access for admins

---

## Verification Status Checks

### Current User's School
```javascript
// Check if school is verified
const isVerified = await isSchoolVerified(currentUserID);
// Returns: true or false

// Get full verification record
const status = await getSchoolVerificationStatus(currentUserID);
// Returns: { id, school_id, transaction_id, verification_status, admin_notes, verified_at, ... }
```

---

## Deployment Checklist

Before going live:

- [ ] Update `ADMIN_PHONE` in `school-verification.html` (line ~220)
- [ ] Run `PERMISSIONS_AND_RLS.sql` in Supabase
- [ ] Create admin role for your admin user (SQL command in PERMISSIONS_AND_RLS.sql)
- [ ] Test registration flow: signup → profile setup → verification
- [ ] Test admin panel: view pending → approve → verify access works
- [ ] Test rejection flow: reject → see in verification page
- [ ] Test login redirect: unverified school → verification page
- [ ] Test approved school: can access dashboard ✓

---

## Troubleshooting

### Problem: "Transaction ID is empty"
- **Solution**: Make sure the input field has focus and user typed the ID correctly

### Problem: Verification page not loading
- **Solution**: Check browser console for errors, ensure Supabase connection works

### Problem: Admin can't see verifications
- **Solution**: Ensure admin user has 'admin' role in `user_roles` table

### Problem: School can't see status updates
- **Solution**: Verification page auto-checks every 5 seconds; check if admin actually approved it

### Problem: School redirects to verification after approval
- **Solution**: Page needs to be refreshed or auto-redirects after 2 seconds; wait for redirect

---

## Future Enhancements

Potential improvements:
- Email notifications when verification is approved/rejected
- SMS notifications for status updates
- Payment integration (automatic verification on payment success)
- Verification deadline reminders
- Bulk verification import for pre-approved schools
- Verification fee management dashboard

---

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review browser console for error messages
3. Verify RLS policies are correctly applied in Supabase
4. Check that admin user has correct role in database

---

**Last Updated**: February 9, 2026  
**System Version**: 1.0
