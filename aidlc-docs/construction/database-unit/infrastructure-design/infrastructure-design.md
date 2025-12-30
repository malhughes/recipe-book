# Database Unit - Infrastructure Design

## Overview

This document defines the specific AWS infrastructure services and configurations for the Database Unit, mapping the logical components to actual AWS services. The design prioritizes **cost optimization** while maintaining **good performance** and **operational simplicity** for a small-scale recipe book application.

## Infrastructure Architecture Summary

The Database Unit infrastructure follows a **AWS Free Tier optimized, single-region deployment** strategy with managed AWS services to minimize costs while providing the necessary functionality for the recipe book application.

### Key Infrastructure Decisions (AWS Free Tier Optimized)
- **Database**: AWS RDS PostgreSQL db.t3.micro (Free Tier: 750 hours/month)
- **Caching**: AWS ElastiCache Redis cache.t2.micro (Free Tier: 750 hours/month)
- **Connection Management**: Application-level connection pooling (eliminates RDS Proxy cost)
- **GDPR Compliance**: AWS Lambda functions (Free Tier: 1M requests/month)
- **Networking**: Default VPC with public subnets (eliminates NAT Gateway cost)
- **Monitoring**: CloudWatch Free Tier (10 custom metrics, 5GB logs)
- **Security**: AWS Free Tier security services with basic encryption

## Database Infrastructure

### Primary Database Service

**Service**: AWS RDS PostgreSQL
**Configuration**: Single instance with automated backups

```yaml
# RDS PostgreSQL Configuration (AWS Free Tier Optimized)
rds_postgresql:
  service_type: "AWS RDS"
  engine: "postgres"
  engine_version: "15.4"
  
  instance_configuration:
    instance_class: "db.t3.micro"  # Free Tier: 750 hours/month
    vcpu: 2
    memory: "1 GiB"
    network_performance: "Up to 5 Gbps"
    
  storage_configuration:
    storage_type: "gp2"
    allocated_storage: 20  # Free Tier: 20GB included
    max_allocated_storage: 20  # Keep within free tier limits
    storage_encrypted: false  # Free tier doesn't include encryption
    
  availability_configuration:
    multi_az: false  # Free tier is single AZ only
    availability_zone: "us-east-2a"
    publicly_accessible: true  # Required for free tier without VPC
    
  backup_configuration:
    backup_retention_period: 7  # Free tier includes automated backups
    backup_window: "03:00-04:00"
    copy_tags_to_snapshot: true
    delete_automated_backups: false
    
  maintenance_configuration:
    maintenance_window: "sun:04:00-sun:05:00"
    auto_minor_version_upgrade: true
    
  monitoring_configuration:
    monitoring_interval: 0  # Free tier doesn't include enhanced monitoring
    performance_insights_enabled: false  # Not included in free tier
    enabled_cloudwatch_logs_exports: []  # Minimize log costs
    
  security_configuration:
    vpc_security_group_ids: ["default"]  # Use default VPC
    db_subnet_group_name: "default"  # Use default subnet group
    
  parameter_group_configuration:
    parameter_group_name: "default.postgres15"  # Use default parameter group
    # Custom parameters would require custom parameter group (additional cost)
```

**Free Tier Optimizations**:
- **Instance Hours**: 750 hours/month of db.t3.micro included in Free Tier
- **Storage**: 20GB of General Purpose SSD storage included
- **Backups**: Automated backups included in Free Tier
- **Data Transfer**: 15GB of data transfer out per month included
- **No Enhanced Monitoring**: Saves additional costs
- **Default VPC**: Eliminates custom VPC costs

### pgvector Extension Configuration

**Extension**: pgvector for PostgreSQL
**Configuration**: Standard extension with basic optimization

```sql
-- pgvector Extension Setup (Free Tier Compatible)
CREATE EXTENSION IF NOT EXISTS vector;

-- Basic Vector Index Configuration (no custom parameters)
CREATE INDEX idx_recipe_embeddings_vector 
ON recipe_embeddings 
USING hnsw (embedding vector_cosine_ops);

-- Use default PostgreSQL settings for Free Tier compatibility
-- No custom parameter group modifications to avoid additional costs
```

**Free Tier Vector Storage**:
- **Extension**: pgvector available on RDS PostgreSQL Free Tier
- **Index Type**: HNSW with default parameters
- **Configuration**: Use default PostgreSQL settings to avoid parameter group costs
- **Performance**: Adequate for small-scale vector operations

## Caching Infrastructure

### Redis Cache Service

**Service**: AWS ElastiCache Redis
**Configuration**: Single node with basic persistence

```yaml
# ElastiCache Redis Configuration (AWS Free Tier Optimized)
elasticache_redis:
  service_type: "AWS ElastiCache"
  engine: "redis"
  engine_version: "7.0"
  
  node_configuration:
    node_type: "cache.t2.micro"  # Free Tier: 750 hours/month
    vcpu: 1
    memory: "555 MiB"
    network_performance: "Low to Moderate"
    
  cluster_configuration:
    num_cache_nodes: 1
    port: 6379
    
  availability_configuration:
    availability_zone: "us-east-2a"
    preferred_availability_zones: ["us-east-2a"]
    
  security_configuration:
    security_group_ids: ["default"]  # Use default security group
    subnet_group_name: "default"  # Use default subnet group
    at_rest_encryption_enabled: false  # Not included in free tier
    transit_encryption_enabled: false  # Not included in free tier
    auth_token_enabled: false  # Not required for free tier
    
  backup_configuration:
    snapshot_retention_limit: 0  # No snapshots to minimize costs
    
  maintenance_configuration:
    maintenance_window: "sun:06:00-sun:07:00"
    
  parameter_group_configuration:
    parameter_group_name: "default.redis7"  # Use default parameter group
    # Custom parameters would require custom parameter group (additional cost)
```

**Free Tier Optimizations**:
- **Instance Hours**: 750 hours/month of cache.t2.micro included in Free Tier
- **Memory**: 555 MiB memory allocation
- **No Encryption**: Encryption not included in Free Tier
- **No Snapshots**: Eliminates backup storage costs
- **Default Configuration**: Uses default parameter groups and security groups

## Connection Management Infrastructure

### Application-Level Connection Pooling

**Service**: Application-Level Connection Pooling (PgBouncer or similar)
**Configuration**: Eliminates AWS RDS Proxy costs

```python
# Application-Level Connection Pool Configuration
import psycopg2.pool

class DatabaseConnectionPool:
    def __init__(self):
        self.connection_pool = psycopg2.pool.ThreadedConnectionPool(
            minconn=1,
            maxconn=20,
            host=os.environ['DATABASE_HOST'],
            database=os.environ['DATABASE_NAME'],
            user=os.environ['DATABASE_USER'],
            password=os.environ['DATABASE_PASSWORD'],
            port=5432
        )
    
    def get_connection(self):
        return self.connection_pool.getconn()
    
    def put_connection(self, connection):
        self.connection_pool.putconn(connection)
```

**Free Tier Benefits**:
- **No Additional Service Costs**: Eliminates $10.80/month RDS Proxy cost
- **Application Control**: Full control over connection pooling logic
- **Resource Efficiency**: Optimized for application-specific patterns
- **Simple Implementation**: Easy to implement and maintain

## Compute Infrastructure

### GDPR Compliance Service

**Service**: AWS Lambda Functions
**Configuration**: Serverless functions for cost efficiency

```yaml
# Lambda Functions for GDPR Compliance (AWS Free Tier Optimized)
gdpr_lambda_functions:
  data_export_function:
    function_name: "recipe-gdpr-data-export"
    runtime: "python3.11"
    handler: "lambda_function.lambda_handler"
    memory_size: 128  # Minimum memory for cost optimization
    timeout: 300  # 5 minutes
    
    environment_variables:
      DATABASE_HOST: "${rds_postgresql.endpoint}"
      DATABASE_NAME: "recipe_book"
      DATABASE_USER: "postgres"
      DATABASE_PASSWORD: "${database_password}"  # Use environment variable
      
    # No VPC configuration to avoid NAT Gateway costs
    
    iam_role: "arn:aws:iam::account:role/lambda-basic-execution-role"
    
  data_deletion_function:
    function_name: "recipe-gdpr-data-deletion"
    runtime: "python3.11"
    handler: "lambda_function.lambda_handler"
    memory_size: 128  # Minimum memory for cost optimization
    timeout: 180  # 3 minutes
    
    environment_variables:
      DATABASE_HOST: "${rds_postgresql.endpoint}"
      DATABASE_NAME: "recipe_book"
      DATABASE_USER: "postgres"
      DATABASE_PASSWORD: "${database_password}"
      
    iam_role: "arn:aws:iam::account:role/lambda-basic-execution-role"
    
  audit_logging_function:
    function_name: "recipe-gdpr-audit-logging"
    runtime: "python3.11"
    handler: "lambda_function.lambda_handler"
    memory_size: 128  # Minimum memory for cost optimization
    timeout: 60  # 1 minute
    
    environment_variables:
      DATABASE_HOST: "${rds_postgresql.endpoint}"
      DATABASE_NAME: "recipe_book"
      DATABASE_USER: "postgres"
      DATABASE_PASSWORD: "${database_password}"
      
    iam_role: "arn:aws:iam::account:role/lambda-basic-execution-role"
```

**Free Tier Benefits of Lambda**:
- **Free Tier**: 1 million requests per month and 400,000 GB-seconds included
- **Minimal Memory**: 128MB memory allocation minimizes GB-seconds usage
- **No VPC**: Eliminates NAT Gateway costs by accessing RDS publicly
- **Basic IAM Role**: Uses simple execution role without complex permissions

## Networking Infrastructure

### Default VPC Configuration

**Service**: AWS Default VPC with Public Subnets
**Configuration**: Use default VPC to eliminate custom networking costs

```yaml
# Default VPC Configuration (Free Tier Optimized)
vpc_configuration:
  use_default_vpc: true
  
  default_vpc_features:
    cidr_block: "172.31.0.0/16"  # AWS default VPC CIDR
    dns_hostnames: true
    dns_support: true
    internet_gateway: true  # Included with default VPC
    
  default_subnets:
    # AWS provides default subnets in each AZ
    subnet_1a:
      cidr_block: "172.31.0.0/20"
      availability_zone: "us-east-1a"
      public: true
      
    subnet_1b:
      cidr_block: "172.31.16.0/20"
      availability_zone: "us-east-1b"
      public: true
      
  routing:
    default_route_table:
      routes:
        - destination: "0.0.0.0/0"
          target: "Internet Gateway"  # Free with default VPC
          
  # No NAT Gateway required - saves $32.40/month
  # No custom VPC costs
  # No additional subnet costs
```

**Free Tier Networking Benefits**:
- **Default VPC**: No additional VPC charges
- **Default Subnets**: No subnet creation charges
- **Internet Gateway**: Included free with default VPC
- **No NAT Gateway**: Eliminates $32.40/month cost
- **Public Access**: RDS and ElastiCache accessible via public subnets with security groups

### Security Groups Configuration

**Configuration**: Basic security groups using default VPC

```yaml
# Security Groups Configuration (Free Tier Optimized)
security_groups:
  database_sg:
    name: "recipe-database-sg"
    description: "Security group for RDS PostgreSQL database"
    vpc_id: "default"  # Use default VPC
    
    ingress_rules:
      - protocol: "tcp"
        from_port: 5432
        to_port: 5432
        cidr_blocks: ["0.0.0.0/0"]  # Open access (secure with strong password)
        description: "PostgreSQL access from internet"
        
    egress_rules:
      - protocol: "-1"
        from_port: 0
        to_port: 0
        cidr_blocks: ["0.0.0.0/0"]
        description: "All outbound traffic"
        
  cache_sg:
    name: "recipe-cache-sg"
    description: "Security group for ElastiCache Redis"
    vpc_id: "default"  # Use default VPC
    
    ingress_rules:
      - protocol: "tcp"
        from_port: 6379
        to_port: 6379
        cidr_blocks: ["0.0.0.0/0"]  # Open access (no auth token in free tier)
        description: "Redis access from internet"
        
    egress_rules:
      - protocol: "-1"
        from_port: 0
        to_port: 0
        cidr_blocks: ["0.0.0.0/0"]
        description: "All outbound traffic"
```

**Free Tier Security Considerations**:
- **Default VPC**: Uses default VPC security groups
- **Public Access**: Services accessible from internet with security group rules
- **Strong Authentication**: Rely on database passwords and application-level security
- **Simplified Rules**: Basic security group rules to minimize complexity

## Monitoring Infrastructure

### CloudWatch Monitoring Configuration (Free Tier)

**Service**: AWS CloudWatch Free Tier
**Configuration**: Basic monitoring within free tier limits

```yaml
# CloudWatch Configuration (Free Tier Optimized)
cloudwatch_monitoring:
  free_tier_limits:
    metrics: 10  # 10 custom metrics included
    logs_ingestion: 5  # 5 GB log ingestion per month
    logs_storage: 5  # 5 GB log storage per month
    api_requests: 1000000  # 1 million API requests per month
    
  log_groups:
    lambda_logs:
      name: "/aws/lambda/recipe-gdpr-functions"
      retention_in_days: 7  # Minimize storage costs
      
  # Limit custom metrics to stay within free tier
  essential_custom_metrics:
    database_metrics:
      namespace: "Recipe/Database"
      metrics:
        - metric_name: "ConnectionCount"
          unit: "Count"
          
        - metric_name: "CacheHitRate"
          unit: "Percent"
          
  # Basic dashboard with AWS native metrics (free)
  dashboards:
    basic_overview:
      name: "Recipe-Basic-Overview"
      widgets:
        - type: "metric"
          properties:
            metrics:
              - ["AWS/RDS", "CPUUtilization", "DBInstanceIdentifier", "recipe-db-primary"]
              - ["AWS/RDS", "DatabaseConnections", "DBInstanceIdentifier", "recipe-db-primary"]
              - ["AWS/ElastiCache", "CPUUtilization", "CacheClusterId", "recipe-cache-redis"]
            period: 300
            stat: "Average"
            region: "us-east-2"
            title: "Basic Performance Metrics"
            
  # Essential alarms only
  alarms:
    database_cpu_critical:
      alarm_name: "recipe-db-cpu-critical"
      comparison_operator: "GreaterThanThreshold"
      evaluation_periods: 2
      metric_name: "CPUUtilization"
      namespace: "AWS/RDS"
      period: 300
      statistic: "Average"
      threshold: 90  # Only critical alerts
      alarm_description: "Database CPU utilization is critically high"
      # Use SNS free tier for notifications
```

**Free Tier Monitoring Benefits**:
- **10 Custom Metrics**: Included in Free Tier
- **5GB Log Storage**: Included in Free Tier
- **Basic Dashboards**: AWS native metrics are free
- **Essential Alarms**: Only critical alerts to minimize costs
- **SNS Notifications**: 1,000 notifications per month included

## Backup and Disaster Recovery Infrastructure

### Backup Configuration

**Service**: AWS RDS Automated Backups with Cross-Region Copying
**Configuration**: Automated testing and cross-region copying

```yaml
# Backup Infrastructure
backup_configuration:
  rds_automated_backups:
    backup_retention_period: 7  # days
    backup_window: "03:00-04:00"
    copy_tags_to_snapshot: true
    delete_automated_backups: false
    
  manual_snapshots:
    frequency: "weekly"
    retention: 4  # weeks
    naming_convention: "recipe-db-manual-{timestamp}"
    
  cross_region_backup:
    enabled: true
    destination_region: "us-west-2"
    retention: 2  # weeks
    frequency: "weekly"
    
  backup_testing:
    test_frequency: "monthly"
    test_environment:
      instance_class: "db.t3.micro"
      storage_type: "gp2"
      allocated_storage: 20
      
    test_procedures:
      - restore_from_snapshot
      - data_integrity_check
      - performance_validation
      - cleanup_test_resources
      
  disaster_recovery:
    rto_target: 4  # hours
    rpo_target: 24  # hours
    
    recovery_procedures:
      automated:
        - identify_latest_backup
        - provision_new_instance
        - restore_from_backup
        - update_connection_strings
        
      manual:
        - validate_data_integrity
        - update_dns_records
        - notify_stakeholders
        - monitor_performance
```

**Backup Features**:
- **Automated Backups**: Daily automated snapshots
- **Cross-Region Protection**: Weekly cross-region backup copying
- **Testing Validation**: Monthly backup restoration testing
- **Disaster Recovery**: Documented recovery procedures

## Security Infrastructure

### AWS Security Services Integration

**Services**: Basic AWS security with KMS encryption and IAM roles
**Configuration**: Cost-optimized security with essential protections

```yaml
# Security Services Configuration
security_services:
  kms_encryption:
    rds_key:
      description: "KMS key for RDS encryption"
      key_usage: "ENCRYPT_DECRYPT"
      key_spec: "SYMMETRIC_DEFAULT"
      key_rotation_enabled: true
      
    elasticache_key:
      description: "KMS key for ElastiCache encryption"
      key_usage: "ENCRYPT_DECRYPT"
      key_spec: "SYMMETRIC_DEFAULT"
      key_rotation_enabled: true
      
  iam_roles:
    rds_monitoring_role:
      name: "recipe-rds-monitoring-role"
      assume_role_policy: "rds.amazonaws.com"
      managed_policies:
        - "arn:aws:iam::aws:policy/service-role/AmazonRDSEnhancedMonitoringRole"
        
    lambda_execution_role:
      name: "recipe-lambda-gdpr-execution-role"
      assume_role_policy: "lambda.amazonaws.com"
      managed_policies:
        - "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
      custom_policies:
        - name: "GDPRDataAccess"
          policy_document:
            Version: "2012-10-17"
            Statement:
              - Effect: "Allow"
                Action:
                  - "rds:DescribeDBInstances"
                  - "rds-db:connect"
                Resource: "*"
              - Effect: "Allow"
                Action:
                  - "elasticache:DescribeCacheClusters"
                Resource: "*"
                
  secrets_manager:
    database_credentials:
      name: "recipe-db-credentials"
      description: "Database credentials for RDS PostgreSQL"
      generate_secret_string:
        secret_string_template: '{"username": "postgres"}'
        generate_string_key: "password"
        exclude_characters: '"@/\'
        password_length: 32
        
  cloudtrail:
    enabled: false  # Cost optimization - basic security only
    
  config:
    enabled: false  # Cost optimization - basic security only
    
  guardduty:
    enabled: false  # Cost optimization - basic security only
```

**Security Features**:
- **Encryption**: KMS encryption for data at rest and in transit
- **IAM Roles**: Least privilege access control
- **Secrets Management**: Secure credential storage
- **Cost Optimization**: Basic security services only

## Cost Optimization Summary

### Monthly Cost Estimates (AWS Free Tier Optimized)

```yaml
# Estimated Monthly Costs (US East 2) - Free Tier Optimized
cost_estimates:
  rds_postgresql:
    instance: "$0.00"     # 750 hours db.t3.micro included in Free Tier
    storage: "$0.00"      # 20GB gp2 included in Free Tier
    backup: "$0.00"       # Automated backups included in Free Tier
    total: "$0.00"
    
  elasticache_redis:
    instance: "$0.00"     # 750 hours cache.t2.micro included in Free Tier
    total: "$0.00"
    
  lambda_functions:
    requests: "$0.00"     # 1M requests included in Free Tier
    compute: "$0.00"      # 400,000 GB-seconds included in Free Tier
    total: "$0.00"
    
  cloudwatch:
    metrics: "$0.00"      # 10 custom metrics included in Free Tier
    logs: "$0.00"         # 5GB log ingestion/storage included in Free Tier
    total: "$0.00"
    
  data_transfer:
    outbound: "$0.00"     # 15GB data transfer out included in Free Tier
    total: "$0.00"
    
  total_monthly_cost: "$0.00"
  
  # After Free Tier expires (12 months):
  post_free_tier_costs:
    rds_postgresql: "$13.87"    # db.t3.micro + 20GB storage
    elasticache_redis: "$11.52" # cache.t2.micro
    lambda_functions: "$2.00"   # Estimated usage
    cloudwatch: "$3.00"         # Basic monitoring
    total_post_free_tier: "$30.39"
```

**Free Tier Optimization Benefits**:
- **12 Months Free**: Complete infrastructure free for first 12 months
- **No Networking Costs**: Eliminated NAT Gateway ($32.40/month savings)
- **No RDS Proxy**: Eliminated managed service ($10.80/month savings)
- **Minimal Monitoring**: Stay within free tier limits
- **Post-Free Tier**: Only $30.39/month after free tier expires

This infrastructure design provides a **cost-effective**, **scalable**, and **secure** foundation for the Database Unit while maintaining the flexibility to grow as the application scales.
