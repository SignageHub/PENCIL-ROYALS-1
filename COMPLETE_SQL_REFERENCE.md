# PENCIL ROYAL - Complete SQL Reference

## TABLE OF CONTENTS
1. [Tables & Schema](#tables--schema)
2. [Functions](#functions)
3. [Row Level Security Policies](#row-level-security-policies)
4. [Indexes](#indexes)
5. [Profile Picture Handling](#profile-picture-handling)

---

## TABLES & SCHEMA

### 1. SCHOOLS TABLE
Stores school information linked to authentication users.

```sql
CREATE TABLE schools (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE REFERENCES auth.users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    location TEXT,
    profile_pic TEXT,                    -- Base64 encoded image or URL
    contact_person TEXT,
    phone TEXT,
    approved BOOLEAN DEFAULT false,      -- Admin approval status
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT valid_school_name CHECK (name != '')
);
```

**Columns:**
- `id` - Unique identifier (UUID)
- `user_id` - Reference to authenticated user (one-to-one)
- `name` - School name (required)
- `location` - District/Location
- `profile_pic` - School logo/picture (Base64 or URL)
- `contact_person` - Name of contact
- `phone` - School phone number
- `approved` - Admin approval flag

---

### 2. STUDENTS TABLE
Stores student records for each school (max 5 boys + 5 girls per school).

```sql
CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    gender TEXT NOT NULL CHECK (gender IN ('male', 'female')),
    age INTEGER CHECK (age >= 5 AND age <= 30),
    profile_pic TEXT,                    -- Student photo (Base64 or URL)
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT valid_student_name CHECK (name != '')
);
```

**Columns:**
- `id` - Unique identifier
- `school_id` - Reference to school
- `name` - Student name (required)
- `gender` - 'male' or 'female' (required)
- `age` - Student age (5-30 years)
- `profile_pic` - Student photo (Base64 or URL)

---

### 3. COMPETITIONS TABLE
Stores internal and national competitions.

```sql
CREATE TABLE competitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    school_id UUID REFERENCES schools(id) ON DELETE CASCADE,
    type TEXT CHECK (type IN ('internal', 'national')) DEFAULT 'internal',
    start_date TIMESTAMP WITH TIME ZONE,
    end_date TIMESTAMP WITH TIME ZONE,
    status TEXT CHECK (status IN ('upcoming', 'active', 'completed')) DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Columns:**
- `id` - Unique identifier
- `name` - Competition name
- `school_id` - School hosting internal competition (NULL for national)
- `type` - 'internal' or 'national'
- `start_date` - Competition start date
- `end_date` - Competition end date
- `status` - 'upcoming', 'active', or 'completed'

---

### 4. COMPETITION_SCORES TABLE
Stores individual scores for students in competitions.

```sql
CREATE TABLE competition_scores (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    score INTEGER CHECK (score >= 0 AND score <= 100),
    gender TEXT NOT NULL CHECK (gender IN ('male', 'female')),
    rank INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Columns:**
- `id` - Unique identifier
- `competition_id` - Reference to competition
- `student_id` - Reference to student
- `school_id` - Reference to school
- `score` - Student score (0-100)
- `gender` - Student gender
- `rank` - Student rank in competition

---

### 5. FINALISTS TABLE
Stores selected finalists (top boy + top girl per school) for national competitions.

```sql
CREATE TABLE finalists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    gender TEXT NOT NULL CHECK (gender IN ('male', 'female')),
    score INTEGER,
    status TEXT CHECK (status IN ('qualified', 'competing', 'finished')) DEFAULT 'qualified',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Columns:**
- `id` - Unique identifier
- `competition_id` - Reference to national competition
- `student_id` - Reference to finalist student
- `school_id` - Reference to school
- `gender` - 'male' or 'female'
- `score` - Final score
- `status` - 'qualified', 'competing', or 'finished'

---

### 6. RESULTS TABLE
Stores final competition results and rankings.

```sql
CREATE TABLE results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    competition_id UUID REFERENCES competitions(id) ON DELETE CASCADE,
    rank INTEGER,
    score INTEGER,
    total_students INTEGER,
    boys_count INTEGER,
    girls_count INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Columns:**
- `id` - Unique identifier
- `school_id` - Reference to school
- `competition_id` - Reference to competition
- `rank` - Final rank of school
- `score` - Total score
- `total_students` - Total students participated
- `boys_count` - Number of boys
- `girls_count` - Number of girls

---

### 7. VOTES TABLE
Stores votes for students (if needed for voting competitions).

```sql
CREATE TABLE votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    voter_id UUID,
    competition_id UUID REFERENCES competitions(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT valid_vote CHECK (voter_id IS NOT NULL)
);
```

**Columns:**
- `id` - Unique identifier
- `student_id` - Student being voted for
- `voter_id` - User casting the vote
- `competition_id` - Competition this vote is for

---

## FUNCTIONS

### 1. GET_STUDENT_COUNTS
Returns total student count and gender breakdown for a school.

```sql
CREATE OR REPLACE FUNCTION get_student_counts(p_school_id UUID)
RETURNS TABLE (
    total_students INTEGER,
    boys_count INTEGER,
    girls_count INTEGER
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        COUNT(*)::INTEGER as total_students,
        COUNT(*) FILTER (WHERE gender = 'male')::INTEGER as boys_count,
        COUNT(*) FILTER (WHERE gender = 'female')::INTEGER as girls_count
    FROM students
    WHERE school_id = p_school_id;
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
-- Get student counts for a school
SELECT * FROM get_student_counts('550e8400-e29b-41d4-a716-446655440000');

-- Returns:
-- total_students | boys_count | girls_count
-- 8              | 4          | 4
```

---

### 2. GET_TOP_STUDENTS
Returns top students by average score across all competitions.

```sql
CREATE OR REPLACE FUNCTION get_top_students(p_limit INTEGER DEFAULT 10)
RETURNS TABLE (
    student_id UUID,
    student_name TEXT,
    school_id UUID,
    school_name TEXT,
    gender TEXT,
    avg_score NUMERIC
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        s.id,
        s.name,
        s.school_id,
        sch.name,
        s.gender,
        AVG(cs.score)::NUMERIC as avg_score
    FROM students s
    JOIN schools sch ON s.school_id = sch.id
    LEFT JOIN competition_scores cs ON s.id = cs.student_id
    GROUP BY s.id, s.name, s.school_id, sch.name, s.gender
    ORDER BY avg_score DESC NULLS LAST
    LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
-- Get top 10 students overall
SELECT * FROM get_top_students(10);

-- Get top 5 students
SELECT * FROM get_top_students(5);

-- Returns:
-- student_id | student_name | school_id | school_name | gender | avg_score
-- ... results sorted by avg_score DESC
```

---

### 3. GET_SCHOOL_FINALISTS
Returns all finalists for a specific school.

```sql
CREATE OR REPLACE FUNCTION get_school_finalists(p_school_id UUID)
RETURNS TABLE (
    student_id UUID,
    student_name TEXT,
    gender TEXT,
    score INTEGER
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        f.student_id,
        s.name,
        f.gender,
        f.score
    FROM finalists f
    JOIN students s ON f.student_id = s.id
    WHERE f.school_id = p_school_id;
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
-- Get finalists for a school
SELECT * FROM get_school_finalists('550e8400-e29b-41d4-a716-446655440000');

-- Returns:
-- student_id | student_name | gender | score
-- uuid1      | John Smith   | male   | 95
-- uuid2      | Jane Doe     | female | 92
```

---

## ROW LEVEL SECURITY POLICIES

### SCHOOLS TABLE Security

#### 1. View Policy
```sql
CREATE POLICY "Users can view own school"
    ON schools FOR SELECT
    USING (auth.uid() = user_id);
```
**Effect:** Users can only view their own school profile.

#### 2. Insert Policy
```sql
CREATE POLICY "Users can create school"
    ON schools FOR INSERT
    WITH CHECK (auth.uid() = user_id);
```
**Effect:** Users can only create a school for themselves.

#### 3. Update Policy
```sql
CREATE POLICY "Users can update own school"
    ON schools FOR UPDATE
    USING (auth.uid() = user_id);
```
**Effect:** Users can only update their own school.

---

### STUDENTS TABLE Security

#### 1. View Policy
```sql
CREATE POLICY "Students are viewable by all"
    ON students FOR SELECT
    USING (true);
```
**Effect:** All students are visible to everyone (public profile).

#### 2. Insert Policy
```sql
CREATE POLICY "School owner can manage students"
    ON students FOR INSERT
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM schools 
            WHERE schools.id = students.school_id 
            AND schools.user_id = auth.uid()
        )
    );
```
**Effect:** Only school owner can add students to their school.

#### 3. Update Policy
```sql
CREATE POLICY "School owner can update students"
    ON students FOR UPDATE
    USING (
        EXISTS (
            SELECT 1 FROM schools 
            WHERE schools.id = students.school_id 
            AND schools.user_id = auth.uid()
        )
    );
```
**Effect:** Only school owner can update students.

#### 4. Delete Policy
```sql
CREATE POLICY "School owner can delete students"
    ON students FOR DELETE
    USING (
        EXISTS (
            SELECT 1 FROM schools 
            WHERE schools.id = students.school_id 
            AND schools.user_id = auth.uid()
        )
    );
```
**Effect:** Only school owner can delete students.

---

### COMPETITION_SCORES TABLE Security

#### 1. View Policy
```sql
CREATE POLICY "Scores are viewable by all"
    ON competition_scores FOR SELECT
    USING (true);
```
**Effect:** All scores are public (for leaderboards/rankings).

#### 2. Insert Policy
```sql
CREATE POLICY "School owner can manage scores"
    ON competition_scores FOR INSERT
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM schools 
            WHERE schools.id = competition_scores.school_id 
            AND schools.user_id = auth.uid()
        )
    );
```
**Effect:** Only school owner can insert scores for their school.

---

### FINALISTS TABLE Security

#### 1. View Policy
```sql
CREATE POLICY "Finalists are viewable by all"
    ON finalists FOR SELECT
    USING (true);
```
**Effect:** All finalists are visible to everyone.

---

## INDEXES

**Performance optimization indexes:**

```sql
-- Schools
CREATE INDEX idx_schools_user_id ON schools(user_id);

-- Students
CREATE INDEX idx_students_school_id ON students(school_id);
CREATE INDEX idx_students_gender ON students(gender);

-- Competitions
CREATE INDEX idx_competitions_school_id ON competitions(school_id);

-- Competition Scores
CREATE INDEX idx_competition_scores_competition_id ON competition_scores(competition_id);
CREATE INDEX idx_competition_scores_student_id ON competition_scores(student_id);
CREATE INDEX idx_competition_scores_school_id ON competition_scores(school_id);
CREATE INDEX idx_competition_scores_gender ON competition_scores(gender);

-- Finalists
CREATE INDEX idx_finalists_competition_id ON finalists(competition_id);
CREATE INDEX idx_finalists_student_id ON finalists(student_id);
CREATE INDEX idx_finalists_school_id ON finalists(school_id);
CREATE INDEX idx_finalists_gender ON finalists(gender);

-- Results
CREATE INDEX idx_results_school_id ON results(school_id);
CREATE INDEX idx_results_competition_id ON results(competition_id);

-- Votes
CREATE INDEX idx_votes_student_id ON votes(student_id);
CREATE INDEX idx_votes_competition_id ON votes(competition_id);
```

---

## PROFILE PICTURE HANDLING

### School Profile Picture
**Column:** `schools.profile_pic` (TEXT)

#### Storing School Logo/Picture
```sql
-- Example: Update school profile picture
UPDATE schools 
SET profile_pic = '[BASE64_ENCODED_IMAGE]'
WHERE id = '[school_id]';
```

#### In JavaScript - Upload & Save
```javascript
// Read file as Base64
const reader = new FileReader();
reader.onload = (event) => {
    const base64Image = event.target.result; // e.g., "data:image/png;base64,iVBORw0KG..."
    
    // Save to database
    const sb = await getSupabase();
    await sb.from('schools').update({ 
        profile_pic: base64Image 
    }).eq('id', schoolId);
};
reader.readAsDataURL(file);
```

#### Displaying School Picture
```html
<!-- In HTML -->
<img src="profile_pic" alt="School Logo" style="width: 150px; height: 150px; border-radius: 50%;">
```

```javascript
// In JavaScript
document.getElementById('schoolLogo').src = school.profile_pic;
```

### Student Profile Picture
**Column:** `students.profile_pic` (TEXT)

#### Storing Student Photo
```sql
-- Example: Update student profile picture
UPDATE students 
SET profile_pic = '[BASE64_ENCODED_IMAGE]'
WHERE id = '[student_id]';
```

#### Upload & Display
Similar to school profile picture:
```javascript
// Upload
const reader = new FileReader();
reader.onload = (event) => {
    await sb.from('students').update({ 
        profile_pic: event.target.result 
    }).eq('id', studentId);
};

// Display
<img src="student.profile_pic" alt="Student Photo" width="100" height="100">
```

---

## COMMON SQL QUERIES

### Show all schools and student counts
```sql
SELECT 
    s.id,
    s.name,
    s.location,
    s.approved,
    (SELECT COUNT(*) FROM students WHERE school_id = s.id) as total_students,
    (SELECT COUNT(*) FROM students WHERE school_id = s.id AND gender = 'male') as boys,
    (SELECT COUNT(*) FROM students WHERE school_id = s.id AND gender = 'female') as girls
FROM schools s
ORDER BY s.name;
```

### Show top schools by total score
```sql
SELECT 
    s.name,
    COUNT(DISTINCT cs.student_id) as students_with_scores,
    AVG(cs.score) as avg_score,
    MAX(cs.score) as max_score
FROM schools s
LEFT JOIN competition_scores cs ON s.id = cs.school_id
GROUP BY s.id, s.name
ORDER BY avg_score DESC;
```

### Show competition leaderboard
```sql
SELECT 
    s.name as school_name,
    std.name as student_name,
    std.gender,
    cs.score,
    cs.rank
FROM competition_scores cs
JOIN students std ON cs.student_id = std.id
JOIN schools s ON cs.school_id = s.id
WHERE cs.competition_id = '[competition_id]'
ORDER BY cs.rank ASC;
```

### Show students by gender for a school
```sql
SELECT 
    name,
    gender,
    age,
    created_at
FROM students
WHERE school_id = '[school_id]'
ORDER BY gender, name;
```

