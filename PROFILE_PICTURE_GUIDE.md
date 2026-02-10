# Profile Picture Storage & Display Guide

## Overview
Both schools and students can have profile pictures stored as **Base64-encoded data** in the database.

---

## SCHOOL PROFILE PICTURE

### Database Column
```
Table: schools
Column: profile_pic (TEXT)
Type: Base64 encoded image or image URL
```

### SQL to Update School Picture
```sql
-- Replace with Base64 image data
UPDATE schools 
SET profile_pic = 'data:image/jpeg;base64,/9j/4AAQSkZJRgABA...'
WHERE id = '550e8400-e29b-41d4-a716-446655440000';
```

### JavaScript Code - Upload & Save
```javascript
// When user selects an image file
document.getElementById('profilePicInput').addEventListener('change', async (e) => {
    const file = e.target.files[0];
    
    // Convert file to Base64
    const reader = new FileReader();
    reader.onload = async (event) => {
        const base64Image = event.target.result;  // Format: data:image/jpeg;base64,xxxxx
        
        // Save to database
        const sb = await getSupabase();
        const { error } = await sb.from('schools')
            .update({ profile_pic: base64Image })
            .eq('id', schoolId);
        
        if (error) {
            console.error('Failed to save picture:', error);
        } else {
            console.log('Picture saved successfully');
        }
    };
    
    // Read file as Base64
    reader.readAsDataURL(file);
});
```

### HTML Display
```html
<!-- Display school profile picture -->
<div class="profile-pic-container">
    <img id="schoolLogo" 
         src="" 
         alt="School Logo" 
         style="width: 150px; height: 150px; border-radius: 50%; object-fit: cover;">
</div>

<script>
// Set the image source from database
async function displaySchoolPicture(schoolId) {
    const sb = await getSupabase();
    const { data } = await sb.from('schools')
        .select('profile_pic')
        .eq('id', schoolId)
        .single();
    
    if (data && data.profile_pic) {
        document.getElementById('schoolLogo').src = data.profile_pic;
    }
}

// Call on page load
displaySchoolPicture('550e8400-e29b-41d4-a716-446655440000');
</script>
```

---

## STUDENT PROFILE PICTURE

### Database Column
```
Table: students
Column: profile_pic (TEXT)
Type: Base64 encoded image or image URL
```

### SQL to Update Student Picture
```sql
-- Replace with Base64 image data
UPDATE students 
SET profile_pic = 'data:image/png;base64,iVBORw0KGgo...'
WHERE id = 'f47ac10b-58cc-4372-a567-0e02b2c3d479';
```

### JavaScript Code - Upload & Save
```javascript
// Upload student photo
async function uploadStudentPhoto(studentId, file) {
    // Convert to Base64
    const reader = new FileReader();
    reader.onload = async (event) => {
        const base64Image = event.target.result;
        
        // Save to database
        const sb = await getSupabase();
        const { error } = await sb.from('students')
            .update({ profile_pic: base64Image })
            .eq('id', studentId);
        
        if (error) {
            console.error('Upload failed:', error);
            showAlert('error', 'Failed to upload photo');
        } else {
            showAlert('success', 'Photo uploaded successfully');
        }
    };
    
    reader.readAsDataURL(file);
}

// Usage when file is selected
document.getElementById('studentPhotoInput').addEventListener('change', (e) => {
    const file = e.target.files[0];
    uploadStudentPhoto(studentId, file);
});
```

### HTML Display
```html
<!-- Display student photo in list -->
<div class="student-card">
    <img class="student-photo" 
         src="" 
         alt="Student Photo" 
         style="width: 80px; height: 80px; border-radius: 50%; object-fit: cover; margin-right: 15px;">
    <div class="student-info">
        <h3 id="studentName">Name</h3>
        <p id="studentGender">Gender</p>
    </div>
</div>

<script>
// Load student data with photo
async function loadStudentData(studentId) {
    const sb = await getSupabase();
    const { data } = await sb.from('students')
        .select('name, gender, age, profile_pic')
        .eq('id', studentId)
        .single();
    
    if (data) {
        document.getElementById('studentName').textContent = data.name;
        document.getElementById('studentGender').textContent = data.gender;
        
        // Set photo if it exists
        if (data.profile_pic) {
            document.querySelector('.student-photo').src = data.profile_pic;
        } else {
            // Show placeholder if no photo
            document.querySelector('.student-photo').src = 'assets/img/placeholder-avatar.png';
        }
    }
}
</script>
```

---

## FILE SIZE CONSIDERATIONS

### Recommended Image Specifications
- **Format**: JPEG or PNG
- **Max File Size**: 500 KB (per image)
- **Recommended Dimensions**: 
  - School logo: 300x300 px (square)
  - Student photo: 200x200 px (square)

### Compress Before Upload
```javascript
// Optional: Compress image before saving
async function compressAndUploadImage(file, targetSize = 500000) {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    const img = new Image();
    img.onload = () => {
        // Resize to 300x300
        canvas.width = 300;
        canvas.height = 300;
        ctx.drawImage(img, 0, 0, 300, 300);
        
        // Convert to Base64
        const base64 = canvas.toDataURL('image/jpeg', 0.8);
        
        // Upload
        uploadImage(base64);
    };
    
    img.src = URL.createObjectURL(file);
}
```

---

## IMAGE FORMAT REFERENCE

### Data URL Format
```
data:[<mediatype>][;base64],<data>
```

### Examples
```
data:image/jpeg;base64,/9j/4AAQSkZJRgABA...
data:image/png;base64,iVBORw0KGgoAAAANS...
data:image/webp;base64,UklGRiYAAABXRUJQ...
```

---

## RETRIEVING IMAGES

### From Database Query
```sql
-- Get school with picture
SELECT id, name, location, profile_pic FROM schools WHERE id = 'xxx';

-- Get student with picture
SELECT id, name, gender, age, profile_pic FROM students WHERE id = 'xxx';
```

### Using Supabase Client
```javascript
// Get school picture
async function getSchoolPicture(schoolId) {
    const sb = await getSupabase();
    const { data } = await sb.from('schools')
        .select('profile_pic')
        .eq('id', schoolId)
        .single();
    
    return data?.profile_pic || null;
}

// Get student picture
async function getStudentPicture(studentId) {
    const sb = await getSupabase();
    const { data } = await sb.from('students')
        .select('profile_pic')
        .eq('id', studentId)
        .single();
    
    return data?.profile_pic || null;
}
```

---

## DELETE/CLEAR IMAGE

### SQL
```sql
-- Clear school picture
UPDATE schools SET profile_pic = NULL WHERE id = 'xxx';

-- Clear student picture
UPDATE students SET profile_pic = NULL WHERE id = 'xxx';
```

### JavaScript
```javascript
async function clearProfilePicture(type, id) {
    const sb = await getSupabase();
    const { error } = await sb.from(type)
        .update({ profile_pic: null })
        .eq('id', id);
    
    if (error) {
        console.error('Failed to clear picture:', error);
    } else {
        console.log('Picture removed');
    }
}

// Usage
clearProfilePicture('schools', schoolId);
clearProfilePicture('students', studentId);
```

---

## TROUBLESHOOTING

### Image not displaying?
1. Check if `profile_pic` field in database is not NULL
2. Verify Base64 data starts with `data:image/`
3. Check browser console for errors
4. Try opening Base64 in new tab: `data:image/jpeg;base64,...`

### Image too large?
1. Compress image before uploading
2. Reduce image dimensions (200x200 or 300x300)
3. Use JPEG format instead of PNG for smaller file size

### Storage limit concerns?
- Average Base64 image: ~200-400 KB
- Supabase PostgreSQL: 500 MB free tier
- Suitable for ~1000-2000 images per project

### Better alternative: Use Supabase Storage
For larger projects, use Supabase Storage bucket instead:
```javascript
// Upload to storage bucket
const { data, error } = await sb.storage
    .from('profile-pictures')
    .upload(`schools/${schoolId}.jpg`, file);

// Get public URL
const { data: { publicUrl } } = sb.storage
    .from('profile-pictures')
    .getPublicUrl(`schools/${schoolId}.jpg`);

// Save URL to database
await sb.from('schools').update({ 
    profile_pic: publicUrl 
}).eq('id', schoolId);
```

