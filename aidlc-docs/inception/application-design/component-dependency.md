# Component Dependencies and Communication

## Dependency Matrix

### Frontend Component Dependencies

| Component | Depends On | Communication Pattern | Data Flow |
|-----------|------------|----------------------|-----------|
| Recipe Management | Data Services, UI Components, Auth | API calls, Props | User Input → API → State Update |
| Search Module | Data Services, UI Components | API calls, Props | Query → API → Results Display |
| Recommendations | Data Services, UI Components, Auth | API calls, Props | User Context → API → Recommendations |
| Authentication | Auth0 SDK, Data Services | OAuth flow, API calls | Auth Flow → Token → Session State |
| UI Components | None (leaf components) | Props, Events | Props In → Events Out |
| Data Services | Backend APIs, Cache | HTTP requests, Local storage | Request → Response → Cache |

### Backend Service Dependencies

| Service Domain | Depends On | Communication Pattern | Data Flow |
|----------------|------------|----------------------|-----------|
| Recipe Orchestration | Recipe Service, AI Service, Cache | Direct calls, Events | Request → Validation → Storage → AI Queue |
| User Management | User Service, Taste Profile, Auth | Direct calls, Events | User Action → Profile Update → Cache Refresh |
| AI Orchestration | AI Processors, Task Queue, Cache | Async tasks, Direct calls | Recipe → Queue → Processing → Storage |
| Search Orchestration | Search Service, Vector Search, Cache | Parallel calls, Aggregation | Query → Multi-search → Merge → Rank |
| Authentication | Auth0, User Service, Cache | OAuth, JWT validation | Token → Validation → User Context |

### AI Processing Dependencies

| AI Component | Depends On | Communication Pattern | Data Flow |
|--------------|------------|----------------------|-----------|
| Core AI Service | OpenAI API, Processors, Monitor | API calls, Coordination | Request → Route → Process → Monitor |
| Embedding Processor | OpenAI API, Vector Store | API calls, Database | Text → Embedding → Storage |
| Recommendation Processor | Taste Profile, Vector Search, Cache | Direct calls, Queries | Profile → Similarity → Ranking → Cache |
| Parsing Processor | OpenAI API, Validation | API calls, Rules | Text → Parse → Validate → Structure |

## Communication Patterns

### Frontend-Backend Communication

#### API Communication Pattern (with shadcn/ui Toast)
```typescript
import { useToast } from "@/components/ui/use-toast"

// Frontend API Client Pattern with shadcn/ui feedback
class APIClient {
  private baseURL: string;
  private authToken: string;
  private toast: ReturnType<typeof useToast>['toast'];

  async makeRequest<T>(endpoint: string, options: RequestOptions): Promise<T> {
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...options,
        headers: {
          'Authorization': `Bearer ${this.authToken}`,
          'Content-Type': 'application/json',
          ...options.headers
        }
      });
      
      if (!response.ok) {
        const error = await response.json();
        this.toast({
          title: "Error",
          description: error.message || "An error occurred",
          variant: "destructive",
        });
        throw new APIError(response.status, error);
      }
      
      return response.json();
    } catch (error) {
      this.toast({
        title: "Network Error",
        description: "Please check your connection and try again",
        variant: "destructive",
      });
      throw error;
    }
  }
}
```

#### State Management Pattern (with shadcn/ui Loading States)
```typescript
import { Skeleton } from "@/components/ui/skeleton"
import { Alert, AlertDescription } from "@/components/ui/alert"

// Frontend State Management with React Query and shadcn/ui
const useRecipes = (userId: string) => {
  return useQuery({
    queryKey: ['recipes', userId],
    queryFn: () => apiClient.getRecipes(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
};

// Usage with shadcn/ui components
const RecipeList = ({ userId }: { userId: string }) => {
  const { data: recipes, isLoading, error } = useRecipes(userId);

  if (isLoading) {
    return (
      <div className="space-y-4">
        {Array.from({ length: 3 }).map((_, i) => (
          <Card key={i}>
            <CardHeader>
              <Skeleton className="h-4 w-[250px]" />
              <Skeleton className="h-4 w-[200px]" />
            </CardHeader>
            <CardContent>
              <Skeleton className="h-4 w-full" />
            </CardContent>
          </Card>
        ))}
      </div>
    );
  }

  if (error) {
    return (
      <Alert variant="destructive">
        <AlertDescription>
          Failed to load recipes. Please try again.
        </AlertDescription>
      </Alert>
    );
  }

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
      {recipes?.map((recipe) => (
        <RecipeCard key={recipe.id} recipe={recipe} />
      ))}
    </div>
  );
};
```

### Backend Service Communication

#### Synchronous Service Communication
```python
# Direct service-to-service communication
class RecipeOrchestrationService:
    async def create_recipe(self, recipe_data: RecipeInput, user_id: str) -> Recipe:
        # Synchronous validation
        validation_result = await self.validation_service.validate(recipe_data)
        
        # Synchronous storage
        recipe = await self.recipe_service.create(recipe_data, user_id)
        
        # Asynchronous AI processing
        await self.ai_orchestration.enqueue_processing(recipe.id)
        
        return recipe
```

#### Asynchronous Task Communication
```python
# Task queue communication pattern
class TaskQueueService:
    async def enqueue_ai_processing(self, recipe_id: str) -> str:
        task_data = {
            'recipe_id': recipe_id,
            'processing_steps': ['embedding', 'parsing', 'enrichment']
        }
        
        task_id = await self.queue.enqueue(
            'ai_processing_workflow',
            task_data,
            priority='normal'
        )
        
        return task_id
```

#### Event-Driven Communication
```python
# Event publishing pattern
class EventPublisher:
    async def publish_recipe_created(self, recipe: Recipe):
        event = RecipeCreatedEvent(
            recipe_id=recipe.id,
            user_id=recipe.user_id,
            timestamp=datetime.utcnow()
        )
        
        await self.event_bus.publish('recipe.created', event)

# Event handling pattern
class TasteProfileEventHandler:
    async def handle_recipe_created(self, event: RecipeCreatedEvent):
        await self.taste_profile_service.update_from_recipe(
            event.user_id, 
            event.recipe_id
        )
```

### AI Service Communication

#### OpenAI API Integration Pattern
```python
class OpenAIClient:
    def __init__(self, api_key: str, cost_monitor: CostMonitor):
        self.client = openai.AsyncOpenAI(api_key=api_key)
        self.cost_monitor = cost_monitor
    
    async def generate_embedding(self, text: str) -> np.ndarray:
        # Cost tracking
        estimated_cost = self.cost_monitor.estimate_embedding_cost(text)
        await self.cost_monitor.check_budget(estimated_cost)
        
        # API call with retry logic
        response = await self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        
        # Track actual cost
        await self.cost_monitor.record_usage('embedding', response.usage)
        
        return np.array(response.data[0].embedding)
```

#### Batch Processing Pattern
```python
class BatchProcessor:
    async def process_batch(self, items: List[ProcessingItem], batch_size: int = 10):
        batches = [items[i:i+batch_size] for i in range(0, len(items), batch_size)]
        
        results = []
        for batch in batches:
            batch_results = await asyncio.gather(*[
                self.process_item(item) for item in batch
            ])
            results.extend(batch_results)
            
            # Rate limiting between batches
            await asyncio.sleep(1)
        
        return results
```

## Data Flow Diagrams

### Recipe Creation Flow
```
User Input (Frontend)
    ↓
Recipe Form Validation (Frontend)
    ↓
API Request (Frontend → Backend)
    ↓
Authentication Validation (Backend)
    ↓
Recipe Orchestration Service (Backend)
    ↓
┌─ Recipe Validation Service (Sync)
│   ↓
├─ Recipe Storage Service (Sync)
│   ↓
└─ AI Processing Queue (Async)
    ↓
AI Workflow Orchestration
    ↓
┌─ Embedding Generation (OpenAI)
├─ Ingredient Parsing (OpenAI)
└─ Metadata Extraction
    ↓
Recipe Update with AI Data
    ↓
Cache Invalidation Events
    ↓
Taste Profile Update (Async)
```

### Search and Recommendation Flow
```
Search Query (Frontend)
    ↓
Search Orchestration Service (Backend)
    ↓
┌─ Database Text Search (Parallel)
└─ Vector Similarity Search (Parallel)
    ↓
Result Merging and Ranking
    ↓
User Taste Profile Application
    ↓
Cache Storage
    ↓
Response to Frontend
    ↓
Results Display (Frontend)
```

### User Authentication Flow
```
User Login Action (Frontend)
    ↓
Auth0 OAuth Flow
    ↓
Authorization Code Exchange
    ↓
JWT Token Validation (Backend)
    ↓
User Profile Retrieval
    ↓
Session Cache Storage
    ↓
User Context Establishment
    ↓
Protected Resource Access
```

## Dependency Management Strategies

### Frontend Dependency Management

#### Component Composition Pattern (with shadcn/ui)
```typescript
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { useToast } from "@/components/ui/use-toast"

// Higher-order component for authentication with shadcn/ui
const withAuth = <P extends object>(Component: React.ComponentType<P>) => {
  return (props: P) => {
    const { user, isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return (
        <Card className="w-[350px] mx-auto mt-8">
          <CardHeader>
            <CardTitle>Authentication Required</CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-sm text-muted-foreground mb-4">
              Please log in to access this feature.
            </p>
            <Button onClick={() => window.location.href = '/login'} className="w-full">
              Log In
            </Button>
          </CardContent>
        </Card>
      );
    }
    
    return <Component {...props} user={user} />;
  };
};

// Usage
const ProtectedRecipeForm = withAuth(RecipeForm);
```

#### Service Injection Pattern (with shadcn/ui Theme)
```typescript
import { ThemeProvider } from "@/components/theme-provider"
import { Toaster } from "@/components/ui/toaster"

// Service provider pattern with shadcn/ui theme support
const ServiceProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const apiClient = new APIClient(process.env.NEXT_PUBLIC_API_URL);
  const recipeService = new RecipeService(apiClient);
  const searchService = new SearchService(apiClient);
  
  return (
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      <ServiceContext.Provider value={{ recipeService, searchService }}>
        {children}
        <Toaster />
      </ServiceContext.Provider>
    </ThemeProvider>
  );
};
```

### Backend Dependency Management

#### Dependency Injection Container
```python
class DIContainer:
    def __init__(self):
        self._services = {}
        self._singletons = {}
    
    def register(self, interface: Type, implementation: Type, singleton: bool = False):
        self._services[interface] = (implementation, singleton)
    
    def get(self, interface: Type):
        if interface in self._singletons:
            return self._singletons[interface]
        
        implementation, is_singleton = self._services[interface]
        instance = self._create_instance(implementation)
        
        if is_singleton:
            self._singletons[interface] = instance
        
        return instance
```

#### Service Factory Pattern
```python
class ServiceFactory:
    @staticmethod
    def create_recipe_orchestration_service() -> RecipeOrchestrationService:
        # Create dependencies
        recipe_service = ServiceFactory.create_recipe_service()
        ai_service = ServiceFactory.create_ai_orchestration_service()
        cache_service = ServiceFactory.create_cache_service()
        
        # Create and return service
        return RecipeOrchestrationService(
            recipe_service=recipe_service,
            ai_orchestration=ai_service,
            cache_service=cache_service
        )
```

## Error Handling and Resilience Patterns

### Circuit Breaker Pattern
```python
class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
    
    async def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise CircuitBreakerOpenError()
        
        try:
            result = await func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
```

### Retry Pattern with Exponential Backoff
```python
async def retry_with_backoff(func, max_retries: int = 3, base_delay: float = 1.0):
    for attempt in range(max_retries):
        try:
            return await func()
        except RetryableError as e:
            if attempt == max_retries - 1:
                raise
            
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            await asyncio.sleep(delay)
```

### Graceful Degradation Pattern
```python
class RecommendationService:
    async def get_recommendations(self, user_id: str) -> List[Recommendation]:
        try:
            # Try AI-powered recommendations
            return await self.ai_recommendation_service.get_recommendations(user_id)
        except AIServiceUnavailableError:
            # Fallback to simple popularity-based recommendations
            return await self.fallback_recommendation_service.get_popular_recipes()
        except Exception:
            # Last resort: return empty list with error message
            return []
```
