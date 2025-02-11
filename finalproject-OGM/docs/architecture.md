# Architecture

## Table of Contents
- [Technical Stack](#technical-stack)
  - [Frontend](#frontend)
  - [Backend](#backend)
- [Architecture Patterns](#architecture-patterns)
  - [CQRS/ES Implementation](#cqrses-implementation)
- [High-Level Design](#high-level-design)
  - [System Overview](#system-overview)
  - [Core Components & Interactions](#core-components--interactions)
- [C4 diagrams](#c4-diagrams)
  - [Context Diagram](#context-diagram)
  - [Container Diagram](#container-diagram)
- [Component Diagrams](#component-diagrams)
  - [API Gateway Module](#api-gateway-module)
  - [Command Module](#command-module)
  - [Query Module](#query-module)
  - [Authentication & Authorization Module](#authentication--authorization-module)
  - [Data Aggregation Module](#data-aggregation-module)
  - [Categorization Module](#categorization-module)
  - [Financial Insights Module](#financial-insights-module)
- [Deployment Model](#deployment-model)
- [Security Considerations](#security-considerations)
- [Infrastructure](#infrastructure)

## Technical Stack

### Frontend
- **Web Client**: 
  - Framework: Deno Fresh (Islands Architecture)
  - Language: TypeScript
  - State Management: Effect TS
  - UI Components: Preact with Tailwind CSS
- **Mobile Client**:
  - Framework: React Native
  - Language: TypeScript
  - State Management: Effect TS
  - UI Components: Native components with custom styling

### Backend
- **Runtime**: Deno
- **Framework**: Fresh for backend APIs
- **Core Libraries**: 
  - Effect TS for functional programming and error handling
  - TrueLayer SDK for financial data aggregation
  - Redis for event bus and caching
- **Database**: PostgreSQL with Effect TS query builders
- **Additional Libraries**:
  - EventStoreDB for event sourcing
  - Redis for event bus and read model cache
  - PostgreSQL for read models

## Architecture Patterns

### CQRS/ES Implementation
The system implements Command Query Responsibility Segregation (CQRS) with Event Sourcing (ES) to achieve:
- Clear separation between write and read operations
- Audit trail and time travel capabilities through event sourcing
- Optimized read models for different client needs
- Eventual consistency with immediate read-your-writes

#### Event Store
- Uses EventStoreDB to store all events
- Events are immutable and represent facts that happened
- Aggregate roots ensure consistency boundaries
- Event streams organized by aggregate type and ID

#### Read Models
- Optimized for specific query needs
- Updated asynchronously via event handlers
- Cached in Redis for performance
- Separate models for web and mobile clients when needed

## High-Level Design

### **System Overview**
The **Welz** platform consists of two main components: 
1. **Frontend Applications** – A web-based client and a mobile application for users to manage their finances.
2. **Backend System** – A single backend service that handles authentication, data aggregation, AI-based categorization, and financial insights.

The system follows a **monolithic backend architecture** while ensuring modularity within the backend to maintain clear responsibility for different functions and to allow future segregation.

---

### **Core Components & Interactions**

#### **Frontend Applications**
- **Web Client**: A responsive web application for users to manage their finances from any device.
- **Mobile Application**: Native mobile app for on-the-go financial management

#### **Backend Core Components**
- **API Gateway**: Single entry point for all client requests, handles routing and basic request processing
- **Command Module**: Handles all write operations and mutations
- **Query Module**: Handles all read operations and data retrieval
- **Authentication & Authorization Module**: Manages user authentication and role-based access control
- **Data Aggregation Module**: Connects to external bank and investment APIs to aggregate and normalize financial data
- **Categorization Module**: Automatically categorizes transactions, allows users to change transaction categories, and refines categorisation accuracy based on user input.
- **Financial Insights Module**: Computes real-time net worth and generates financial reports and AI-driven suggestions.
- **Event Bus**: Redis-backed event bus for async communication between modules
- **Database**: PostgreSQL for relational data storage with read replicas for queries

## Infrastructure
Describe the infrastructure setup, including cloud regions, CI/CD pipelines, and monitoring tools.

## Deployment Model
Describe the deployment model, including Docker containers for on-prem PoC and future Kubernetes deployment in the cloud.

## Security Considerations
Detail security measures, including encryption, audit logs, and compliance with GDPR and PSD2.

## C4 diagrams

### **Context Diagram**
The context diagram provides a high-level overview of the system and its interactions with external entities.

```plantuml
@startuml
package "Welz Platform" {
    [Web Client]
    [Mobile App]
    [Backend System]
}

actor User
cloud "OAuth Provider"
cloud "Bank & Investment APIs"

User --> [Web Client] : Uses
User --> [Mobile App] : Uses
[Web Client] --> [Backend System] : API Requests
[Mobile App] --> [Backend System] : API Requests
[Backend System] --> [OAuth Provider] : OAuth / OpenID
[Backend System] --> [Bank & Investment APIs] : Sync
@enduml
```

**Description**: 
- Users interact with the Welz platform through the Web Client or Mobile App.
- Both clients send API requests to the Backend System.
- The Backend System handles authentication via an OAuth Provider and synchronizes financial data with various Bank APIs.

### **Container Diagram**

```plantuml
@startuml
actor "User"

package "Welz Platform" {
    [Web Client]
    
    package "Backend" {
        [API Gateway]
        [Command Module]
        [Query Module]
        [Authentication Module]

        package "Domain Services" {
            [Data Aggregation Module]
            [Categorization Module]
            [Financial Insights Module]
        }
        queue "Event Bus"
        database "PostgreSQL"
    }
}

cloud "OAuth Provider"
cloud "Bank & Investment APIs"

' Client flows
User --> [Web Client]
[Web Client] --> [API Gateway]

' Gateway flows
[API Gateway] --> [Command Module] : "Write operations"
[API Gateway] --> [Query Module] : "Read operations"
[API Gateway] -right-> [Authentication Module]

' Command & Domain Service interactions
[Command Module] --> [Domain Services] : "Service calls"
[Command Module] -left-> [PostgreSQL] : "Write"

' Query flows
[Query Module] -right-> [PostgreSQL] : "Read"

' Event flows
[Command Module] --> [Event Bus] : "Publish events"
[Event Bus] -up-> [Query Module] : "Subscribe"
[Event Bus] --> [Financial Insights Module] : "Subscribe"
[Event Bus] --> [Data Aggregation Module] : "Subscribe"
[Event Bus] --> [Categorization Module] : "Subscribe"

' External flows
[Authentication Module] -right-> [OAuth Provider]
[Data Aggregation Module] -down-> [Bank & Investment APIs]

@enduml
```

#### API Gateway
- Single entry point for all API requests
- Handles request routing, validation, and authentication
- Manages API versioning and documentation
- Implements rate limiting and request throttling
- Routes requests to Command or Query modules based on operation type

#### Command Module
- Processes write commands from external and internal sources
- Enforce domain rules and maintain data consistency
- Generate and store events in the Event Store
- Handle idempotency and retries
- Publish events to subscribers via Event Bus

#### Query Module
- Handles all read operations
- Implements caching strategies
- Uses read replicas for better performance
- Subscribes to relevant events to invalidate caches

#### Authentication & Authorization Module
- Handles user authentication via an OAuth Provider
- Manages Role-Based Access Control (RBAC) for profile sharing

#### Data Aggregation Module
- Connects to external bank and investment APIs
- Normalizes data into a unified financial model
- Synchronizes data with various Bank APIs and stores it in the Database

#### AI-Powered Categorization Module
- Automatically categorizes transactions based on past behavior
- Allows users to change transaction categories providing reinforcement to categorization accuracy
- Stores categorized transactions in the Database

#### Financial Insights Module
- Computes real-time net worth based on assets and liabilities
- Generates financial reports and AI-driven suggestions
- Stores financial metrics in the Database

#### Event Bus
- Redis-backed event bus for async communication between modules

#### Database
- PostgreSQL for relational data storage
- Ensures efficient and structured data access

## Component Diagrams

### **API Gateway Module**

The API Gateway Module serves as the unified entry point for all external requests, with client-specific response transformation for web and mobile clients.

#### **Core Responsibilities:**
- Process all incoming requests regardless of client type
- Route requests to Command or Query modules based on operation type
- Implement unified authentication and authorization
- Apply rate limiting and request throttling
- Transform responses based on client type and version
- Provide unified error handling and logging
- Monitor API health and metrics

#### **Component Diagram:**
```plantuml
@startuml
[Web Client]
[Mobile App]

package "API Gateway" {
    [Request Handler]
    [Auth Middleware]
    [Rate Limiter]
    [Route Resolver]
    [Web Transformer]
    [Mobile Transformer]
    [API Documentation]
}

package "Backend Modules" {
    [Command Module]
    [Query Module]
    [Authentication Module]
}

[Web Client] --> [Request Handler]
[Mobile App] --> [Request Handler]

[Request Handler] --> [Auth Middleware]
[Request Handler] --> [Rate Limiter]
[Request Handler] --> [Route Resolver]

[Auth Middleware] --> [Authentication Module]
[Route Resolver] --> [Command Module] : Write ops
[Route Resolver] --> [Query Module] : Read ops

[Command Module] --> [Request Handler]
[Query Module] --> [Request Handler]

[Request Handler] -left-> [Web Transformer] : Web requests
[Request Handler] -right-> [Mobile Transformer] : Mobile requests
@enduml
```

#### **Component Details:**

**Request Handler**
- Unified entry point for all HTTP requests
- Client type detection and routing
- Request lifecycle management
- Circuit breaking for downstream services
- Response transformer selection

**Auth Middleware**
- Unified token validation for all clients
- Role-based access control
- Session management
- Device-specific authentication flows

**Rate Limiter**
- Unified rate limiting strategy
- Client-specific quota management
- Rate limit headers in responses

**Route Resolver**
- Command/Query operation routing
- Unified timeout policies
- Core business logic routing

**Web Transformer**
- Web-specific response formats
- Desktop browser optimizations
- Web-specific caching strategies

**Mobile Transformer**
- Mobile-optimized payloads
- Version-specific transformations
- Bandwidth-conscious compression
- Partial response support

**API Documentation**
- OpenAPI/Swagger specifications
- Client-specific examples
- Version compatibility matrices

### **Financial Insights Module**

TODO: Document the module

### **Command Module**

The Command Module handles all write operations in the system, serving as the single entry point for both API and internal commands from domain modules.

#### **Core Responsibilities:**
- Process write commands from external and internal sources
- Enforce domain rules and maintain data consistency
- Generate and store events in the Event Store
- Handle idempotency and retries
- Publish events to subscribers via Event Bus

#### **Component Diagram:**
```plantuml
@startuml
package "Command Module" {
    [Command Handler]
    [Command Validator]
    [Aggregate Root]
    [Event Generator]
    [Idempotency Manager]
    database "Event Store"
}

queue "Event Bus"

[API Gateway] --> [Command Handler] : User Commands
[Domain Services] --> [Command Handler] : Sync/Categorization Commands

[Command Handler] --> [Command Validator] : Validate
[Command Handler] -left-> [Idempotency Manager] : Check

[Command Validator] --> [Domain Services] : Business Rules

[Command Handler] --> [Aggregate Root] : Update
[Aggregate Root] --> [Event Generator] : Generate Events
[Event Generator] --> [Event Store] : Store Events
[Event Generator] --> [Event Bus] : Publish Events
@enduml
```

#### **Component Details:**

**Command Handler**
- Processes incoming commands from API Gateway
- Orchestrates command validation and execution
- Manages transaction boundaries
- Implements retry policies
- Returns command execution results

**Command Validator**
- Validates command structure and data
- Checks business rules and constraints
- Validates aggregate state transitions
- Enforces authorization rules
- Returns early on validation failures

**Aggregate Root**
- Maintains consistency boundaries
- Enforces invariants
- Manages entity state
- Generates domain events
- Ensures event ordering

**Event Generator**
- Creates versioned event objects
- Adds metadata and timestamps
- Manages event versioning
- Handles event schema evolution
- Ensures event immutability

**Idempotency Manager**
- Tracks processed commands
- Prevents duplicate processing
- Manages idempotency keys
- Handles concurrent commands
- Maintains command status

#### **Key Interactions:**

1. **External Command Flow**:
   ```
   API Gateway -> Command Handler -> Process
   ```

2. **Internal Command Flow**:
   ```
   Domain Module -> Command Handler -> Process
   ```

3. **Background Process Flow**:
   ```
   Data Sync Job -> Data Aggregation Module -> Command Handler -> Process
   ```

### **Query Module**

The Query Module handles all read operations, managing optimized read models and caching strategies for different client needs.

#### **Core Responsibilities:**
- Process read queries from external clients
- Maintain optimized read models
- Implement caching strategies
- Subscribe to relevant events for model updates
- Transform query results for different clients

#### **Component Diagram:**
```plantuml
@startuml
package "Query Module" {
    [Query Handler]
    [Cache Manager]
    [Read Model Updater]
    [Query Optimizer]
    database "Redis Cache"
    database "Read Models"
}

queue "Event Bus"
[API Gateway]
[Domain Services]

[API Gateway] --> [Query Handler] : Client Queries
[Domain Services] --> [Query Handler] : Internal Queries

[Query Handler] --> [Cache Manager] : Check Cache
[Cache Manager] --> [Redis Cache] : Get/Set

[Query Handler] --> [Query Optimizer] : Optimize & Execute
[Query Optimizer] --> [Read Models] : Fetch Data

[Event Bus] --> [Read Model Updater] : Domain Events
[Read Model Updater] --> [Read Models] : Update
[Read Model Updater] --> [Cache Manager] : Invalidate
@enduml
```

#### **Component Details:**

**Query Handler**
- Routes queries to appropriate handlers
- Implements query timeout policies
- Manages response transformation
- Handles partial response requests
- Returns cached/live data based on freshness requirements

**Cache Manager**
- Implements multi-level caching
- Manages cache invalidation
- Handles cache warming
- Supports different caching strategies per query type
- Provides cache statistics and monitoring

**Read Model Updater**
- Subscribes to domain events
- Updates read models asynchronously
- Maintains read model consistency
- Handles event ordering and versioning
- Manages model migrations

**Query Optimizer**
- Optimizes query execution plans
- Manages database connection pool
- Implements query result pagination
- Handles sorting and filtering
- Monitors query performance

#### **Key Interactions:**

1. **Query Flow with Cache Hit**:
   ```
   API Gateway -> Query Handler -> Cache Manager -> Return Cached Data
   ```

2. **Query Flow with Cache Miss**:
   ```
   Query Handler -> Query Optimizer -> Read Models -> Cache Manager -> Return Fresh Data
   ```

3. **Event Processing Flow**:
   ```
   Event Bus -> Read Model Updater -> Update Models -> Invalidate Cache
   ```

### **Authentication & Authorization Module**

The Authentication & Authorization Module manages user identity, access control, and secure session handling across all platform interfaces.

#### **Core Responsibilities:**
- Implement OAuth2/OpenID Connect flows
- Manage user sessions and tokens
- Enforce role-based access control (RBAC)
- Handle multi-device authentication
- Provide audit logging of security events

#### **Component Diagram:**
```plantuml
@startuml
package "Authentication Module" {
    [OAuth Handler]
    [Session Manager]
    [Token Service]
    [RBAC Manager]
    [Audit Logger]
    database "Session Store"
}

cloud "OAuth Providers" {
    [Google]
}

queue "Event Bus"
[API Gateway]
database "User Store"

' External flows
[OAuth Handler] -left-> [Google]

' Internal flows
[API Gateway] --> [OAuth Handler] : "Login Request"
[OAuth Handler] -right-> [Token Service] : "Generate Tokens"
[Token Service] -right-> [Session Manager] : "Create Session"
[Session Manager] -right-> [Session Store] : "Store"

[API Gateway] --> [Session Manager] : "Validate Session"
[Session Manager] --> [RBAC Manager] : "Check Permissions"
[RBAC Manager] --> [User Store] : "Fetch Roles"

' Audit flows
[OAuth Handler] --> [Audit Logger] : "Log Auth Events"
[RBAC Manager] -left-> [Audit Logger] : "Log Access Events"

' Event flows
[Audit Logger] --> [Event Bus] : "Security Events"
@enduml
```

#### **Component Details:**

**OAuth Handler**
- Implements OAuth2/OpenID Connect flows
- Manages provider-specific authentication
- Handles token exchange
- Validates OAuth responses
- Supports Google as provider

**Session Manager**
- Creates and validates sessions
- Manages session lifecycle
- Handles session expiration
- Supports multiple devices
- Implements session revocation

**Token Service**
- Generates JWT tokens
- Handles token refresh
- Manages token blacklisting
- Implements token rotation
- Ensures token security

**RBAC Manager**
- Manages role definitions
- Enforces access policies
- Handles permission inheritance
- Supports dynamic roles
- Validates access rights

**Audit Logger**
- Logs security events
- Tracks auth attempts
- Records permission changes
- Maintains audit trail
- Supports compliance reporting

#### **Key Interactions:**

1. **Authentication Flow**:
   ```
   User -> OAuth Provider -> OAuth Handler -> Token Service -> Session Manager
   ```

2. **Authorization Flow**:
   ```
   Request -> Session Manager -> RBAC Manager -> Grant/Deny Access
   ```

3. **Audit Flow**:
   ```
   Security Event -> Audit Logger -> Event Bus -> Compliance Reporting
   ```

### **Data Aggregation Module**

The Data Aggregation Module orchestrates the synchronization of financial data from multiple providers, handling both webhook-based and polling-based updates.

#### **Core Responsibilities:**
- Manage financial institution connections
- Handle provider-specific OAuth flows
- Process webhooks and polling updates
- Normalize data across providers
- Ensure data consistency and freshness

#### **Component Diagram:**
```plantuml
@startuml
package "Data Aggregation Module" {
    [Connection Manager]
    [Provider Gateway]
    [Webhook Handler]
    [Sync Scheduler]
    [Data Normalizer]
    database "Connection Cache"
}

cloud "Financial APIs" {
    [TrueLayer]
    [Stock APIs]
}

[Command Module]
queue "Event Bus"
database "Read Models"

' User flow
[Command Module] --> [Connection Manager] : "Connect\nRequest"

' Connection flows
[Provider Gateway] --> [TrueLayer] : "Auth & Connect"
[Provider Gateway] --> [Stock APIs] : "API Keys"
[Connection Manager] -down-> [Provider Gateway] : "OAuth Flow"

' Sync flows
[TrueLayer] --> [Webhook Handler] : "Updates"
[Sync Scheduler] --> [Provider Gateway] : "Poll Updates"
[Webhook Handler] -up-> [Provider Gateway] : "Push Updates"

' Data flows
[Provider Gateway] -up-> [Data Normalizer] : "Raw Data"
[Data Normalizer] --> [Command Module] : "Generate Commands"
[Command Module] -right-> [Event Bus] : "Entity Connected/Synced"
[Event Bus] -right-> [Read Models] : "Update Models"

' State management
[Connection Manager] -down-> [Connection Cache] : "Connection State"

' Diagram Layout
[TrueLayer] -[hidden]up-> [Stock APIs]
@enduml
```

#### **Component Details:**

**Connection Manager**
- Handles provider OAuth flows
- Manages API credentials
- Maintains connection state
- Implements retry logic
- Monitors provider health

**Provider Gateway**
- Implements provider APIs
- Handles rate limiting
- Manages API versioning
- Supports multiple providers
- Implements failover logic

**Webhook Handler**
- Processes TrueLayer webhooks
- Validates webhook signatures
- Handles duplicate events
- Manages webhook registration
- Monitors webhook health

**Sync Scheduler**
- Manages polling intervals
- Handles provider timeouts
- Implements backoff strategy
- Coordinates parallel syncs
- Tracks sync status

**Data Normalizer**
- Standardizes data formats
- Resolves data conflicts
- Enriches transaction data
- Validates data integrity
- Generates domain events

#### **Key Flows:**

1. **Entity Connection Flow**:
   ```
   Command Module -> Connection Manager -> Provider Gateway -> OAuth -> Command Module (EntityConnected)
   ```

2. **Webhook Update Flow**:
   ```
   TrueLayer -> Webhook Handler -> Provider Gateway -> Data Normalizer -> Command Module
   ```

3. **Polling Update Flow**:
   ```
   Scheduler -> Provider Gateway -> Data Normalizer -> Command Module
   ```

### **Categorization Module**

The Categorization Module provides automatic transaction categorization using AI/ML techniques and learns from user category modifications.

#### **Core Responsibilities:**
- Automatically categorize new transactions
- Learn from category modifications
- Maintain categorization rules
- Handle bulk categorization
- Continuously improve categorization accuracy

#### **Component Diagram:**
```plantuml
@startuml
package "Categorization Module" {
    [Category Manager]
    [ML Predictor]
    [Rules Engine]
    [Training Manager]
    database "ML Models"
    database "Training Data"
}

queue "Event Bus"
[Command Module]

' Event flows
[Category Manager] -right-> [Event Bus] : "Transaction\nCategorized"
[Event Bus] -left-> [Category Manager] : "Transaction Added"
[Command Module] --> [Category Manager] : "Change Transaction Category"

' Prediction flows
[Category Manager] -left-> [Rules Engine] : "Apply Rules"
[Category Manager] --> [ML Predictor] : "Predict Category"
[ML Predictor] --> [ML Models] : "Load Model"

' Training flows
[Category Manager] --> [Training Manager] : "User reinforcement"
[Training Manager] --> [Training Data] : "Store Examples"
[Training Manager] --> [ML Models] : "Update Model"
@enduml
```

#### **Component Details:**

**Category Manager**
- Orchestrates categorization workflow
- Manages category hierarchy
- Handles user overrides
- Coordinates prediction requests
- Publishes categorization events

**ML Predictor**
- Loads trained ML models
- Makes category predictions
- Handles prediction errors
- Provides confidence scores
- Supports model versioning

**Rules Engine**
- Applies predefined rules
- Handles regex patterns
- Manages rule priorities
- Supports custom rules
- Validates rule syntax

**Training Manager**
- Collects training examples
- Updates model periodically
- Validates training data
- Monitors model performance
- Manages model deployment

#### **Key Flows:**

1. **New Transaction Flow**:
    ```
    Event Bus -> Category Manager -> Rules Engine/ML Predictor -> Command Module (CategoryAssigned)
    ```

2. **User Override Flow**:
    ```
    Command Module -> Category Manager -> Training Manager -> Update Models
    ```

3. **Model Update Flow**:
    ```
    Training Manager -> Training Data -> Update Model -> ML Models
    ```
