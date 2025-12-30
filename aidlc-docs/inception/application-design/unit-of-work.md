# Units of Work Definition

## Unit Organization Strategy

Based on the decomposition decisions, the system is organized using a **technology-based unit structure** with specialized AI processing units. This approach aligns with team expertise and enables parallel development across different technology stacks.

## Core Units of Work

### 1. Frontend Unit (Next.js + shadcn/ui)
**Technology Stack**: Next.js, TypeScript, shadcn/ui, Tailwind CSS, React Query
**Team Alignment**: Frontend Development Team
**Deployment Target**: Vercel

#### Responsibilities
- **User Interface**: Complete user interface using shadcn/ui components
- **Authentication UI**: Login/logout interfaces with Auth0 integration
- **Recipe Management UI**: Recipe creation, editing, viewing, and management interfaces
- **Search & Discovery UI**: Search interface with advanced filtering and results display
- **Recommendations UI**: AI-powered recommendation display with user feedback
- **State Management**: Client-side state management and API integration
- **Responsive Design**: Mobile-first responsive design implementation

#### Key Components
- Recipe Management Module (Form, List, Detail components)
- Search Module (Command, Popover, Filter components)
- Recommendations Module (Card, Skeleton, Sheet components)
- Authentication Module (Button, Avatar, DropdownMenu components)
- Shared UI Components (shadcn/ui component library)
- Data Services Layer (API clients, caching, error handling)

#### Integration Points
- REST API calls to Backend Unit
- Auth0 authentication service integration
- Real-time updates for recommendations and search results

### 2. Backend Unit (FastAPI Modular Monolith)
**Technology Stack**: FastAPI, Python, SQLAlchemy, Pydantic, AsyncIO
**Team Alignment**: Backend Development Team
**Deployment Target**: AWS ECS/Fargate

#### Responsibilities
- **Core Business Logic**: Recipe management, user management, search orchestration
- **API Layer**: RESTful API endpoints for all frontend operations
- **Service Orchestration**: Coordinate between different service domains
- **Authentication**: JWT token validation and user session management
- **Data Validation**: Input validation and business rule enforcement
- **Integration Management**: Coordinate with AI Processing Units and Database Unit

#### Key Service Domains
- **Recipe Domain**: Recipe CRUD, validation, metadata management
- **User Domain**: User profiles, preferences, GDPR compliance
- **Search Domain**: Hybrid search coordination, result ranking
- **Integration Services**: Auth0 integration, cache management, event handling

#### Integration Points
- Database Unit for data persistence
- AI Processing Units for enrichment and recommendations
- Infrastructure Unit for deployment and monitoring
- Authentication service (Auth0) integration

### 3. AI Processing Units

#### 3.1 Embedding Unit
**Technology Stack**: Python, OpenAI SDK, NumPy, pgvector
**Team Alignment**: AI/ML Development Team
**Deployment Target**: AWS ECS/Fargate (separate service)

##### Responsibilities
- **Embedding Generation**: Create semantic embeddings for recipes using OpenAI
- **Vector Storage**: Store and index embeddings in pgvector
- **Similarity Search**: Perform vector similarity searches for recipe matching
- **Batch Processing**: Efficient batch processing for cost optimization
- **Embedding Management**: Update and maintain embedding indexes

##### Key Components
- OpenAI Embedding API integration
- Vector storage and retrieval services
- Similarity calculation algorithms
- Batch processing workflows
- Cost monitoring and optimization

#### 3.2 Recommendation Unit
**Technology Stack**: Python, OpenAI SDK, Scikit-learn, Redis
**Team Alignment**: AI/ML Development Team
**Deployment Target**: AWS ECS/Fargate (separate service)

##### Responsibilities
- **Taste Profile Management**: Build and maintain user taste profiles
- **Recommendation Generation**: Generate personalized recipe recommendations
- **Ranking Algorithms**: Rank recommendations based on user preferences
- **Feedback Processing**: Process user feedback to improve recommendations
- **Recommendation Caching**: Cache recommendations for performance

##### Key Components
- Taste profile analysis algorithms
- Multi-factor recommendation engine
- User feedback processing
- Recommendation ranking and filtering
- Cache management for recommendations

#### 3.3 Parsing Unit
**Technology Stack**: Python, OpenAI SDK, spaCy, Regular Expressions
**Team Alignment**: AI/ML Development Team
**Deployment Target**: AWS ECS/Fargate (separate service)

##### Responsibilities
- **Ingredient Parsing**: Parse free-form ingredient text into structured data
- **Metadata Extraction**: Extract cooking time, servings, difficulty from recipes
- **Content Enrichment**: Enhance recipe data with additional metadata
- **Validation**: Validate parsed results for accuracy and completeness
- **Standardization**: Standardize ingredient names and measurements

##### Key Components
- Natural language processing for ingredient parsing
- OpenAI API integration for content understanding
- Structured data extraction algorithms
- Validation and quality assurance
- Data standardization services

### 4. Database Unit
**Technology Stack**: PostgreSQL, pgvector, Redis, Database Migrations
**Team Alignment**: Backend/DevOps Team
**Deployment Target**: AWS RDS, AWS ElastiCache

#### Responsibilities
- **Data Persistence**: All application data storage and retrieval
- **Vector Storage**: Semantic embeddings storage with pgvector
- **Caching Layer**: Redis-based caching for performance optimization
- **Data Migrations**: Database schema management and migrations
- **Data Backup**: Automated backup and recovery procedures
- **Performance Optimization**: Query optimization and indexing strategies

#### Key Components
- **PostgreSQL Database**: Primary data storage with pgvector extension
- **Redis Cache**: Multi-level caching with TTL strategies
- **Migration System**: Database schema versioning and updates
- **Backup System**: Automated backup and point-in-time recovery
- **Monitoring**: Database performance monitoring and alerting

#### Data Models
- User profiles and authentication data
- Recipe data with metadata and embeddings
- Taste profiles and user preferences
- Search indexes and cached results
- System configuration and settings

### 5. Infrastructure Unit
**Technology Stack**: Terraform, Docker, AWS Services, GitHub Actions
**Team Alignment**: DevOps/Platform Team
**Deployment Target**: AWS (ECS, RDS, ElastiCache), Vercel

#### Responsibilities
- **Infrastructure as Code**: Terraform configurations for all AWS resources
- **Container Management**: Docker images and container orchestration
- **CI/CD Pipelines**: Automated build, test, and deployment pipelines
- **Monitoring & Logging**: Comprehensive monitoring and log aggregation
- **Security**: Security policies, AWS Secrets Manager for API keys, and compliance
- **Scaling**: Auto-scaling policies and resource optimization

#### Key Components
- **AWS Infrastructure**: ECS clusters, RDS instances, ElastiCache, VPC, security groups
- **Vercel Configuration**: Frontend deployment and domain management
- **CI/CD Pipelines**: GitHub Actions workflows for automated deployment
- **Monitoring Stack**: CloudWatch, application metrics, alerting
- **Security Management**: IAM roles, AWS Secrets Manager for API keys (including OPENAI_API_KEY), security scanning

#### Infrastructure Services
- Container orchestration (AWS ECS/Fargate)
- Database hosting (AWS RDS PostgreSQL)
- Caching infrastructure (AWS ElastiCache Redis)
- Load balancing and networking
- SSL/TLS certificate management
- Backup and disaster recovery
- Secure API key management (AWS Secrets Manager for OPENAI_API_KEY)

## Unit Development Sequence

### Phase 1: Foundation (Parallel Development)
1. **Infrastructure Unit**: Set up basic AWS infrastructure and CI/CD pipelines
2. **Database Unit**: Set up PostgreSQL with pgvector and Redis infrastructure
3. **Backend Unit**: Core API structure and authentication integration

### Phase 2: Core Functionality (Sequential Development)
1. **Backend Unit**: Complete core business logic (Recipe, User domains)
2. **Frontend Unit**: Basic UI components and recipe management features
3. **Embedding Unit**: Basic embedding generation and storage

### Phase 3: AI Integration (Parallel Development)
1. **Parsing Unit**: Ingredient parsing and metadata extraction
2. **Recommendation Unit**: Taste profile and recommendation generation
3. **Frontend Unit**: AI-powered features integration

### Phase 4: Integration and Optimization
1. **All Units**: Integration testing and performance optimization
2. **Infrastructure Unit**: Production deployment and monitoring setup
3. **All Units**: End-to-end testing and user acceptance testing

## Success Criteria

### Individual Unit Criteria
- **Frontend Unit**: Complete UI with all features, responsive design, accessibility compliance
- **Backend Unit**: All API endpoints functional, proper error handling, authentication working
- **AI Processing Units**: AI features working with cost optimization, proper error handling
- **Database Unit**: All data operations functional, backup/recovery tested, performance optimized
- **Infrastructure Unit**: Production deployment successful, monitoring operational, security validated

### Integration Criteria
- All units communicate properly through defined interfaces
- End-to-end user workflows function correctly
- Performance targets met (1-3 second recommendations)
- Security and privacy requirements satisfied
- Cost optimization targets achieved
