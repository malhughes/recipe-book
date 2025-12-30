# Component Methods and Interfaces

## Frontend Component Methods

### Recipe Management Module

#### RecipeFormComponent (shadcn/ui Form)
```typescript
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from "@/components/ui/form"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"

const recipeSchema = z.object({
  title: z.string().min(1, "Title is required"),
  ingredients: z.string().min(1, "Ingredients are required"),
  instructions: z.string().min(1, "Instructions are required"),
})

interface RecipeFormProps {
  recipe?: Recipe;
  onSubmit: (recipe: RecipeInput) => Promise<void>;
  onCancel: () => void;
}

// Key Methods
validateRecipeInput(input: RecipeInput): ValidationResult
parseIngredients(text: string): ParsedIngredient[]
handleFormSubmit(values: z.infer<typeof recipeSchema>): Promise<void>
resetForm(): void
```

#### RecipeListComponent (shadcn/ui Table + Card)
```typescript
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { Badge } from "@/components/ui/badge"
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from "@/components/ui/alert-dialog"

interface RecipeListProps {
  recipes: Recipe[];
  onRecipeSelect: (recipe: Recipe) => void;
  onRecipeDelete: (recipeId: string) => Promise<void>;
}

// Key Methods
filterRecipes(criteria: FilterCriteria): Recipe[]
sortRecipes(sortBy: SortOption): Recipe[]
handleRecipeAction(action: RecipeAction, recipeId: string): Promise<void>
loadMoreRecipes(): Promise<void>
renderRecipeCard(recipe: Recipe): JSX.Element
```

### Search Module

#### SearchComponent (shadcn/ui Command + Popover)
```typescript
import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList } from "@/components/ui/command"
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover"
import { Button } from "@/components/ui/button"
import { Badge } from "@/components/ui/badge"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"

interface SearchProps {
  onSearchResults: (results: SearchResult[]) => void;
  initialQuery?: string;
}

// Key Methods
executeSearch(query: SearchQuery): Promise<SearchResult[]>
buildSearchQuery(filters: SearchFilters): SearchQuery
handleSearchSubmit(query: string): Promise<void>
clearSearch(): void
renderAdvancedFilters(): JSX.Element
```

#### SearchResultsComponent (shadcn/ui Card + Badge)
```typescript
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Button } from "@/components/ui/button"
import { Separator } from "@/components/ui/separator"

interface SearchResultsProps {
  results: SearchResult[];
  onResultSelect: (result: SearchResult) => void;
}

// Key Methods
renderSearchResults(results: SearchResult[]): JSX.Element
highlightSearchTerms(text: string, terms: string[]): JSX.Element
handleResultInteraction(result: SearchResult): void
renderResultCard(result: SearchResult): JSX.Element
```

### Recommendations Module

#### RecommendationsComponent (shadcn/ui Card + Skeleton + Sheet)
```typescript
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { Badge } from "@/components/ui/badge"
import { Skeleton } from "@/components/ui/skeleton"
import { Sheet, SheetContent, SheetDescription, SheetHeader, SheetTitle, SheetTrigger } from "@/components/ui/sheet"
import { useToast } from "@/components/ui/use-toast"

interface RecommendationsProps {
  userId: string;
  onRecommendationSelect: (recipe: Recipe) => void;
}

// Key Methods
loadRecommendations(userId: string): Promise<Recommendation[]>
refreshRecommendations(): Promise<void>
handleRecommendationFeedback(recipeId: string, feedback: FeedbackType): Promise<void>
trackRecommendationView(recommendationId: string): void
renderRecommendationCard(recommendation: Recommendation): JSX.Element
renderLoadingSkeleton(): JSX.Element
```

### Authentication Module

#### AuthProvider (shadcn/ui Button + Avatar + DropdownMenu)
```typescript
import { Button } from "@/components/ui/button"
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar"
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from "@/components/ui/dropdown-menu"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"

interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
}

// Key Methods
initializeAuth(): Promise<void>
handleAuthCallback(code: string): Promise<User>
refreshAuthToken(): Promise<string>
validateSession(): Promise<boolean>
renderLoginCard(): JSX.Element
renderUserMenu(): JSX.Element
```

## Backend Component Methods

### Recipe Domain

#### RecipeService
```python
class RecipeService:
    def create_recipe(self, user_id: str, recipe_data: RecipeInput) -> Recipe
    def get_recipe(self, recipe_id: str, user_id: str) -> Optional[Recipe]
    def update_recipe(self, recipe_id: str, user_id: str, updates: RecipeUpdate) -> Recipe
    def delete_recipe(self, recipe_id: str, user_id: str) -> bool
    def list_user_recipes(self, user_id: str, pagination: Pagination) -> List[Recipe]
    def validate_recipe_data(self, recipe_data: RecipeInput) -> ValidationResult
```

#### RecipeRepository
```python
class RecipeRepository:
    def save(self, recipe: Recipe) -> Recipe
    def find_by_id(self, recipe_id: str) -> Optional[Recipe]
    def find_by_user_id(self, user_id: str, limit: int, offset: int) -> List[Recipe]
    def update(self, recipe_id: str, updates: dict) -> Recipe
    def delete(self, recipe_id: str) -> bool
    def search_recipes(self, query: SearchQuery) -> List[Recipe]
```

### User Domain

#### UserService
```python
class UserService:
    def create_user_profile(self, auth_user: AuthUser) -> UserProfile
    def get_user_profile(self, user_id: str) -> Optional[UserProfile]
    def update_user_profile(self, user_id: str, updates: ProfileUpdate) -> UserProfile
    def delete_user_data(self, user_id: str) -> bool
    def export_user_data(self, user_id: str) -> UserDataExport
    def get_user_preferences(self, user_id: str) -> UserPreferences
```

#### TasteProfileService
```python
class TasteProfileService:
    def get_taste_profile(self, user_id: str) -> TasteProfile
    def update_taste_profile(self, user_id: str, recipe_data: Recipe) -> TasteProfile
    def calculate_taste_similarity(self, profile: TasteProfile, recipe: Recipe) -> float
    def refresh_taste_profile(self, user_id: str) -> TasteProfile
    def cache_taste_profile(self, user_id: str, profile: TasteProfile) -> None
```

### AI Processing Domain

#### AIOrchestrationService
```python
class AIOrchestrationService:
    def process_recipe_async(self, recipe: Recipe) -> TaskResult
    def get_processing_status(self, task_id: str) -> ProcessingStatus
    def enrich_recipe_data(self, recipe: Recipe) -> EnrichedRecipe
    def generate_recommendations(self, user_id: str, count: int) -> List[Recommendation]
    def track_ai_usage(self, operation: str, cost: float) -> None
```

#### EmbeddingProcessor
```python
class EmbeddingProcessor:
    def generate_recipe_embedding(self, recipe: Recipe) -> np.ndarray
    def batch_generate_embeddings(self, recipes: List[Recipe]) -> List[np.ndarray]
    def store_embedding(self, recipe_id: str, embedding: np.ndarray) -> bool
    def find_similar_recipes(self, embedding: np.ndarray, limit: int) -> List[SimilarityMatch]
    def preprocess_recipe_text(self, recipe: Recipe) -> str
```

#### RecommendationProcessor
```python
class RecommendationProcessor:
    def generate_user_recommendations(self, user_id: str, count: int) -> List[Recommendation]
    def calculate_recipe_score(self, user_profile: TasteProfile, recipe: Recipe) -> float
    def filter_recommendations(self, recommendations: List[Recommendation], filters: RecommendationFilters) -> List[Recommendation]
    def rank_recommendations(self, recommendations: List[Recommendation]) -> List[Recommendation]
    def explain_recommendation(self, recommendation: Recommendation) -> RecommendationExplanation
```

#### IngredientParsingProcessor
```python
class IngredientParsingProcessor:
    def parse_ingredient_text(self, text: str) -> List[ParsedIngredient]
    def extract_quantities(self, ingredient_text: str) -> QuantityInfo
    def standardize_ingredient_names(self, ingredients: List[str]) -> List[str]
    def validate_parsed_ingredients(self, ingredients: List[ParsedIngredient]) -> ValidationResult
    def enrich_ingredient_data(self, ingredients: List[ParsedIngredient]) -> List[EnrichedIngredient]
```

### Search Domain

#### SearchService
```python
class SearchService:
    def search_recipes(self, query: SearchQuery, user_id: str) -> SearchResults
    def hybrid_search(self, text_query: str, vector_query: np.ndarray, filters: SearchFilters) -> List[Recipe]
    def build_search_filters(self, criteria: SearchCriteria) -> SearchFilters
    def rank_search_results(self, results: List[Recipe], query: SearchQuery) -> List[Recipe]
    def track_search_analytics(self, query: SearchQuery, results_count: int) -> None
```

#### VectorSearchService
```python
class VectorSearchService:
    def similarity_search(self, query_embedding: np.ndarray, limit: int) -> List[SimilarityMatch]
    def batch_similarity_search(self, embeddings: List[np.ndarray], limit: int) -> List[List[SimilarityMatch]]
    def index_recipe_embedding(self, recipe_id: str, embedding: np.ndarray) -> bool
    def remove_recipe_embedding(self, recipe_id: str) -> bool
    def optimize_vector_index() -> None
```

### Infrastructure Components

#### CacheService
```python
class CacheService:
    def get(self, key: str) -> Optional[Any]
    def set(self, key: str, value: Any, ttl: int) -> bool
    def delete(self, key: str) -> bool
    def get_multi(self, keys: List[str]) -> Dict[str, Any]
    def set_multi(self, data: Dict[str, Any], ttl: int) -> bool
    def invalidate_pattern(self, pattern: str) -> int
```

#### TaskQueueService
```python
class TaskQueueService:
    def enqueue_task(self, task_name: str, payload: dict, delay: int = 0) -> str
    def get_task_status(self, task_id: str) -> TaskStatus
    def retry_failed_task(self, task_id: str) -> bool
    def cancel_task(self, task_id: str) -> bool
    def get_queue_stats() -> QueueStats
```

#### AuthenticationService
```python
class AuthenticationService:
    def validate_jwt_token(self, token: str) -> Optional[UserClaims]
    def get_user_from_token(self, token: str) -> Optional[User]
    def refresh_user_session(self, refresh_token: str) -> AuthTokens
    def revoke_user_session(self, user_id: str) -> bool
    def check_user_permissions(self, user_id: str, resource: str, action: str) -> bool
```

## API Endpoint Methods

### Recipe API Endpoints
```python
# Recipe CRUD operations
POST /api/recipes - create_recipe(recipe_data: RecipeInput) -> Recipe
GET /api/recipes/{recipe_id} - get_recipe(recipe_id: str) -> Recipe
PUT /api/recipes/{recipe_id} - update_recipe(recipe_id: str, updates: RecipeUpdate) -> Recipe
DELETE /api/recipes/{recipe_id} - delete_recipe(recipe_id: str) -> bool
GET /api/recipes - list_recipes(pagination: Pagination, filters: RecipeFilters) -> List[Recipe]
```

### Search API Endpoints
```python
# Search operations
GET /api/search - search_recipes(query: str, filters: SearchFilters) -> SearchResults
POST /api/search/advanced - advanced_search(criteria: SearchCriteria) -> SearchResults
```

### Recommendations API Endpoints
```python
# Recommendation operations
GET /api/recommendations - get_recommendations(count: int, filters: RecommendationFilters) -> List[Recommendation]
POST /api/recommendations/feedback - submit_feedback(recommendation_id: str, feedback: FeedbackData) -> bool
```

### User API Endpoints
```python
# User management operations
GET /api/user/profile - get_user_profile() -> UserProfile
PUT /api/user/profile - update_user_profile(updates: ProfileUpdate) -> UserProfile
DELETE /api/user/data - delete_user_data() -> bool
GET /api/user/export - export_user_data() -> UserDataExport
```

**Note**: Detailed business logic implementation will be defined in the Functional Design phase (per-unit, CONSTRUCTION phase).
