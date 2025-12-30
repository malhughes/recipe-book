# Unit of Work Dependencies

## Dependency Matrix

| Unit | Depends On | Dependency Type | Integration Pattern | Critical Path |
|------|------------|----------------|-------------------|---------------|
| Frontend Unit | Backend Unit | Runtime | REST API calls | No |
| Frontend Unit | Infrastructure Unit | Deployment | Vercel deployment | No |
| Backend Unit | Database Unit | Runtime | Database connections | Yes |
| Backend Unit | AI Processing Units | Runtime | Async API calls | No |
| Backend Unit | Infrastructure Unit | Deployment | AWS ECS deployment | Yes |
| Embedding Unit | Database Unit | Runtime | Vector storage | Yes |
| Embedding Unit | Infrastructure Unit | Deployment | AWS ECS deployment | Yes |
| Recommendation Unit | Database Unit | Runtime | Cache and data access | Yes |
| Recommendation Unit | Embedding Unit | Runtime | Similarity queries | No |
| Recommendation Unit | Infrastructure Unit | Deployment | AWS ECS deployment | Yes |
| Parsing Unit | Infrastructure Unit | Deployment | AWS ECS deployment | Yes |
| Database Unit | Infrastructure Unit | Deployment | AWS RDS/ElastiCache | Yes |

## Dependency Analysis

### Critical Path Dependencies
**Critical Path**: Infrastructure Unit → Database Unit → Backend Unit → AI Processing Units

1. **Infrastructure Unit** must be established first to provide:
   - AWS infrastructure (ECS, RDS, ElastiCache)
   - Networking and security configurations
   - CI/CD pipelines for deployment

2. **Database Unit** depends on Infrastructure Unit for:
   - AWS RDS PostgreSQL instance
   - AWS ElastiCache Redis instance
   - Network connectivity and security groups

3. **Backend Unit** depends on Database Unit for:
   - Database connections and data persistence
   - Redis caching infrastructure
   - Data migration and schema setup

4. **AI Processing Units** depend on both Database Unit and Infrastructure Unit for:
   - Vector storage (pgvector) and caching
   - Container deployment infrastructure
   - Service discovery and networking

### Non-Critical Dependencies
- **Frontend Unit** can be developed in parallel with minimal dependencies
- **AI Processing Units** can be developed independently of each other
- Integration testing requires all units to be functional

## Integration Patterns

### Frontend ↔ Backend Integration
**Pattern**: RESTful API with JSON communication
**Authentication**: JWT tokens from Auth0
**Error Handling**: Standardized error responses with shadcn/ui Toast notifications
**State Management**: React Query for caching and synchronization

```typescript
// Frontend API Integration Pattern
const useRecipes = (userId: string) => {
  return useQuery({
    queryKey: ['recipes', userId],
    queryFn: () => apiClient.get(`/api/recipes?user_id=${userId}`),
    staleTime: 5 * 60 * 1000,
  });
};
```

### Backend ↔ Database Integration
**Pattern**: SQLAlchemy ORM with async database operations
**Connection Management**: Connection pooling with automatic retry
**Migrations**: Alembic for database schema management
**Caching**: Redis integration for performance optimization

```python
# Backend Database Integration Pattern
class RecipeRepository:
    async def create_recipe(self, recipe_data: RecipeCreate) -> Recipe:
        async with self.db_session() as session:
            recipe = Recipe(**recipe_data.dict())
            session.add(recipe)
            await session.commit()
            await session.refresh(recipe)
            return recipe
```

### Backend ↔ AI Processing Integration
**Pattern**: Asynchronous API calls with task queues
**Communication**: HTTP APIs with JSON payloads
**Error Handling**: Circuit breaker pattern with fallback strategies
**Cost Control**: Batching and rate limiting for OpenAI API calls

```python
# Backend AI Integration Pattern
class AIOrchestrationService:
    async def enrich_recipe(self, recipe_id: str) -> None:
        # Parallel AI processing
        embedding_task = self.embedding_client.generate_embedding(recipe_id)
        parsing_task = self.parsing_client.parse_ingredients(recipe_id)
        
        # Wait for completion with timeout
        await asyncio.gather(embedding_task, parsing_task, timeout=30)
```

### AI Processing ↔ Database Integration
**Pattern**: Direct database access for vector operations
**Vector Storage**: pgvector for embedding storage and similarity search
**Caching**: Redis for computed results and intermediate data
**Batch Operations**: Bulk operations for performance optimization

```python
# AI Database Integration Pattern
class EmbeddingService:
    async def store_embedding(self, recipe_id: str, embedding: np.ndarray):
        async with self.db_session() as session:
            await session.execute(
                text("UPDATE recipes SET embedding = :embedding WHERE id = :id"),
                {"embedding": embedding.tolist(), "id": recipe_id}
            )
            await session.commit()
```

## Development Sequence and Coordination

### Phase 1: Foundation Setup (Week 1)
**Parallel Development**:
- **Infrastructure Unit**: AWS infrastructure setup, CI/CD pipelines
- **Database Unit**: PostgreSQL and Redis setup, initial schema
- **Backend Unit**: Project structure, authentication setup

**Coordination Points**:
- Infrastructure team provides database connection strings
- Database team provides schema and migration scripts
- Backend team validates database connectivity

### Phase 2: Core Development (Week 2-3)
**Sequential Dependencies**:
1. **Database Unit**: Complete schema, migrations, and data access layer
2. **Backend Unit**: Core business logic, API endpoints
3. **Frontend Unit**: UI components, API integration

**Coordination Points**:
- Database team provides final schema and repository interfaces
- Backend team provides API documentation and test endpoints
- Frontend team validates API contracts and error handling

### Phase 3: AI Integration (Week 3-4)
**Parallel Development**:
- **Embedding Unit**: Vector generation and storage
- **Parsing Unit**: Ingredient parsing and metadata extraction
- **Recommendation Unit**: Taste profile and recommendation algorithms

**Coordination Points**:
- AI teams coordinate on shared data models and interfaces
- Backend team integrates AI service calls
- Database team optimizes vector queries and caching

### Phase 4: Integration and Testing (Week 4-5)
**Integration Testing**:
- Unit-to-unit integration testing
- End-to-end workflow testing
- Performance and load testing
- Security and compliance testing

**Coordination Points**:
- All teams participate in integration testing
- Infrastructure team monitors performance and scaling
- Frontend team validates user experience and error handling

## Communication Protocols

### Inter-Unit Communication Standards
- **API Contracts**: OpenAPI specifications for all service interfaces
- **Data Models**: Shared Pydantic models for data validation
- **Error Handling**: Standardized error codes and messages
- **Logging**: Structured logging with correlation IDs for tracing

### Development Coordination
- **Daily Standups**: Cross-unit coordination and dependency updates
- **Integration Reviews**: Weekly reviews of integration points and contracts
- **Shared Documentation**: Centralized API documentation and data models
- **Testing Coordination**: Shared testing utilities and integration test suites

## Risk Mitigation

### Dependency Risks
- **Database Bottleneck**: Database Unit delays affect all other units
  - *Mitigation*: Early database setup, mock data for development
- **Infrastructure Dependencies**: AWS setup delays affect deployment
  - *Mitigation*: Local development environments, infrastructure automation
- **AI Service Integration**: Complex AI integration may cause delays
  - *Mitigation*: Fallback strategies, gradual AI feature rollout

### Integration Risks
- **API Contract Changes**: Breaking changes affect multiple units
  - *Mitigation*: API versioning, contract testing, change management
- **Performance Issues**: Integration performance may not meet targets
  - *Mitigation*: Early performance testing, caching strategies, optimization
- **Security Vulnerabilities**: Cross-unit security issues
  - *Mitigation*: Security reviews, penetration testing, compliance validation

## Success Metrics

### Integration Success Criteria
- All API endpoints respond within SLA (< 500ms for non-AI operations)
- AI recommendations generated within 1-3 seconds
- Zero data loss during normal operations
- 99.9% uptime for core functionality
- All security and privacy requirements met

### Development Coordination Success
- No unit blocks another unit for more than 1 day
- Integration issues resolved within 24 hours
- All units meet their individual success criteria
- End-to-end user workflows function correctly
- Performance targets achieved in production environment
