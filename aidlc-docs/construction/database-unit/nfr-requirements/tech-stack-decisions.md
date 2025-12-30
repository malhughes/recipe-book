# Database Unit - Technology Stack Decisions

## Overview

This document outlines the specific technology stack decisions for the Database Unit based on the NFR requirements assessment. The selections prioritize **cost optimization**, **good performance**, and **operational simplicity** for a small-scale recipe book application.

## Primary Database Technology

### PostgreSQL Selection and Configuration

**Decision**: PostgreSQL 15+ with pgvector extension on AWS RDS

**Rationale**:
- **Proven Reliability**: Mature, ACID-compliant relational database
- **Vector Support**: Native pgvector extension for semantic embeddings
- **AWS Integration**: Managed service reduces operational overhead
- **Cost Effectiveness**: Good performance-to-cost ratio for small scale
- **GDPR Compliance**: Strong data integrity and audit capabilities

**Configuration Specifications**:
```yaml
Database Engine: PostgreSQL 15.4
Instance Class: db.t3.medium
  - vCPUs: 2
  - Memory: 4 GB
  - Network Performance: Up to 5 Gbps
Storage:
  - Type: General Purpose SSD (gp2)
  - Initial Size: 100 GB
  - Auto Scaling: Enabled (up to 500 GB)
  - IOPS: 300 baseline, burst to 3,000
Backup:
  - Automated Backups: Enabled
  - Retention Period: 30 days
  - Backup Window: 03:00-04:00 UTC
  - Maintenance Window: Sunday 04:00-05:00 UTC
```

**Performance Tuning Parameters**:
```sql
-- Memory Configuration
shared_buffers = '1GB'                    -- 25% of 4GB memory
effective_cache_size = '3GB'              -- 75% of 4GB memory
work_mem = '4MB'                          -- Per connection work memory
maintenance_work_mem = '64MB'             -- Maintenance operations

-- Connection Configuration
max_connections = 100                     -- Maximum connections
shared_preload_libraries = 'pg_stat_statements,pgvector'

-- Query Performance
random_page_cost = 1.1                    -- SSD optimization
effective_io_concurrency = 200            -- SSD concurrent I/O

-- Logging Configuration
log_min_duration_statement = 2000         -- Log queries >2 seconds
log_checkpoints = on
log_connections = on
log_disconnections = on
```

## Vector Database Extension

### pgvector Configuration

**Decision**: pgvector extension with HNSW indexing

**Rationale**:
- **Native Integration**: Seamless integration with PostgreSQL
- **Performance**: HNSW indexes provide excellent search performance
- **Cost Effective**: No additional vector database licensing costs
- **Operational Simplicity**: Single database system to manage
- **Proven Scale**: Handles 1M+ embeddings efficiently

**Vector Configuration**:
```sql
-- Extension Installation
CREATE EXTENSION IF NOT EXISTS vector;

-- Index Configuration for Recipe Embeddings
CREATE INDEX idx_recipe_embeddings_vector 
ON recipe_embeddings 
USING hnsw (embedding vector_cosine_ops) 
WITH (m = 16, ef_construction = 64);

-- Runtime Parameters
SET hnsw.ef_search = 40;                  -- Query-time search parameter
```

**Index Parameters Explanation**:
- **m = 16**: Connectivity parameter (balance between accuracy and speed)
- **ef_construction = 64**: Build-time search parameter (higher = better accuracy)
- **ef_search = 40**: Query-time search parameter (adjustable per query)

## Caching Technology

### Redis Configuration

**Decision**: Single Redis instance on AWS ElastiCache

**Rationale**:
- **Cost Optimization**: Single instance sufficient for small scale
- **Performance**: In-memory caching for sub-millisecond access
- **AWS Integration**: Managed service with automated backups
- **Operational Simplicity**: No clustering complexity needed
- **Proven Reliability**: Mature caching technology

**Configuration Specifications**:
```yaml
Engine: Redis 7.0
Node Type: cache.t3.micro
  - vCPUs: 2
  - Memory: 1 GB
  - Network Performance: Up to 5 Gbps
Configuration:
  - Parameter Group: default.redis7
  - Port: 6379
  - Encryption in Transit: Enabled
  - Encryption at Rest: Enabled
Backup:
  - Automatic Backups: Enabled
  - Backup Retention: 5 days
  - Backup Window: 05:00-06:00 UTC
```

**Redis Configuration Parameters**:
```redis
# Memory Management
maxmemory 900mb                           # 90% of 1GB allocation
maxmemory-policy allkeys-lru              # Evict least recently used keys

# Persistence Configuration
save 900 1                                # Save if 1 key changes in 900 seconds
save 300 10                               # Save if 10 keys change in 300 seconds
save 60 10000                             # Save if 10000 keys change in 60 seconds

# Performance Tuning
tcp-keepalive 300                         # TCP keepalive
timeout 0                                 # Client idle timeout (disabled)
```

## Connection Management

### Connection Pooling Strategy

**Decision**: PgBouncer for connection pooling

**Rationale**:
- **Connection Efficiency**: Reduces database connection overhead
- **Resource Optimization**: Maximizes database connection utilization
- **Performance**: Faster connection establishment for applications
- **Cost Effective**: Reduces need for larger database instances

**PgBouncer Configuration**:
```ini
[databases]
recipe_db = host=rds-endpoint.amazonaws.com port=5432 dbname=recipe_book

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt

# Connection Limits
max_client_conn = 100                     # Maximum client connections
default_pool_size = 20                    # Default pool size per database
min_pool_size = 5                         # Minimum pool size
reserve_pool_size = 5                     # Reserve connections

# Pool Configuration
pool_mode = transaction                   # Transaction-level pooling
server_reset_query = DISCARD ALL         # Reset query between transactions
server_check_delay = 30                   # Health check interval

# Performance Tuning
server_connect_timeout = 15               # Connection timeout
server_login_retry = 15                   # Login retry interval
query_timeout = 0                         # Query timeout (disabled)
```

## AWS Infrastructure Decisions

### RDS Configuration

**Decision**: AWS RDS PostgreSQL with Multi-AZ for production

**Infrastructure Specifications**:
```yaml
RDS Configuration:
  Engine: postgres
  Engine Version: "15.4"
  Instance Class: db.t3.medium
  Allocated Storage: 100
  Storage Type: gp2
  Storage Encrypted: true
  
  # High Availability (Production)
  Multi AZ: false                         # Cost optimization for small scale
  
  # Backup Configuration
  Backup Retention Period: 30
  Backup Window: "03:00-04:00"
  Maintenance Window: "sun:04:00-sun:05:00"
  
  # Security Configuration
  VPC Security Group: database-sg
  DB Subnet Group: private-subnet-group
  Publicly Accessible: false
  
  # Monitoring
  Performance Insights Enabled: true
  Performance Insights Retention: 7       # Days (free tier)
  Monitoring Interval: 60                 # Seconds
  Enhanced Monitoring: true
```

### ElastiCache Configuration

**Decision**: AWS ElastiCache Redis with basic configuration

**Infrastructure Specifications**:
```yaml
ElastiCache Configuration:
  Engine: redis
  Engine Version: "7.0"
  Node Type: cache.t3.micro
  Num Cache Nodes: 1
  
  # Security Configuration
  Security Group: cache-sg
  Subnet Group: private-subnet-group
  
  # Backup Configuration
  Snapshot Retention Limit: 5
  Snapshot Window: "05:00-06:00"
  
  # Maintenance
  Maintenance Window: "sun:06:00-sun:07:00"
  
  # Encryption
  Transit Encryption Enabled: true
  At Rest Encryption Enabled: true
```

## Security Configuration

### Network Security

**VPC and Security Group Configuration**:
```yaml
Database Security Group (database-sg):
  Inbound Rules:
    - Type: PostgreSQL
      Protocol: TCP
      Port: 5432
      Source: application-sg
      Description: "Application access to database"
    
    - Type: Custom TCP
      Protocol: TCP  
      Port: 6432
      Source: application-sg
      Description: "PgBouncer connection pooling"

Cache Security Group (cache-sg):
  Inbound Rules:
    - Type: Custom TCP
      Protocol: TCP
      Port: 6379
      Source: application-sg
      Description: "Application access to Redis cache"

  Outbound Rules:
    - Type: All Traffic
      Protocol: All
      Port: All
      Destination: 0.0.0.0/0
      Description: "Allow all outbound traffic"
```

### Encryption Configuration

**Data Encryption Strategy**:
```yaml
Encryption at Rest:
  RDS: 
    - KMS Key: AWS Managed Key (aws/rds)
    - Algorithm: AES-256
  ElastiCache:
    - KMS Key: AWS Managed Key (aws/elasticache)
    - Algorithm: AES-256

Encryption in Transit:
  Database Connections:
    - Protocol: TLS 1.2+
    - Certificate Verification: Required
  Cache Connections:
    - Protocol: TLS 1.2+
    - AUTH Token: Enabled
```

## Monitoring and Observability Stack

### CloudWatch Configuration

**Monitoring Strategy**:
```yaml
CloudWatch Metrics:
  RDS Metrics:
    - CPUUtilization
    - DatabaseConnections
    - ReadLatency / WriteLatency
    - ReadIOPS / WriteIOPS
    - FreeStorageSpace
    - ReplicaLag (if applicable)
  
  ElastiCache Metrics:
    - CPUUtilization
    - CacheHitRate
    - CacheMissRate
    - CurrentConnections
    - NetworkBytesIn / NetworkBytesOut
    - EvictionCount

CloudWatch Alarms:
  Database Alarms:
    - CPU > 70% for 5 minutes
    - Connections > 80% for 5 minutes
    - Free Storage < 20% for 5 minutes
    - Read/Write Latency > 200ms for 5 minutes
  
  Cache Alarms:
    - CPU > 80% for 5 minutes
    - Cache Hit Rate < 70% for 10 minutes
    - Connection Count > 80% for 5 minutes
```

### Application Performance Monitoring

**Performance Insights Configuration**:
```yaml
RDS Performance Insights:
  Enabled: true
  Retention Period: 7 days              # Free tier
  
  Key Metrics:
    - Top SQL statements by execution time
    - Database load by wait events
    - Connection and session analysis
    - Lock contention analysis

Custom Application Metrics:
  Database Metrics:
    - Query execution time by operation type
    - Connection pool utilization
    - Transaction rollback rate
    - Vector search performance
  
  Cache Metrics:
    - Cache hit rate by data type
    - Cache eviction rate
    - Key expiration patterns
```

## Migration and Deployment Strategy

### Database Migration Tools

**Migration Framework**: Alembic (SQLAlchemy)

**Migration Strategy**:
```python
# Migration Configuration
alembic_config = {
    'script_location': 'migrations',
    'sqlalchemy.url': 'postgresql://user:pass@host:5432/db',
    'version_table': 'alembic_version',
    'version_table_schema': 'public'
}

# Migration Patterns
migration_patterns = [
    'Create tables with proper indexes',
    'Add pgvector extension and vector columns',
    'Create HNSW indexes for vector columns',
    'Add audit triggers and functions',
    'Create partitioned tables for audit logs',
    'Add GDPR compliance functions'
]
```

### Deployment Pipeline

**CI/CD Integration**:
```yaml
Database Deployment Pipeline:
  1. Schema Validation:
     - Validate migration scripts
     - Check for breaking changes
     - Verify rollback procedures
  
  2. Testing:
     - Run migrations on test database
     - Execute integration tests
     - Performance test vector operations
  
  3. Production Deployment:
     - Create database backup
     - Apply migrations with rollback plan
     - Verify deployment success
     - Update monitoring dashboards
```

## Cost Optimization Strategy

### Resource Right-Sizing

**Cost Optimization Decisions**:
```yaml
Cost Optimization Measures:
  Database:
    - Instance Type: t3.medium (burstable performance)
    - Storage: gp2 (cost-effective for small workloads)
    - Multi-AZ: Disabled (cost vs. availability trade-off)
    - Reserved Instances: Consider for production
  
  Cache:
    - Instance Type: t3.micro (smallest viable option)
    - Single Node: No clustering overhead
    - Backup Retention: 5 days (balance cost/recovery)
  
  Monitoring:
    - Performance Insights: 7-day retention (free tier)
    - CloudWatch Logs: 30-day retention
    - Custom Metrics: Essential metrics only
```

### Scaling Economics

**Auto-Scaling Strategy**:
```yaml
Scaling Triggers:
  Scale Up Conditions:
    - CPU > 70% for 5 minutes
    - Memory > 80% for 5 minutes
    - Connection Pool > 80% for 5 minutes
  
  Scale Down Conditions:
    - CPU < 30% for 15 minutes
    - Memory < 50% for 15 minutes
    - Connection Pool < 40% for 15 minutes
  
  Cost Controls:
    - Maximum Instance Size: db.t3.large
    - Scaling Cooldown: 15 minutes
    - Budget Alerts: $100/month threshold
```

## Technology Decision Summary

| Component | Technology | Rationale |
|-----------|------------|-----------|
| **Primary Database** | PostgreSQL 15+ on AWS RDS | ACID compliance, pgvector support, managed service |
| **Vector Storage** | pgvector extension | Native integration, cost-effective, proven performance |
| **Caching** | Redis on AWS ElastiCache | High performance, managed service, operational simplicity |
| **Connection Pooling** | PgBouncer | Connection efficiency, resource optimization |
| **Monitoring** | CloudWatch + Performance Insights | AWS native, cost-effective monitoring |
| **Migration** | Alembic | Python ecosystem integration, version control |
| **Encryption** | AWS KMS managed keys | Security compliance, operational simplicity |
| **Backup** | AWS automated backups | Reliability, point-in-time recovery |

This technology stack provides a **cost-optimized**, **performant**, and **operationally simple** foundation for the Database Unit while maintaining the flexibility to scale as the application grows.
