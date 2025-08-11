# FitTrack — AI-Powered Personal Fit 5

Description

FitTrack is a web and mobile platform that uses AI to generate personalized workout and nutrition plans based on user goals, preferences, and real-time health tracking data. The implementation uses Go for the AI/ML inference and services, GraphQL for the API schema and gateway, and ASP.NET Core for the main backend and API orchestration.

## Tech Stack

- Go (AI/ML service and supporting microservices)
- GraphQL (API schema and gateway)
- ASP.NET Core (backend, authentication, and GraphQL host)

## Requirements

- Provide personalized workout and nutrition plans using AI based on user goals and preferences.
- Consume real-time health tracking data from devices (heart rate, steps, sleep, etc.) and incorporate into AI-generated plans.
- Serve a GraphQL API for web and mobile clients.
- Use Go for AI/ML and inference logic and ASP.NET Core to host and orchestrate the GraphQL API and backend concerns.

## Installation

Prerequisites

- Go 1.18+ installed and available in PATH
- .NET SDK (6.0 or later; ensure compatibility with your project templates)
- Git

Clone repository

bash
git clone https://github.com/your-org/fittrack-ai-powered-personal-fit-5.git
cd fittrack-ai-powered-personal-fit-5


Environment variables

Create an environment file or set environment variables in your shell. At minimum set:

- ASPNETCORE_ENVIRONMENT=Development
- ASPNETCORE_URLS=http://localhost:5000
- GRAPHQL_PATH=/graphql
- JWT_SECRET=your_jwt_secret_here
- DATABASE_URL=<your-database-connection-string>  # generic DB connection string
- AI_MODEL_PATH=/path/to/ai/model or endpoint configuration
- AI_SERVICE_PORT=8081
- GO_AI_BIND_ADDR=localhost:8081

Note: The project is intentionally agnostic about the persistent data store. Configure DATABASE_URL to match your chosen DB and migration tooling.

Install and run Go AI service (dev)

bash
# From repository root
cd services/ai-service
go mod download
# Run in dev (hot-reload tools are not assumed; use go run)
go run ./cmd/main.go
# Or build
go build -o bin/ai-service ./cmd/main.go
./bin/ai-service


Install and run ASP.NET Core backend (GraphQL host)

bash
# From repository root
cd services/backend
# Restore and build
dotnet restore
dotnet build
# Run backend (will host GraphQL endpoint)
dotnet run --urls "http://localhost:5000"


Files you should see in the repository (important for these stacks)

- services/ai-service/go.mod
- services/ai-service/cmd/main.go
- services/ai-service/internal/inference.go
- services/backend/FitTrack.Backend.csproj
- services/backend/Program.cs (or Startup.cs depending on template)
- services/backend/appsettings.json
- services/backend/schema.graphql

## Usage

Run both the Go AI service and the ASP.NET Core backend as described in Installation. By default:

- ASP.NET Core GraphQL endpoint: http://localhost:5000/graphql
- AI service health: http://localhost:8081/health (or configured AI_SERVICE_PORT)

Examples

1) Generate a personalized plan (GraphQL mutation)

POST a GraphQL mutation to the /graphql endpoint. Example payload (JSON):


{
  "query": "mutation GeneratePlan($input: PlanRequestInput!) { generatePlan(input: $input) { planId, summary, recommendedWorkouts { title, durationMinutes }, nutrition { calories, macros } } }",
  "variables": {
    "input": {
      "userId": "user-123",
      "goals": { "target": "weight_loss", "targetValue": 5 },
      "preferences": { "workoutDaysPerWeek": 4, "dietaryRestrictions": ["gluten-free"] },
      "latestHealthData": { "restingHeartRate": 60, "sleepHours": 7 }
    }
  }
}


2) Submit real-time tracking data (GraphQL mutation)

Clients should send incoming device data to a mutation like updateHealthData which the backend funnels to the AI service or stores then triggers plan adjustments.

3) Subscribe to plan updates (GraphQL subscriptions)

If using subscriptions, clients can open a WebSocket connection to /graphql and subscribe to events like planUpdated for real-time updates when the AI service recalculates a plan.

## Implementation Steps

1. Repository layout
   - services/backend/  -> ASP.NET Core project (GraphQL host, auth, orchestration)
   - services/ai-service/ -> Go module (AI inference, model loader, health endpoint)
   - schema/ or services/backend/schema.graphql -> central GraphQL schema definitions

2. Create ASP.NET Core project files
   - FitTrack.Backend.csproj
   - Program.cs (or Program + Startup depending on template) to configure Kestrel and environment
   - appsettings.json for configuration values (GRAPHQL_PATH, connection strings)
   - schema.graphql file containing types, queries, mutations, and subscriptions
   - Implement GraphQL resolvers that call the Go AI service and data store
   - Provide a health endpoint: GET /health

3. Create Go AI service
   - go.mod
   - cmd/main.go: start HTTP server, expose /health and /infer endpoints
   - internal/inference.go: model loading, request handling, and inference orchestration
   - Provide a configuration mechanism (environment variables) for model path and listen address

4. GraphQL schema and mappings
   - Define input and output types for PlanRequestInput, PlanResult, HealthDataInput
   - Create queries: getPlan(planId), listPlans(userId)
   - Create mutations: generatePlan(input), updateHealthData(input)
   - Create subscriptions: planUpdated(userId) for real-time notifications

5. Authentication and authorization
   - Implement JWT-based auth in ASP.NET Core; protect GraphQL mutations and sensitive queries
   - Environment variable JWT_SECRET to sign and validate tokens

6. Connect GraphQL resolvers to services
   - Resolvers call the AI service over HTTP (e.g., POST to GO_AI_BIND_ADDR/infer)
   - Store plan metadata in your configured database via DATABASE_URL
   - When health data arrives, persist then trigger an async request to AI service for plan recalculation

7. Testing and validation
   - Unit tests for Go inference package
   - Integration tests for GraphQL endpoints in ASP.NET Core

8. Observability and health
   - Implement simple /health endpoints for both services
   - Log inference requests and GraphQL execution traces for debugging

9. Deployment (developer-friendly)
   - Use standard dotnet publish and go build to produce binaries
   - Deploy services to your chosen infrastructure (not prescribed here)

(Optional) ## API Endpoints

The architecture centers on GraphQL, but both services expose minimal HTTP endpoints for health and orchestration.

- POST /graphql (ASP.NET Core)
  - Purpose: GraphQL HTTP endpoint for queries and mutations. Example operations: generatePlan, updateHealthData, getPlan.

- GET /graphql (ASP.NET Core, development)
  - Purpose: GraphQL IDE/Playground (development only) to explore schema and run queries.

- WS /graphql (ASP.NET Core)
  - Purpose: GraphQL subscriptions over WebSocket for real-time updates like planUpdated.

- GET /health (ASP.NET Core)
  - Purpose: Backend health check.

- GET /health (Go AI service)
  - Purpose: AI service health check for orchestration and readiness.

- POST /infer or POST /v1/infer (Go AI service)
  - Purpose: Internal endpoint the ASP.NET Core resolvers call to request plan generation or recalculation. Receives structured request with user context, preferences, and health data and returns a plan result.

Notes

- This README intentionally focuses on the selected stacks: Go (AI/ML services), GraphQL (API schema), and ASP.NET Core (backend and GraphQL host). It does not prescribe a specific database or external AI provider; configure DATABASE_URL and AI_MODEL_PATH/endpoint according to your environment and compliance needs.
- Include required files for these stacks (go.mod, main.go, .csproj, Program.cs / Startup.cs, appsettings.json, schema.graphql) when creating the repository.

If you want, I can generate initial skeleton files for the Go AI service (go.mod, main.go, a minimal inference handler) and for the ASP.NET Core backend (csproj, Program.cs, sample schema.graphql and a minimal resolver) to jump-start development.