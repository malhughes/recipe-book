# Database Unit - Business Logic Model

## Data Access Patterns and Repository Interfaces

Based on the functional design decisions, the database unit implements comprehensive data access patterns optimized for CRUD operations, complex search queries, AI processing, and batch operations.

## Core Repository Interfaces

### User Repository Interface
**Purpose**: Handle all user-related data operations with GDPR compliance

```python
from abc import ABC, abstractmethod
from typing import Optional, List, Dict, Any
from datetime import datetime
from uuid import UUID

class UserRepositoryInterface(ABC):
    
    @abstractmethod
    async def create_user(self, user_data: UserCreateModel) -> UserModel:
        """Create new user with GDPR consent tracking"""
        pass
    
    @abstractmethod
    async def get_user_by_id(self, user_id: UUID) -> Optional[UserModel]:
        """Retrieve user by ID with active status check"""
        pass
    
    @abstractmethod
    async def get_user_by_auth0_id(self, auth0_user_id: str) -> Optional[UserModel]:
        """Retrieve user by Auth0 ID for authentication"""
        pass
    
    @abstractmethod
    async def update_user(self, user_id: UUID, updates: UserUpdateModel) -> UserModel:
        """Update user data with audit logging"""
        pass
    
    @abstractmethod
    async def update_consent(self, user_id: UUID, consent_version: str) -> UserModel:
        """Update GDPR consent with timestamp"""
        pass
    
    @abstractmethod
    async def request_data_export(self, user_id: UUID, export_format: str) -> UUID:
        """Create data export request for GDPR compliance"""
        pass
    
    @abstractmethod
    async def request_data_deletion(self, user_id: UUID) -> bool:
        """Mark user for data deletion (GDPR right to be forgotten)"""
        pass
    
    @abstractmethod
    async def get_user_data_export(self, user_id: UUID) -> Dict[str, Any]:
        """Generate complete user data export for GDPR"""
        pass
    
    @abstractmethod
    async def delete_user_data(self, user_id: UUID) -> bool:
        """Permanently delete all user data (GDPR compliance)"""
        pass
```

### Recipe Repository Interface
**Purpose**: Handle recipe CRUD operations with AI processing coordination

```python
class RecipeRepositoryInterface(ABC):
    
    @abstractmethod
    async def create_recipe(self, recipe_data: RecipeCreateModel) -> RecipeModel:
        """Create recipe and trigger AI processing pipeline"""
        pass
    
    @abstractmethod
    async def get_recipe_by_id(self, recipe_id: UUID, user_id: UUID) -> Optional[RecipeModel]:
        """Retrieve recipe with ownership validation"""
        pass
    
    @abstractmethod
    async def get_user_recipes(
        self, 
        user_id: UUID, 
        limit: int = 50, 
        offset: int = 0,
        filters: Optional[RecipeFilters] = None
    ) -> List[RecipeModel]:
        """Get user's recipes with pagination and filtering"""
        pass
    
    @abstractmethod
    async def update_recipe(self, recipe_id: UUID, user_id: UUID, updates: RecipeUpdateModel) -> RecipeModel:
        """Update recipe and trigger AI re-processing if content changed"""
        pass
    
    @abstractmethod
    async def delete_recipe(self, recipe_id: UUID, user_id: UUID) -> bool:
        """Delete recipe and all associated data (embeddings, parsed ingredients)"""
        pass
    
    @abstractmethod
    async def search_recipes(self, search_query: RecipeSearchQuery) -> RecipeSearchResults:
        """Full-text search across recipe titles and ingredients"""
        pass
    
    @abstractmethod
    async def get_recipes_for_ai_processing(self, limit: int = 100) -> List[RecipeModel]:
        """Get recipes pending AI processing for batch operations"""
        pass
    
    @abstractmethod
    async def update_ai_processing_status(
        self, 
        recipe_id: UUID, 
        status: AIProcessingStatus,
        error_message: Optional[str] = None
    ) -> bool:
        """Update AI processing status and error information"""
        pass
```

### Embedding Repository Interface
**Purpose**: Handle vector storage and similarity search operations

```python
import numpy as np

class EmbeddingRepositoryInterface(ABC):
    
    @abstractmethod
    async def store_embedding(
        self, 
        recipe_id: UUID, 
        embedding: np.ndarray,
        model: str,
        version: str,
        source_text_hash: str
    ) -> UUID:
        """Store recipe embedding with metadata"""
        pass
    
    @abstractmethod
    async def get_embedding(self, recipe_id: UUID) -> Optional[EmbeddingModel]:
        """Retrieve embedding for specific recipe"""
        pass
    
    @abstractmethod
    async def similarity_search(
        self, 
        query_embedding: np.ndarray, 
        limit: int = 20,
        user_id: Optional[UUID] = None,
        exclude_recipe_ids: Optional[List[UUID]] = None
    ) -> List[SimilarityMatch]:
        """Find similar recipes using vector similarity search"""
        pass
    
    @abstractmethod
    async def batch_similarity_search(
        self, 
        query_embeddings: List[np.ndarray], 
        limit: int = 20
    ) -> List[List[SimilarityMatch]]:
        """Batch similarity search for multiple embeddings"""
        pass
    
    @abstractmethod
    async def update_embedding(
        self, 
        recipe_id: UUID, 
        embedding: np.ndarray,
        source_text_hash: str
    ) -> bool:
        """Update existing embedding when recipe content changes"""
        pass
    
    @abstractmethod
    async def delete_embedding(self, recipe_id: UUID) -> bool:
        """Delete embedding when recipe is deleted"""
        pass
    
    @abstractmethod
    async def get_embeddings_for_reprocessing(
        self, 
        model: str, 
        version: str,
        limit: int = 1000
    ) -> List[UUID]:
        """Get recipe IDs that need embedding updates for new model versions"""
        pass
```

### Parsed Ingredients Repository Interface
**Purpose**: Handle structured ingredient data operations

```python
class ParsedIngredientsRepositoryInterface(ABC):
    
    @abstractmethod
    async def store_parsed_ingredients(
        self, 
        recipe_id: UUID, 
        ingredients: List[ParsedIngredientModel]
    ) -> List[UUID]:
        """Store parsed ingredients for a recipe"""
        pass
    
    @abstractmethod
    async def get_recipe_ingredients(self, recipe_id: UUID) -> List[ParsedIngredientModel]:
        """Get all parsed ingredients for a recipe"""
        pass
    
    @abstractmethod
    async def search_by_ingredient(
        self, 
        ingredient_name: str, 
        user_id: Optional[UUID] = None
    ) -> List[RecipeIngredientMatch]:
        """Find recipes containing specific ingredients"""
        pass
    
    @abstractmethod
    async def get_ingredient_statistics(self, user_id: UUID) -> IngredientStats:
        """Get ingredient usage statistics for taste profile calculation"""
        pass
    
    @abstractmethod
    async def update_parsed_ingredients(
        self, 
        recipe_id: UUID, 
        ingredients: List[ParsedIngredientModel]
    ) -> bool:
        """Update parsed ingredients when recipe is modified"""
        pass
    
    @abstractmethod
    async def delete_recipe_ingredients(self, recipe_id: UUID) -> bool:
        """Delete all ingredients for a recipe"""
        pass
```

### Taste Profile Repository Interface
**Purpose**: Handle user taste profile operations and calculations

```python
class TasteProfileRepositoryInterface(ABC):
    
    @abstractmethod
    async def create_taste_profile(self, user_id: UUID) -> TasteProfileModel:
        """Create initial empty taste profile for new user"""
        pass
    
    @abstractmethod
    async def get_taste_profile(self, user_id: UUID) -> Optional[TasteProfileModel]:
        """Retrieve user's current taste profile"""
        pass
    
    @abstractmethod
    async def update_taste_profile(
        self, 
        user_id: UUID, 
        profile_data: TasteProfileUpdateModel
    ) -> TasteProfileModel:
        """Update taste profile with new preference data"""
        pass
    
    @abstractmethod
    async def calculate_taste_profile(self, user_id: UUID) -> TasteProfileModel:
        """Recalculate taste profile based on user's recipe history"""
        pass
    
    @abstractmethod
    async def get_profiles_for_refresh(self, limit: int = 100) -> List[UUID]:
        """Get user IDs whose taste profiles need recalculation"""
        pass
    
    @abstractmethod
    async def delete_taste_profile(self, user_id: UUID) -> bool:
        """Delete taste profile (GDPR compliance)"""
        pass
```

## Caching Strategy and Implementation

### Smart Caching Architecture
**Purpose**: Implement multi-level caching with intelligent invalidation

```python
from enum import Enum
from typing import Union, Any

class CacheLevel(Enum):
    L1_APPLICATION = "l1_app"
    L2_REDIS = "l2_redis"
    L3_DATABASE = "l3_db"

class CacheStrategy:
    """Smart caching strategy with warming, invalidation, and monitoring"""
    
    def __init__(self, redis_client, metrics_collector):
        self.redis = redis_client
        self.metrics = metrics_collector
        self.cache_config = {
            # User data caching
            "user_profile": {"ttl": 3600, "level": CacheLevel.L2_REDIS},
            "user_recipes": {"ttl": 1800, "level": CacheLevel.L2_REDIS},
            "taste_profile": {"ttl": 7200, "level": CacheLevel.L2_REDIS},
            
            # Recipe data caching
            "recipe_details": {"ttl": 3600, "level": CacheLevel.L2_REDIS},
            "recipe_embeddings": {"ttl": 86400, "level": CacheLevel.L2_REDIS},
            "parsed_ingredients": {"ttl": 3600, "level": CacheLevel.L2_REDIS},
            
            # Search and recommendations
            "search_results": {"ttl": 900, "level": CacheLevel.L2_REDIS},
            "recommendations": {"ttl": 1800, "level": CacheLevel.L2_REDIS},
            "similarity_matches": {"ttl": 3600, "level": CacheLevel.L2_REDIS},
        }
    
    async def get_cached_data(self, cache_key: str, data_type: str) -> Optional[Any]:
        """Retrieve data from appropriate cache level"""
        config = self.cache_config.get(data_type, {})
        
        try:
            # Try Redis cache first
            cached_data = await self.redis.get(cache_key)
            if cached_data:
                self.metrics.increment("cache_hit", tags={"type": data_type, "level": "redis"})
                return json.loads(cached_data)
            
            self.metrics.increment("cache_miss", tags={"type": data_type, "level": "redis"})
            return None
            
        except Exception as e:
            self.metrics.increment("cache_error", tags={"type": data_type, "error": str(e)})
            return None
    
    async def set_cached_data(self, cache_key: str, data: Any, data_type: str) -> bool:
        """Store data in appropriate cache level with TTL"""
        config = self.cache_config.get(data_type, {"ttl": 3600})
        
        try:
            await self.redis.setex(
                cache_key, 
                config["ttl"], 
                json.dumps(data, default=str)
            )
            self.metrics.increment("cache_set", tags={"type": data_type})
            return True
            
        except Exception as e:
            self.metrics.increment("cache_set_error", tags={"type": data_type, "error": str(e)})
            return False
    
    async def invalidate_cache_pattern(self, pattern: str) -> int:
        """Invalidate cache entries matching pattern"""
        try:
            keys = await self.redis.keys(pattern)
            if keys:
                deleted_count = await self.redis.delete(*keys)
                self.metrics.increment("cache_invalidation", tags={"pattern": pattern, "count": deleted_count})
                return deleted_count
            return 0
            
        except Exception as e:
            self.metrics.increment("cache_invalidation_error", tags={"pattern": pattern, "error": str(e)})
            return 0
    
    async def warm_cache(self, user_id: UUID) -> bool:
        """Pre-warm cache with user's frequently accessed data"""
        try:
            # Warm user profile data
            user_cache_key = f"user_profile:{user_id}"
            recipes_cache_key = f"user_recipes:{user_id}"
            taste_profile_key = f"taste_profile:{user_id}"
            
            # This would be called by background tasks
            self.metrics.increment("cache_warming", tags={"user_id": str(user_id)})
            return True
            
        except Exception as e:
            self.metrics.increment("cache_warming_error", tags={"user_id": str(user_id), "error": str(e)})
            return False
```

### Cache Invalidation Events
**Purpose**: Event-driven cache invalidation for data consistency

```python
class CacheInvalidationHandler:
    """Handle cache invalidation based on data change events"""
    
    def __init__(self, cache_strategy: CacheStrategy):
        self.cache = cache_strategy
    
    async def on_user_updated(self, user_id: UUID) -> None:
        """Invalidate user-related caches when user data changes"""
        patterns = [
            f"user_profile:{user_id}",
            f"user_recipes:{user_id}*",
            f"taste_profile:{user_id}",
        ]
        
        for pattern in patterns:
            await self.cache.invalidate_cache_pattern(pattern)
    
    async def on_recipe_updated(self, recipe_id: UUID, user_id: UUID) -> None:
        """Invalidate recipe-related caches when recipe changes"""
        patterns = [
            f"recipe_details:{recipe_id}",
            f"user_recipes:{user_id}*",
            f"recommendations:{user_id}*",
            f"search_results:*",  # Broad invalidation for search
            f"parsed_ingredients:{recipe_id}",
        ]
        
        for pattern in patterns:
            await self.cache.invalidate_cache_pattern(pattern)
    
    async def on_taste_profile_updated(self, user_id: UUID) -> None:
        """Invalidate recommendation caches when taste profile changes"""
        patterns = [
            f"taste_profile:{user_id}",
            f"recommendations:{user_id}*",
            f"similarity_matches:{user_id}*",
        ]
        
        for pattern in patterns:
            await self.cache.invalidate_cache_pattern(pattern)
```

## Data Migration and Schema Management

### Migration System Architecture
**Purpose**: Comprehensive migration system with rollback capabilities and validation

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any
from enum import Enum

class MigrationStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"
    ROLLED_BACK = "rolled_back"

class Migration(ABC):
    """Base class for database migrations"""
    
    def __init__(self, version: str, description: str):
        self.version = version
        self.description = description
        self.requires_rollback = True
    
    @abstractmethod
    async def up(self, connection) -> bool:
        """Apply the migration"""
        pass
    
    @abstractmethod
    async def down(self, connection) -> bool:
        """Rollback the migration"""
        pass
    
    @abstractmethod
    async def validate(self, connection) -> bool:
        """Validate migration was applied correctly"""
        pass

class MigrationManager:
    """Manage database migrations with rollback and validation"""
    
    def __init__(self, db_connection, audit_logger):
        self.db = db_connection
        self.audit = audit_logger
        self.migrations: List[Migration] = []
    
    async def register_migration(self, migration: Migration) -> None:
        """Register a migration for execution"""
        self.migrations.append(migration)
        self.migrations.sort(key=lambda m: m.version)
    
    async def apply_migrations(self) -> Dict[str, Any]:
        """Apply all pending migrations with validation"""
        results = {"applied": [], "failed": [], "skipped": []}
        
        for migration in self.migrations:
            try:
                # Check if migration already applied
                if await self._is_migration_applied(migration.version):
                    results["skipped"].append(migration.version)
                    continue
                
                # Start transaction for migration
                async with self.db.begin() as transaction:
                    # Record migration start
                    await self._record_migration_start(migration)
                    
                    # Apply migration
                    success = await migration.up(transaction)
                    if not success:
                        await transaction.rollback()
                        results["failed"].append(migration.version)
                        continue
                    
                    # Validate migration
                    validation_success = await migration.validate(transaction)
                    if not validation_success:
                        await transaction.rollback()
                        results["failed"].append(migration.version)
                        continue
                    
                    # Record migration completion
                    await self._record_migration_completion(migration)
                    await transaction.commit()
                    
                    results["applied"].append(migration.version)
                    
            except Exception as e:
                await self.audit.log_error(f"Migration {migration.version} failed: {str(e)}")
                results["failed"].append(migration.version)
        
        return results
    
    async def rollback_migration(self, version: str) -> bool:
        """Rollback a specific migration"""
        migration = next((m for m in self.migrations if m.version == version), None)
        if not migration:
            return False
        
        try:
            async with self.db.begin() as transaction:
                success = await migration.down(transaction)
                if success:
                    await self._record_migration_rollback(migration)
                    await transaction.commit()
                    return True
                else:
                    await transaction.rollback()
                    return False
                    
        except Exception as e:
            await self.audit.log_error(f"Migration rollback {version} failed: {str(e)}")
            return False
```

## Connection Management and Resource Optimization

### Advanced Connection Pooling
**Purpose**: Connection pooling with monitoring, health checks, and performance optimization

```python
import asyncio
from contextlib import asynccontextmanager
from typing import AsyncGenerator

class DatabaseConnectionManager:
    """Advanced connection management with monitoring and optimization"""
    
    def __init__(self, database_url: str, pool_config: Dict[str, Any]):
        self.database_url = database_url
        self.pool_config = pool_config
        self.pool = None
        self.metrics = MetricsCollector()
        self.health_checker = DatabaseHealthChecker()
    
    async def initialize_pool(self) -> None:
        """Initialize connection pool with advanced configuration"""
        self.pool = await asyncpg.create_pool(
            self.database_url,
            min_size=self.pool_config.get("min_connections", 5),
            max_size=self.pool_config.get("max_connections", 20),
            max_queries=self.pool_config.get("max_queries_per_connection", 50000),
            max_inactive_connection_lifetime=self.pool_config.get("max_idle_time", 300),
            command_timeout=self.pool_config.get("command_timeout", 30),
            server_settings={
                "application_name": "recipe_book_app",
                "jit": "off",  # Disable JIT for consistent performance
            }
        )
        
        # Start background health monitoring
        asyncio.create_task(self._monitor_pool_health())
    
    @asynccontextmanager
    async def get_connection(self) -> AsyncGenerator:
        """Get database connection with monitoring and error handling"""
        connection = None
        start_time = time.time()
        
        try:
            # Get connection from pool
            connection = await self.pool.acquire()
            self.metrics.increment("connection_acquired")
            
            # Health check
            if not await self.health_checker.check_connection(connection):
                await self.pool.release(connection, discard=True)
                raise DatabaseConnectionError("Connection failed health check")
            
            yield connection
            
        except Exception as e:
            self.metrics.increment("connection_error", tags={"error": str(e)})
            if connection:
                await self.pool.release(connection, discard=True)
            raise
            
        finally:
            if connection:
                await self.pool.release(connection)
                
            # Record connection metrics
            duration = time.time() - start_time
            self.metrics.histogram("connection_duration", duration)
    
    async def _monitor_pool_health(self) -> None:
        """Background task to monitor pool health and performance"""
        while True:
            try:
                # Collect pool metrics
                pool_size = self.pool.get_size()
                idle_connections = self.pool.get_idle_size()
                
                self.metrics.gauge("pool_size", pool_size)
                self.metrics.gauge("idle_connections", idle_connections)
                self.metrics.gauge("active_connections", pool_size - idle_connections)
                
                # Check for connection leaks
                if idle_connections < 2 and pool_size > 15:
                    self.metrics.increment("potential_connection_leak")
                
                await asyncio.sleep(30)  # Check every 30 seconds
                
            except Exception as e:
                self.metrics.increment("health_monitor_error", tags={"error": str(e)})
                await asyncio.sleep(60)  # Back off on errors
```

## Performance Monitoring and Optimization

### Query Performance Monitoring
**Purpose**: Monitor and optimize database query performance

```python
class QueryPerformanceMonitor:
    """Monitor query performance and identify optimization opportunities"""
    
    def __init__(self, metrics_collector):
        self.metrics = metrics_collector
        self.slow_query_threshold = 1.0  # 1 second
        self.query_stats = {}
    
    async def monitor_query(self, query: str, params: Dict[str, Any], execution_time: float) -> None:
        """Monitor individual query performance"""
        query_hash = hashlib.md5(query.encode()).hexdigest()[:8]
        
        # Record query metrics
        self.metrics.histogram("query_duration", execution_time, tags={"query_hash": query_hash})
        
        # Track slow queries
        if execution_time > self.slow_query_threshold:
            self.metrics.increment("slow_query", tags={"query_hash": query_hash})
            await self._log_slow_query(query, params, execution_time)
        
        # Update query statistics
        if query_hash not in self.query_stats:
            self.query_stats[query_hash] = {
                "count": 0,
                "total_time": 0.0,
                "max_time": 0.0,
                "query": query[:200]  # Store truncated query for reference
            }
        
        stats = self.query_stats[query_hash]
        stats["count"] += 1
        stats["total_time"] += execution_time
        stats["max_time"] = max(stats["max_time"], execution_time)
    
    async def get_performance_report(self) -> Dict[str, Any]:
        """Generate query performance report"""
        report = {
            "total_queries": sum(stats["count"] for stats in self.query_stats.values()),
            "slow_queries": [],
            "most_frequent": [],
            "recommendations": []
        }
        
        # Identify slow queries
        for query_hash, stats in self.query_stats.items():
            avg_time = stats["total_time"] / stats["count"]
            if avg_time > self.slow_query_threshold:
                report["slow_queries"].append({
                    "query_hash": query_hash,
                    "avg_time": avg_time,
                    "max_time": stats["max_time"],
                    "count": stats["count"],
                    "query": stats["query"]
                })
        
        # Identify most frequent queries
        frequent_queries = sorted(
            self.query_stats.items(),
            key=lambda x: x[1]["count"],
            reverse=True
        )[:10]
        
        for query_hash, stats in frequent_queries:
            report["most_frequent"].append({
                "query_hash": query_hash,
                "count": stats["count"],
                "avg_time": stats["total_time"] / stats["count"],
                "query": stats["query"]
            })
        
        return report
```

This comprehensive business logic model establishes the foundation for all database operations, caching strategies, migration management, and performance optimization that will support the entire recipe book application.
