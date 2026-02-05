# 02 - Database Schema

> **Agent:** Database Agent 🗄️
> **Goal:** Set up Supabase database with all tables and policies

---

## ⚠️ CRITICAL: Avoid Duplicate Fields

**NEVER define these fields in your custom schema - they are automatically added:**
- `id` - UUID Primary Key (auto-generated)
- `user_id` - UUID Foreign Key to auth.users (auto-generated)
- `created_at` - TIMESTAMPTZ (auto-generated)
- `updated_at` - TIMESTAMPTZ (auto-generated)

**Only define entity-SPECIFIC fields in your tables.**

Example of WRONG vs RIGHT:
```sql
-- WRONG: Duplicate fields!
CREATE TABLE items (
  id UUID PRIMARY KEY,       -- Already auto-added!
  user_id UUID,              -- Already auto-added!
  created_at TIMESTAMPTZ,    -- Already auto-added!
  name TEXT,
  id UUID,                   -- DUPLICATE! Will cause error!
  created_at TIMESTAMPTZ     -- DUPLICATE! Will cause error!
);

-- RIGHT: Only custom fields
CREATE TABLE items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  -- Only custom fields below:
  name TEXT NOT NULL,
  description TEXT,
  status TEXT NOT NULL DEFAULT 'active'
);
```

---

## Context

**Product:** MemoryMosaic Premium
**Entities:** Idea, Story, User

### Data Model

**Idea**: A user's idea for a story
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `title`: TEXT (required)
- `description`: TEXT
Relationships:
  - one_to_many → User: A user can have many ideas

**Story**: A user's story
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `title`: TEXT (required)
- `content`: TEXT (required)
Relationships:
  - many_to_one → Idea: A story is based on an idea
  - one_to_many → User: A user can have many stories

**User**: A user of the platform
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `name`: TEXT (required)
- `email`: TEXT (required, unique)
- `password`: TEXT (required)
Relationships:
  - one_to_many → Idea: A user can have many ideas
  - one_to_many → Story: A user can have many stories

---

## Instructions

### 1. Profiles Table

Create a **profiles** table that syncs with auth.users:

**Requirements:**
- Primary Key: `id` (UUID) - Reference to auth.users(id)
- Fields: username (unique), display_name, avatar_url, bio
- Timestamps: created_at, updated_at
- RLS: Everyone can view profiles, only owner can update
- Auto-Trigger: Automatically create profile on user signup

**Important:**
- ON DELETE CASCADE for foreign key to auth.users
- Trigger Function for auto-create on signup
- Public read access, owner-only write access

### 2. Entity Tables

Create a table for each entity:

**1. Idea Table (`ideas`):**
Purpose: A user's idea for a story

```sql
CREATE TABLE ideas (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  title TEXT NOT NULL,
  description TEXT

);

-- Indexes for performance
CREATE INDEX ideas_user_id_idx ON ideas(user_id);
CREATE INDEX ideas_created_at_idx ON ideas(created_at DESC);



-- RLS Policies (Granular)
ALTER TABLE ideas ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "ideas_select_own"
  ON ideas FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "ideas_insert_own"
  ON ideas FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "ideas_update_own"
  ON ideas FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "ideas_delete_own"
  ON ideas FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- one_to_many to User (foreign key: user_id)

**2. Story Table (`storys`):**
Purpose: A user's story

```sql
CREATE TABLE storys (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  title TEXT NOT NULL,
  content TEXT NOT NULL

);

-- Indexes for performance
CREATE INDEX storys_user_id_idx ON storys(user_id);
CREATE INDEX storys_created_at_idx ON storys(created_at DESC);



-- RLS Policies (Granular)
ALTER TABLE storys ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "storys_select_own"
  ON storys FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "storys_insert_own"
  ON storys FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "storys_update_own"
  ON storys FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "storys_delete_own"
  ON storys FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- many_to_one to Idea (foreign key: idea_id)
- one_to_many to User (foreign key: user_id)

**3. User Table (`users`):**
Purpose: A user of the platform

```sql
CREATE TABLE users (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  password TEXT NOT NULL

);

-- Indexes for performance
CREATE INDEX users_user_id_idx ON users(user_id);
CREATE INDEX users_created_at_idx ON users(created_at DESC);



-- RLS Policies (Granular)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "users_select_own"
  ON users FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "users_insert_own"
  ON users FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "users_update_own"
  ON users FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "users_delete_own"
  ON users FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- one_to_many to Idea (foreign key: idea_id)
- one_to_many to Story (foreign key: story_id)


### 3. Updated-At Automation

Create a **Trigger Function** that automatically sets `updated_at`:
- Function Name: `update_updated_at()`
- Trigger: BEFORE UPDATE on all tables
- Logic: SET NEW.updated_at = now()

Apply the trigger to all tables (profiles + entities).



### 5. Type Safety

Generate TypeScript types from the schema:
- Use Supabase CLI: `supabase gen types typescript`
- Export to: `lib/database.types.ts`
- Use the types for type-safe queries

---

## Validation Checklist

Verify that:
- [ ] profiles table exists with RLS
- [ ] Auto-create Profile Trigger works
- [ ] Idea table exists with correct fields
- [ ] Story table exists with correct fields
- [ ] User table exists with correct fields
- [ ] All tables have RLS enabled
- [ ] Foreign Keys correctly set
- [ ] Indexes created for performance
- [ ] updated_at Trigger on all tables

- [ ] TypeScript Types generated

**Test:**
- Signup new user → Profile automatically created
- Create Entity record → Only visible to owner
- Update record → updated_at automatically set


---

## Troubleshooting

**RLS Issues:**
- Check if policies correctly use auth.uid()
- Test with SQL Editor: `SELECT auth.uid()`
- Policies must have USING and WITH CHECK

**Foreign Key Errors:**
- Check CASCADE on user_id
- Entities with relationships: Correct order when creating

**Storage Issues:**
- Folder structure must follow {user_id}/ pattern
- Policies use storage.foldername() for user isolation

---

**Continue to:** `03-auth.md`
