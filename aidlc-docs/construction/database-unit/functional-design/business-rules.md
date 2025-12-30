# Database Unit - Business Rules

## Data Integrity and Validation Rules

Based on the comprehensive functional design decisions, the database unit enforces strict business rules to ensure data quality, security, and compliance with GDPR requirements.

## User Management Business Rules

### User Registration and Authentication Rules
**Rule UR-001**: User Email Uniqueness
- **Description**: Each user email address must be unique across the entire system
- **Implementation**: UNIQUE constraint on users.email column
- **Validation**: Email format validation using RFC 5322 standard
- **Error Handling**: Return specific error code for duplicate email attempts

**Rule UR-002**: Auth0 Integration Consistency
- **Description**: Auth0 user ID must be unique and cannot be null for active users
- **Implementation**: UNIQUE NOT NULL constraint on users.auth0_user_id
- **Validation**: Auth0 user ID format validation (auth0|[alphanumeric])
- **Business Logic**: User creation must include valid Auth0 user ID

**Rule UR-003**: GDPR Consent Requirement
- **Description**: Active users must have valid consent timestamp and version
- **Implementation**: NOT NULL constraint on consent_given_at for active users
- **Validation**: Consent version must match current privacy policy version
- **Business Logic**: Users cannot access features without valid consent

**Rule UR-004**: User Data Retention
- **Description**: User data must be retained according to GDPR requirements
- **Implementation**: Soft delete with data_deletion_requested_at timestamp
- **Business Logic**: Hard delete only after 30-day grace period
- **Audit Requirement**: All user data operations must be logged

### User Profile Management Rules
**Rule UP-001**: Profile Data Validation
- **Description**: User profile data must meet minimum quality standards
- **Implementation**: 
  - Name: 1-255 characters, no special characters except spaces, hyphens, apostrophes
  - Email: Valid RFC 5322 format
- **Validation**: Server-side validation before database insertion
- **Error Handling**: Detailed validation error messages for user feedback

**Rule UP-002**: Profile Update Auditing
- **Description**: All profile updates must be audited for security and compliance
- **Implementation**: Trigger-based audit logging to audit_logs table
- **Business Logic**: Store old and new values for all profile changes
- **Retention**: Audit logs retained for 7 years for compliance

## Recipe Management Business Rules

### Recipe Creation and Ownership Rules
**Rule RC-001**: Recipe Ownership
- **Description**: Recipes must belong to exactly one user and cannot be orphaned
- **Implementation**: NOT NULL foreign key constraint on recipes.user_id
- **Business Logic**: Recipe creation requires valid, active user ID
- **Cascade Rule**: Recipe deletion when user is deleted (GDPR compliance)

**Rule RC-002**: Recipe Content Requirements
- **Description**: Recipes must have minimum required content for usefulness
- **Implementation**: NOT NULL constraints on title, ingredients, instructions
- **Validation Rules**:
  - Title: 1-500 characters, no leading/trailing whitespace
  - Ingredients: Minimum 1 character, maximum 10,000 characters
  - Instructions: Minimum 1 character, maximum 50,000 characters
- **Business Logic**: Empty or whitespace-only content is rejected

**Rule RC-003**: Recipe Metadata Validation
- **Description**: Optional recipe metadata must be within valid ranges
- **Implementation**: CHECK constraints and application-level validation
- **Validation Rules**:
  - cooking_time_minutes: 1-1440 (24 hours maximum)
  - servings: 1-100 (reasonable serving range)
  - difficulty_level: ENUM('easy', 'medium', 'hard')
  - cuisine_type: Predefined list of valid cuisine types
  - meal_type: ENUM('breakfast', 'lunch', 'dinner', 'snack', 'dessert')

### Recipe AI Processing Rules
**Rule RA-001**: AI Processing Pipeline Integrity
- **Description**: AI processing status must be accurately tracked and consistent
- **Implementation**: State machine for ai_processed, ai_processing_started_at, ai_processing_completed_at
- **Business Logic**:
  - New recipes start with ai_processed = FALSE
  - Processing start timestamp required when processing begins
  - Completion timestamp required when processing finishes
  - Error message stored if processing fails
- **Retry Logic**: Failed processing can be retried up to 3 times

**Rule RA-002**: Content Change Detection
- **Description**: AI processing must be triggered when recipe content changes significantly
- **Implementation**: SHA-256 hash comparison of title + ingredients + instructions
- **Business Logic**:
  - Hash stored with each embedding
  - Content changes trigger ai_processed = FALSE
  - Minor changes (punctuation, spacing) may not trigger reprocessing
- **Optimization**: Batch processing for multiple content changes

## Vector Embedding Business Rules

### Embedding Storage and Integrity Rules
**Rule VE-001**: Embedding-Recipe Relationship
- **Description**: Each recipe can have at most one current embedding
- **Implementation**: UNIQUE constraint on recipe_embeddings.recipe_id
- **Business Logic**: New embeddings replace old ones for the same recipe
- **Cascade Rule**: Embeddings deleted when recipe is deleted

**Rule VE-002**: Embedding Dimension Consistency
- **Description**: All embeddings must have consistent dimensions for the model used
- **Implementation**: CHECK constraint on vector dimension (1536 for text-embedding-3-small)
- **Validation**: Embedding dimension validation before storage
- **Business Logic**: Model changes require full re-embedding of all recipes

**Rule VE-003**: Embedding Versioning
- **Description**: Embeddings must be versioned to support model updates
- **Implementation**: embedding_model and embedding_version columns
- **Business Logic**:
  - Version increments trigger re-embedding pipeline
  - Old embeddings marked for cleanup after successful re-embedding
  - Gradual rollout of new embedding versions

### Vector Search Optimization Rules
**Rule VS-001**: Search Index Maintenance
- **Description**: Vector indexes must be maintained for optimal search performance
- **Implementation**: HNSW index with optimized parameters (m=16, ef_construction=64)
- **Business Logic**:
  - Index rebuilding scheduled during low-traffic periods
  - Search performance monitoring with automatic index optimization
  - Fallback to exact search if index is corrupted

**Rule VS-002**: Search Result Filtering
- **Description**: Vector search results must respect user privacy and ownership
- **Implementation**: User-based filtering in similarity search queries
- **Business Logic**:
  - Users can only see their own recipes in search results
  - Deleted or inactive recipes excluded from search
  - Search results limited to reasonable maximum (100 results)

## Parsed Ingredients Business Rules

### Ingredient Parsing and Standardization Rules
**Rule PI-001**: Ingredient Parsing Accuracy
- **Description**: Parsed ingredients must maintain traceability to original text
- **Implementation**: original_text column preserves user input exactly
- **Business Logic**:
  - Original text never modified after initial storage
  - Parsing confidence score tracked for quality assurance
  - Low confidence ingredients flagged for manual review

**Rule PI-002**: Ingredient Standardization
- **Description**: Ingredient names must be standardized for consistent matching
- **Implementation**: standardized_name column with controlled vocabulary
- **Business Logic**:
  - Standardization rules applied consistently across all recipes
  - Synonym mapping for common ingredient variations
  - New ingredients added to standardization dictionary

**Rule PI-003**: Quantity and Unit Validation
- **Description**: Ingredient quantities and units must be reasonable and consistent
- **Implementation**: Validation rules for quantity ranges and unit compatibility
- **Validation Rules**:
  - quantity: Positive decimal, maximum 9999.999
  - unit: From predefined list of valid cooking units
  - Unit-quantity compatibility (e.g., no "5 pinches of flour")
- **Business Logic**: Invalid combinations flagged for review

### Ingredient Relationship Rules
**Rule IR-001**: Recipe-Ingredient Consistency
- **Description**: Parsed ingredients must maintain correct relationship to recipes
- **Implementation**: Foreign key constraint with CASCADE DELETE
- **Business Logic**:
  - All ingredients deleted when recipe is deleted
  - Ingredient order preserved for recipe display
  - Ingredient groups maintained for complex recipes

**Rule IR-002**: Ingredient Search Optimization
- **Description**: Ingredient data must be optimized for search and matching
- **Implementation**: Full-text search indexes on ingredient names
- **Business Logic**:
  - Search includes both original and standardized names
  - Partial matching supported for ingredient discovery
  - Search results ranked by relevance and user preference

## Taste Profile Business Rules

### Profile Calculation and Updates Rules
**Rule TP-001**: Taste Profile Accuracy
- **Description**: Taste profiles must accurately reflect user preferences based on recipe history
- **Implementation**: JSON storage with structured preference scoring
- **Business Logic**:
  - Preferences calculated from user's recipe collection
  - Scores range from -1.0 (dislike) to 1.0 (like)
  - Profile strength indicates confidence based on data volume

**Rule TP-002**: Profile Update Triggers
- **Description**: Taste profiles must be updated when user behavior changes
- **Implementation**: Event-driven updates on recipe additions/modifications
- **Business Logic**:
  - Profile recalculated when user adds/modifies recipes
  - Batch updates for performance optimization
  - Incremental updates for minor changes

**Rule TP-003**: Profile Privacy and Security
- **Description**: Taste profiles must respect user privacy and be securely stored
- **Implementation**: User-specific access controls and encryption
- **Business Logic**:
  - Profiles only accessible by profile owner
  - Profile data included in GDPR data exports
  - Profiles deleted when user account is deleted

### Preference Scoring Rules
**Rule PS-001**: Cuisine Preference Calculation
- **Description**: Cuisine preferences calculated from recipe cuisine distribution
- **Implementation**: Weighted scoring based on recipe frequency and recency
- **Business Logic**:
  - Recent recipes weighted more heavily than old ones
  - Minimum 3 recipes required for reliable cuisine preference
  - Preference decay over time if cuisine not used

**Rule PS-002**: Ingredient Preference Calculation
- **Description**: Ingredient preferences calculated from ingredient usage patterns
- **Implementation**: Frequency analysis with standardized ingredient names
- **Business Logic**:
  - Common ingredients (salt, pepper) excluded from preference calculation
  - Rare ingredients weighted more heavily for preference indication
  - Negative preferences inferred from ingredient avoidance

## Caching Business Rules

### Cache Consistency Rules
**Rule CC-001**: Cache Invalidation Consistency
- **Description**: Cache invalidation must maintain data consistency across all cache levels
- **Implementation**: Event-driven invalidation with pattern matching
- **Business Logic**:
  - User data changes invalidate all user-related caches
  - Recipe changes invalidate recipe, search, and recommendation caches
  - Taste profile changes invalidate recommendation caches

**Rule CC-002**: Cache TTL Management
- **Description**: Cache TTL values must balance performance and data freshness
- **Implementation**: Tiered TTL strategy based on data volatility
- **Business Logic**:
  - User profiles: 1 hour TTL (moderate change frequency)
  - Recipe data: 1 hour TTL (user-controlled changes)
  - Taste profiles: 2 hours TTL (calculated data, less frequent changes)
  - Search results: 15 minutes TTL (dynamic, personalized data)
  - Recommendations: 30 minutes TTL (AI-generated, expensive to compute)

### Cache Performance Rules
**Rule CP-001**: Cache Hit Rate Optimization
- **Description**: Cache hit rates must be monitored and optimized for performance
- **Implementation**: Metrics collection and automated cache warming
- **Business Logic**:
  - Target cache hit rate: >80% for user data, >70% for search results
  - Cache warming for frequently accessed data
  - Cache preloading for new user onboarding

**Rule CP-002**: Cache Memory Management
- **Description**: Cache memory usage must be controlled to prevent resource exhaustion
- **Implementation**: LRU eviction policy with memory limits
- **Business Logic**:
  - Maximum cache size: 80% of available Redis memory
  - Automatic eviction of least recently used items
  - Priority-based eviction (user data > search results > recommendations)

## Data Migration and Schema Rules

### Migration Safety Rules
**Rule MS-001**: Migration Rollback Capability
- **Description**: All schema migrations must be reversible for safe deployment
- **Implementation**: Paired up/down migration methods with validation
- **Business Logic**:
  - Every migration must include rollback procedure
  - Rollback tested in staging environment before production
  - Data migrations preserve original data during transformation

**Rule MS-002**: Migration Data Integrity
- **Description**: Migrations must preserve data integrity and business rule compliance
- **Implementation**: Pre and post-migration validation checks
- **Business Logic**:
  - Foreign key constraints validated after migration
  - Business rule compliance checked post-migration
  - Migration fails if data integrity is compromised

### Schema Evolution Rules
**Rule SE-001**: Backward Compatibility
- **Description**: Schema changes must maintain backward compatibility when possible
- **Implementation**: Additive changes preferred over destructive changes
- **Business Logic**:
  - New columns added as nullable initially
  - Column removal requires deprecation period
  - Index changes applied during maintenance windows

**Rule SE-002**: Performance Impact Assessment
- **Description**: Schema changes must be assessed for performance impact
- **Implementation**: Query plan analysis before and after migration
- **Business Logic**:
  - Large table migrations scheduled during low-traffic periods
  - Index creation uses CONCURRENTLY option when possible
  - Performance regression triggers rollback

## Backup and Recovery Rules

### Backup Integrity Rules
**Rule BR-001**: Backup Completeness
- **Description**: Backups must include all data necessary for complete system recovery
- **Implementation**: Full database backup with transaction log backup
- **Business Logic**:
  - Daily full backups with hourly transaction log backups
  - Backup integrity verified through automated restore testing
  - Cross-region backup replication for disaster recovery

**Rule BR-002**: Recovery Time Objectives
- **Description**: System recovery must meet defined RTO and RPO objectives
- **Implementation**: Automated recovery procedures with monitoring
- **Business Logic**:
  - RTO (Recovery Time Objective): 4 hours maximum
  - RPO (Recovery Point Objective): 1 hour maximum data loss
  - Recovery procedures tested monthly

### Data Retention Rules
**Rule DR-001**: GDPR Data Retention
- **Description**: Personal data retention must comply with GDPR requirements
- **Implementation**: Automated data lifecycle management
- **Business Logic**:
  - User data deleted within 30 days of deletion request
  - Audit logs retained for 7 years for compliance
  - Anonymized analytics data retained indefinitely

**Rule DR-002**: Operational Data Retention
- **Description**: Operational data retained according to business requirements
- **Implementation**: Tiered storage with automated archival
- **Business Logic**:
  - Active user data: Retained while account is active
  - Inactive user data: Archived after 2 years of inactivity
  - System logs: Retained for 1 year, then archived for 5 years

## Security and Access Control Rules

### Data Encryption Rules
**Rule DE-001**: Encryption at Rest
- **Description**: Sensitive data must be encrypted at rest using industry standards
- **Implementation**: AES-256 encryption for database storage
- **Business Logic**:
  - User PII encrypted with user-specific keys
  - Recipe content encrypted with application-level encryption
  - Encryption keys managed through AWS KMS

**Rule DE-002**: Encryption in Transit
- **Description**: All data transmission must be encrypted using TLS
- **Implementation**: TLS 1.3 for all database connections
- **Business Logic**:
  - Database connections require SSL/TLS
  - Certificate validation enforced
  - Weak cipher suites disabled

### Access Control Rules
**Rule AC-001**: Principle of Least Privilege
- **Description**: Database access must follow principle of least privilege
- **Implementation**: Role-based access control with minimal permissions
- **Business Logic**:
  - Application users: Read/write access to own data only
  - Service accounts: Specific permissions for required operations
  - Admin accounts: Elevated permissions with audit logging

**Rule AC-002**: Authentication and Authorization
- **Description**: All database access must be authenticated and authorized
- **Implementation**: Multi-factor authentication for admin access
- **Business Logic**:
  - Service accounts use certificate-based authentication
  - Admin access requires MFA and VPN connection
  - Failed authentication attempts logged and monitored

These comprehensive business rules ensure data integrity, security, performance, and compliance across all database operations in the recipe book application.
