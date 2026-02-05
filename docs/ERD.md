# Entity Relationship Diagram - MemoryMosaic Premium

> **Auto-generated** from your idea analysis
> **Entities:** 3

---

## Visual Diagram

```mermaid
erDiagram
    profiles {
        uuid id PK
        text username UK
        text display_name
        text avatar_url
        timestamptz created_at
        timestamptz updated_at
    }

    ideas {
        uuid id PK
        uuid user_id FK
        uuid id
        timestamptz created_at
        timestamptz updated_at
        uuid user_id FK
        text title
        text description
        timestamptz created_at
        timestamptz updated_at
    }

    storys {
        uuid id PK
        uuid user_id FK
        uuid id
        timestamptz created_at
        timestamptz updated_at
        uuid user_id FK
        text title
        text content
        timestamptz created_at
        timestamptz updated_at
    }

    users {
        uuid id PK
        uuid user_id FK
        uuid id
        timestamptz created_at
        timestamptz updated_at
        uuid user_id FK
        text name
        text email UK
        text password
        timestamptz created_at
        timestamptz updated_at
    }

    %% Relationships
    profiles ||--o{ ideas : owns
    profiles ||--o{ storys : owns
    profiles ||--o{ users : owns
    ideas ||--o{ users : "A user can have many ideas"
    storys }o--|| ideas : "A story is based on an idea"
    storys ||--o{ users : "A user can have many stories"
    users ||--o{ ideas : "A user can have many ideas"
    users ||--o{ storys : "A user can have many stories"
```

---

## Entity Details

### Idea
> A user's idea for a story

**Fields:**
  - `id`: uuid (required) - Primary key
  - `created_at`: datetime (required) - Creation timestamp
  - `updated_at`: datetime (required) - Last update timestamp
  - `user_id`: uuid (required) - Owner user ID
  - `title`: string (required)
  - `description`: text

**Relationships:**
  - one_to_many → **User**: A user can have many ideas

### Story
> A user's story

**Fields:**
  - `id`: uuid (required) - Primary key
  - `created_at`: datetime (required) - Creation timestamp
  - `updated_at`: datetime (required) - Last update timestamp
  - `user_id`: uuid (required) - Owner user ID
  - `title`: string (required)
  - `content`: text (required)

**Relationships:**
  - many_to_one → **Idea**: A story is based on an idea
  - one_to_many → **User**: A user can have many stories

### User
> A user of the platform

**Fields:**
  - `id`: uuid (required) - Primary key
  - `created_at`: datetime (required) - Creation timestamp
  - `updated_at`: datetime (required) - Last update timestamp
  - `user_id`: uuid (required) - Owner user ID
  - `name`: string (required)
  - `email`: string (required, unique)
  - `password`: string (required)

**Relationships:**
  - one_to_many → **Idea**: A user can have many ideas
  - one_to_many → **Story**: A user can have many stories

---

## Notes

- All entities have standard fields: `id`, `user_id`, `created_at`, `updated_at`
- `PK` = Primary Key, `FK` = Foreign Key, `UK` = Unique Key
- Copy the Mermaid code block to visualize in any Mermaid-compatible tool
- Relationships: `||--o{` = one-to-many, `||--||` = one-to-one, `}o--o{` = many-to-many
