# Application Components

## Frontend Components (Next.js + shadcn/ui)

### shadcn/ui Setup and Configuration

#### Core Dependencies
- **Tailwind CSS**: Utility-first CSS framework for styling
- **shadcn/ui**: Copy-paste React components built on Radix UI primitives
- **Radix UI**: Unstyled, accessible components as foundation
- **class-variance-authority (cva)**: For component variant management
- **clsx**: For conditional className handling
- **tailwind-merge**: For merging Tailwind classes safely

#### Theme Configuration
```typescript
// tailwind.config.js
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: 0 },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: 0 },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### Core Feature Modules

#### Recipe Management Module
- **Purpose**: Handle all recipe-related user interactions using shadcn/ui components
- **Technology Stack**: shadcn/ui Form, Input, Textarea, Button, Card, Dialog components
- **Responsibilities**:
  - Recipe creation and editing forms with shadcn/ui Form components
  - Recipe display using Card, Badge, and Typography components
  - Recipe list management with Table and Pagination components
  - Recipe deletion confirmation using AlertDialog component
- **Key Interfaces**: 
  - Recipe CRUD operations with form validation using react-hook-form + zod
  - File upload handling with shadcn/ui Input (file type)
  - Toast notifications for user feedback

#### Search Module
- **Purpose**: Provide comprehensive search functionality with shadcn/ui components
- **Technology Stack**: shadcn/ui Input, Command, Popover, Select, Badge components
- **Responsibilities**:
  - Search interface using Command component for advanced search
  - Search results display with Card and Badge components
  - Search filters using Select, Checkbox, and RadioGroup components
  - Search history with Command component and local storage
- **Key Interfaces**: 
  - Search API integration with debounced input
  - Advanced filtering with Popover and form components
  - Search result highlighting and pagination

#### Recommendations Module
- **Purpose**: Display and manage AI-powered recipe recommendations using shadcn/ui
- **Technology Stack**: shadcn/ui Card, Button, Badge, Skeleton, Sheet components
- **Responsibilities**:
  - Recommendation display using Card components with images and metadata
  - Recommendation interaction with Button and Sheet components for details
  - Loading states using Skeleton components during AI processing
  - Recommendation feedback using Button variants and Toast notifications
- **Key Interfaces**: 
  - AI service integration with loading and error states
  - Recommendation explanation display using Sheet or Dialog
  - User preference handling with interactive components

#### Authentication Module
- **Purpose**: Handle user authentication and session management with shadcn/ui
- **Technology Stack**: shadcn/ui Button, Card, Form, Avatar, DropdownMenu components
- **Responsibilities**:
  - Login/logout interface using Card and Button components
  - Social login integration with styled Button variants
  - User profile display using Avatar and DropdownMenu components
  - Session state management with loading states using Skeleton
- **Key Interfaces**: 
  - Auth0 integration with shadcn/ui components
  - Protected route handling with loading states
  - User menu and profile management using DropdownMenu

### Shared Components

#### UI Component Library (shadcn/ui + Tailwind CSS)
- **Purpose**: Provide consistent, accessible UI elements across the application using shadcn/ui components
- **Technology Stack**: 
  - **Tailwind CSS**: Utility-first CSS framework for styling
  - **shadcn/ui**: High-quality, accessible React components built on Radix UI
  - **Radix UI**: Unstyled, accessible components as foundation
- **Responsibilities**:
  - Reusable form components (Input, Select, Textarea, Checkbox, RadioGroup)
  - Navigation and layout components (Navigation Menu, Breadcrumb, Sidebar)
  - Data display components (Card, Table, Badge, Avatar, Separator)
  - Interactive components (Button, Dialog, Dropdown Menu, Tooltip, Sheet)
  - Feedback components (Alert, Toast, Progress, Skeleton)
  - Loading and error state components
- **Key Interfaces**: 
  - shadcn/ui component props and variants
  - Tailwind CSS utility classes for custom styling
  - Accessibility features built into Radix UI primitives
  - Dark/light theme support through CSS variables

#### Data Services Layer
- **Purpose**: Handle all API communication and data management
- **Responsibilities**:
  - API client configuration and requests
  - Data caching and state management
  - Error handling and retry logic
  - Request/response transformation
- **Key Interfaces**: REST API integration, caching layer, error boundaries

## Backend Components (FastAPI Modular Monolith)

### Core Domain Modules

#### Recipe Domain
- **Purpose**: Manage all recipe-related business logic and data
- **Responsibilities**:
  - Recipe CRUD operations
  - Recipe validation and sanitization
  - Recipe metadata management
  - Recipe versioning and history
- **Key Interfaces**: Recipe repository, validation services, metadata extraction

#### User Domain
- **Purpose**: Handle user management and profile operations
- **Responsibilities**:
  - User profile management
  - User preferences and settings
  - User data export and deletion (GDPR)
  - User activity tracking
- **Key Interfaces**: User repository, profile services, privacy controls

#### AI Processing Domain
- **Purpose**: Coordinate all AI-related operations and processing
- **Responsibilities**:
  - AI service orchestration
  - Processing pipeline management
  - Cost optimization and monitoring
  - AI result caching and storage
- **Key Interfaces**: OpenAI integration, processing queues, result storage

#### Search Domain
- **Purpose**: Provide comprehensive search capabilities
- **Responsibilities**:
  - Search query processing
  - Hybrid search coordination (database + vector)
  - Search result ranking and filtering
  - Search analytics and optimization
- **Key Interfaces**: Database queries, vector search, result aggregation

### AI Processing Components

#### Core AI Service
- **Purpose**: Central coordination of all AI operations
- **Responsibilities**:
  - OpenAI API management and authentication
  - Request routing to specialized processors
  - Cost tracking and optimization
  - Error handling and fallback strategies
- **Key Interfaces**: OpenAI client, processor coordination, monitoring

#### Embedding Processor
- **Purpose**: Handle recipe embedding generation and management
- **Responsibilities**:
  - Recipe text preprocessing for embeddings
  - OpenAI embedding API calls
  - Embedding storage and indexing
  - Embedding similarity calculations
- **Key Interfaces**: Text processing, OpenAI embeddings API, vector storage

#### Recommendation Processor
- **Purpose**: Generate personalized recipe recommendations
- **Responsibilities**:
  - User taste profile analysis
  - Recipe similarity matching
  - Recommendation ranking and filtering
  - Recommendation result formatting
- **Key Interfaces**: Taste profile service, similarity matching, ranking algorithms

#### Parsing Processor
- **Purpose**: Smart parsing of recipe ingredients and metadata
- **Responsibilities**:
  - Ingredient text parsing and structuring
  - Recipe metadata extraction
  - Content enrichment and standardization
  - Parsing result validation
- **Key Interfaces**: NLP processing, structured data extraction, validation

### Infrastructure Components

#### Database Layer
- **Purpose**: Manage all data persistence and retrieval
- **Responsibilities**:
  - PostgreSQL connection management
  - Database migrations and schema management
  - Query optimization and performance monitoring
  - Vector search integration (pgvector)
- **Key Interfaces**: Database connections, migration tools, query builders

#### Caching Layer
- **Purpose**: Implement multi-level caching strategy
- **Responsibilities**:
  - Redis connection and session management
  - Cache key management and TTL strategies
  - Cache invalidation and refresh logic
  - Performance monitoring and optimization
- **Key Interfaces**: Redis client, cache strategies, invalidation patterns

#### Task Queue System
- **Purpose**: Handle asynchronous processing and background tasks
- **Responsibilities**:
  - Task queue management and processing
  - Job scheduling and retry logic
  - Task monitoring and error handling
  - Resource management and scaling
- **Key Interfaces**: Queue management, task processing, monitoring

#### Authentication Integration
- **Purpose**: Integrate with Auth0 for authentication services
- **Responsibilities**:
  - Auth0 configuration and integration
  - JWT token validation and management
  - User session handling
  - Security policy enforcement
- **Key Interfaces**: Auth0 SDK, JWT handling, session management

## Component Integration Patterns

### Frontend-Backend Integration
- **Pattern**: RESTful API with JSON communication
- **Authentication**: JWT tokens from Auth0
- **Error Handling**: Standardized error responses with user-friendly messages
- **Caching**: Client-side caching for static data, server-side for dynamic content

### AI Processing Integration
- **Pattern**: Asynchronous processing with immediate feedback
- **Queue Management**: Background tasks for AI enrichment
- **Cost Control**: Batching and caching to minimize API calls
- **Fallback Strategy**: Graceful degradation when AI services unavailable

### Data Flow Integration
- **Pattern**: Event-driven updates with caching layers
- **Consistency**: Eventual consistency for AI-processed data
- **Performance**: Multi-level caching with intelligent invalidation
- **Monitoring**: Comprehensive logging and metrics collection
