# Database Unit - Domain Entities

## Core Domain Model

Based on the functional design decisions, the database implements a comprehensive relational model with vector storage capabilities, GDPR compliance, and performance optimization.

## Primary Entities

### User Entity
**Purpose**: Store user account information with GDPR compliance and consent tracking

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    auth0_user_id VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login_at TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    
    -- GDPR Compliance Fields
    consent_given_at TIMESTAMP WITH TIME ZONE,
    consent_version VARCHAR(50),
    data_export_requested_at TIMESTAMP WITH TIME ZONE,
    data_deletion_requested_at TIMESTAMP WITH TIME ZONE,
    
    -- Audit Fields
    created_by VARCHAR(255),
    updated_by VARCHAR(255)
);
```

**Key Attributes**:
- **id**: Primary key (UUID for security and distribution)
- **email**: User's email address (unique, required for authentication)
- **name**: Display name for the user
- **auth0_user_id**: External Auth0 user identifier for authentication
- **consent_given_at**: Timestamp when user gave GDPR consent
- **consent_version**: Version of privacy policy user consented to
- **data_export_requested_at**: Timestamp for GDPR data export requests
- **data_deletion_requested_at**: Timestamp for GDPR data deletion requests

**Business Rules**:
- Email must be unique across all users
- Auth0 user ID must be unique and not null
- Consent tracking required for GDPR compliance
- Soft delete capability for data retention policies

### Recipe Entity
**Purpose**: Store recipe information with metadata and AI processing status

```sql
CREATE TABLE recipes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(500) NOT NULL,
    ingredients TEXT NOT NULL,
    instructions TEXT NOT NULL,
    
    -- Metadata Fields
    cooking_time_minutes INTEGER,
    servings INTEGER,
    difficulty_level VARCHAR(50), -- 'easy', 'medium', 'hard'
    cuisine_type VARCHAR(100),
    meal_type VARCHAR(100), -- 'breakfast', 'lunch', 'dinner', 'snack'
    
    -- AI Processing Status
    ai_processed BOOLEAN DEFAULT FALSE,
    ai_processing_started_at TIMESTAMP WITH TIME ZONE,
    ai_processing_completed_at TIMESTAMP WITH TIME ZONE,
    ai_processing_error TEXT,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Audit Fields
    created_by VARCHAR(255),
    updated_by VARCHAR(255)
);
```

**Key Attributes**:
- **id**: Primary key (UUID)
- **user_id**: Foreign key to users table (cascade delete for privacy)
- **title**: Recipe name (required, searchable)
- **ingredients**: Free-form ingredient text (will be parsed by AI)
- **instructions**: Cooking instructions
- **ai_processed**: Flag indicating if AI processing is complete
- **ai_processing_***: Tracking fields for AI processing pipeline

**Business Rules**:
- Title, ingredients, and instructions are required
- Recipes belong to exactly one user
- AI processing status must be tracked for pipeline management
- Cascade delete when user is deleted (GDPR compliance)

### Recipe Embeddings Entity
**Purpose**: Store semantic embeddings for vector similarity search using pgvector

```sql
CREATE TABLE recipe_embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipe_id UUID NOT NULL REFERENCES recipes(id) ON DELETE CASCADE,
    embedding vector(1536), -- OpenAI text-embedding-3-small dimension
    embedding_model VARCHAR(100) NOT NULL DEFAULT 'text-embedding-3-small',
    embedding_version VARCHAR(50) NOT NULL,
    
    -- Processing Metadata
    source_text_hash VARCHAR(64) NOT NULL, -- SHA-256 of source text
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- HNSW Index for fast vector similarity search
CREATE INDEX idx_recipe_embeddings_vector ON recipe_embeddings 
USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
```

**Key Attributes**:
- **recipe_id**: Foreign key to recipes table
- **embedding**: pgvector column storing 1536-dimensional embedding
- **embedding_model**: OpenAI model used for embedding generation
- **embedding_version**: Version tracking for embedding updates
- **source_text_hash**: Hash of source text to detect changes

**Business Rules**:
- One embedding per recipe (1:1 relationship)
- Embeddings are regenerated when recipe content changes significantly
- HNSW index optimized for cosine similarity search
- Cascade delete when recipe is deleted

### Parsed Ingredients Entity
**Purpose**: Store structured ingredient data parsed from free-form text

```sql
CREATE TABLE parsed_ingredients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipe_id UUID NOT NULL REFERENCES recipes(id) ON DELETE CASCADE,
    
    -- Parsed Ingredient Data
    original_text TEXT NOT NULL,
    ingredient_name VARCHAR(255) NOT NULL,
    quantity DECIMAL(10,3),
    unit VARCHAR(50),
    preparation VARCHAR(255), -- 'chopped', 'diced', 'minced', etc.
    
    -- Standardization
    standardized_name VARCHAR(255),
    ingredient_category VARCHAR(100), -- 'protein', 'vegetable', 'spice', etc.
    
    -- Processing Metadata
    parsing_confidence DECIMAL(3,2), -- 0.00 to 1.00
    parsed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    parsing_model VARCHAR(100) NOT NULL,
    
    -- Order and Grouping
    ingredient_order INTEGER NOT NULL,
    ingredient_group VARCHAR(255) -- 'marinade', 'sauce', 'garnish', etc.
);
```

**Key Attributes**:
- **recipe_id**: Foreign key to recipes table
- **original_text**: Original ingredient text from user input
- **ingredient_name**: Parsed ingredient name
- **quantity**: Numeric quantity (nullable for "to taste" items)
- **unit**: Unit of measurement (cups, tablespoons, etc.)
- **standardized_name**: Standardized ingredient name for matching
- **parsing_confidence**: AI confidence score for parsing accuracy

**Business Rules**:
- Multiple ingredients per recipe (1:many relationship)
- Original text preserved for reference and re-parsing
- Ingredient order maintained for recipe display
- Confidence tracking for quality assurance

### User Taste Profile Entity
**Purpose**: Store dynamic user taste profiles for personalized recommendations

```sql
CREATE TABLE user_taste_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Taste Profile Data (JSON for flexibility)
    cuisine_preferences JSONB, -- {'italian': 0.8, 'mexican': 0.6, ...}
    ingredient_preferences JSONB, -- {'chicken': 0.9, 'mushrooms': -0.3, ...}
    cooking_time_preference INTEGER, -- preferred cooking time in minutes
    difficulty_preference VARCHAR(50), -- 'easy', 'medium', 'hard'
    meal_type_preferences JSONB, -- {'dinner': 0.8, 'breakfast': 0.4, ...}
    
    -- Profile Metadata
    profile_strength DECIMAL(3,2) DEFAULT 0.0, -- 0.0 to 1.0, based on data volume
    last_recipe_count INTEGER DEFAULT 0, -- number of recipes when last updated
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_calculated_at TIMESTAMP WITH TIME ZONE
);
```

**Key Attributes**:
- **user_id**: Foreign key to users table (1:1 relationship)
- **cuisine_preferences**: JSON object with cuisine types and preference scores
- **ingredient_preferences**: JSON object with ingredients and preference scores
- **profile_strength**: Confidence metric based on amount of user data
- **last_recipe_count**: Tracking field to determine when profile needs updating

**Business Rules**:
- One taste profile per user
- Preferences stored as scores between -1.0 and 1.0
- Profile automatically updated when user adds/modifies recipes
- JSON storage for flexible preference categories

## Supporting Entities

### User Sessions Entity
**Purpose**: Track user sessions for security and analytics

```sql
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_accessed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ip_address INET,
    user_agent TEXT
);
```

### Data Export Requests Entity
**Purpose**: Track GDPR data export requests

```sql
CREATE TABLE data_export_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    request_status VARCHAR(50) NOT NULL DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
    export_format VARCHAR(50) NOT NULL DEFAULT 'json', -- 'json', 'csv'
    file_path VARCHAR(500),
    expires_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE
);
```

### Audit Log Entity
**Purpose**: Comprehensive audit logging for security and compliance

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    entity_type VARCHAR(100) NOT NULL, -- 'user', 'recipe', 'taste_profile'
    entity_id UUID,
    action VARCHAR(100) NOT NULL, -- 'create', 'update', 'delete', 'view'
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Entity Relationships

### Primary Relationships
- **User → Recipes**: One-to-Many (cascade delete)
- **User → User Taste Profile**: One-to-One (cascade delete)
- **Recipe → Recipe Embeddings**: One-to-One (cascade delete)
- **Recipe → Parsed Ingredients**: One-to-Many (cascade delete)
- **User → User Sessions**: One-to-Many (cascade delete)
- **User → Data Export Requests**: One-to-Many (cascade delete)

### Referential Integrity
- All foreign keys enforce referential integrity
- Cascade deletes ensure GDPR compliance (complete user data removal)
- Audit logs maintain referential integrity through soft references

## Data Validation Rules

### User Entity Validation
- Email format validation (RFC 5322 compliant)
- Name length between 1 and 255 characters
- Auth0 user ID format validation
- Consent timestamp required for active users

### Recipe Entity Validation
- Title length between 1 and 500 characters
- Ingredients and instructions cannot be empty
- Cooking time must be positive integer if provided
- Servings must be positive integer if provided
- Difficulty level must be in allowed values

### Embedding Entity Validation
- Embedding vector must be exactly 1536 dimensions
- Source text hash must be valid SHA-256
- Embedding model must be from approved list

### Parsed Ingredients Validation
- Quantity must be positive if provided
- Ingredient order must be positive integer
- Parsing confidence must be between 0.00 and 1.00

### Taste Profile Validation
- Preference scores must be between -1.0 and 1.0
- Profile strength must be between 0.0 and 1.0
- JSON fields must contain valid JSON objects

## Performance Considerations

### Indexing Strategy
- Primary keys: Automatic UUID indexes
- Foreign keys: Automatic indexes for referential integrity
- Search fields: Full-text search indexes on recipe titles and ingredients
- Vector similarity: HNSW index on embeddings
- Composite indexes: User-recipe queries, timestamp ranges

### Partitioning Strategy
- Audit logs: Partitioned by month for performance and retention
- User sessions: Partitioned by expiration date for cleanup
- Large tables: Consider partitioning by user_id for horizontal scaling

### Data Archival
- Expired sessions: Automatic cleanup after expiration
- Old audit logs: Archive after 2 years, delete after 7 years
- Deleted user data: Immediate purge for GDPR compliance
