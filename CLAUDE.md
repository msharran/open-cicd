# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open-CICD is a modern, HTTP-first CI/CD tool written in Go that provides an alternative to Jenkins. It follows a control plane/data plane architecture where the control plane is a Go HTTP server and data plane consists of ephemeral Docker-based agents.

## Architecture

### Control Plane
- **HTTP Server**: Go standard library with Gorilla Mux for high-performance HTTP
- **Job Queue**: Internal queue for pending jobs
- **Scheduler**: Kubernetes-style push scheduler that assigns jobs to agents
- **Agent Registry**: Tracks agent capabilities and availability
- **Database**: PostgreSQL for metadata, Redis for caching
- **Storage**: S3-compatible (MinIO) for artifacts and logs
- **UI**: SvelteKit for modern, reactive web interface

### Data Plane
- **Agents**: Docker containers with HTTP endpoints
- **Communication**: HTTP/JSON API for job assignment and status updates
- **Registration**: Agents self-register via `POST /register`
- **Job Assignment**: Control plane pushes jobs via `POST /jobs` to selected agents
- **Health**: Control plane monitors agents via `GET /health`

## Development Commands

### Building and Running
```bash
# Build the project
go build ./cmd/server

# Run the application
go run ./cmd/server

# Run tests
go test ./...
```

### Dependencies
- Gorilla Mux for HTTP routing: `go get github.com/gorilla/mux`
- PostgreSQL driver: `go get github.com/lib/pq`
- Redis client: `go get github.com/redis/go-redis/v9`
- JSON parsing uses Go standard library encoding/json

## Project Structure

```
cmd/
├── server/              # Control plane server entry point
│   └── main.go         # Server main function
├── agent/              # Agent entry point
│   └── main.go         # Agent main function
internal/
├── server/             # Control plane HTTP server
│   ├── server.go       # HTTP server setup
│   ├── scheduler/      # Job scheduling components
│   │   ├── queue.go    # Job queue management
│   │   ├── scheduler.go # Kubernetes-style job scheduler
│   │   └── registry.go # Agent capability registry
│   ├── handlers/       # HTTP request handlers
│   │   ├── register.go # Agent registration endpoint
│   │   ├── jobs.go     # Job management endpoints
│   │   ├── logs.go     # Log streaming endpoints (SSE)
│   │   └── health.go   # Health check endpoints
│   └── middleware/     # HTTP middleware
├── agent/              # Agent-related code
│   ├── client.go       # HTTP client for agent communication
│   └── types.go        # Agent data structures
├── database/           # Database integration
│   ├── postgres.go     # PostgreSQL connection and queries
│   └── redis.go        # Redis caching layer
├── types/              # Common data structures
│   ├── job.go          # Job-related types
│   ├── agent.go        # Agent-related types
│   └── api.go          # API request/response types
└── utils/              # Utility functions
    ├── json.go         # JSON serialization helpers
    └── http.go         # HTTP utility functions
```

## Key API Endpoints

### Control Plane Endpoints
- `POST /register` - Agent registration with capabilities
- `GET /agents` - List all active agents
- `POST /jobs` - Create new job (adds to queue)
- `GET /jobs/{id}` - Get job status and logs
- `POST /jobs/{id}/status` - Receive status updates from agents
- `GET /jobs/{id}/logs/stream` - SSE endpoint for real-time log streaming
- `POST /jobs/{id}/logs` - Receive log chunks from agents
- `GET /health` - Control plane health check

### Agent Endpoints
- `GET /health` - Agent health status
- `POST /jobs` - Receive job assignment from scheduler (pushed by control plane)
- `GET /jobs/{id}` - Query specific job status
- `DELETE /jobs/{id}` - Cancel running job

## Database Schema

### Key Tables
- `agents` - Registered agents with capabilities and status
- `jobs` - Job definitions and current status
- `job_logs` - Streaming logs from job execution
- `artifacts` - Metadata for build artifacts stored in S3

## Development Workflow

### Backend Development (Go)
1. **Agent Registration**: Agents POST to `/register` with capabilities, stored in registry
2. **Job Submission**: User submits job → Job Queue → Scheduler watches queue
3. **Job Assignment**: Scheduler selects suitable agent → POSTs job to agent
4. **Status Updates**: Agents POST status updates back to control plane
5. **Log Streaming**: Agents POST log chunks → Control plane broadcasts via SSE
6. **Health Monitoring**: Control plane polls agent `/health` endpoints periodically

### Frontend Development (SvelteKit)
1. **Real-time Updates**: SSE integration for live job status and notifications
2. **Pipeline Visualization**: Interactive DAG graphs with real-time status updates
3. **Log Streaming**: Live log viewer with syntax highlighting via SSE
4. **Resource Monitoring**: Real-time CPU/memory charts and metrics
5. **Modern UI Components**: Responsive design with TailwindCSS

## Testing

### Unit Tests
```bash
# Run all tests
go test ./...

# Run specific package tests
go test ./internal/server/handlers

# Run with verbose output
go test -v ./...
```

### Integration Tests
- Test agent registration flow
- Test job assignment and execution
- Test health monitoring
- Test API endpoints with mock agents

## Performance Considerations

- Go HTTP server provides excellent performance with built-in concurrency
- Use connection pooling for PostgreSQL (database/sql package)
- Implement caching with Redis for frequently accessed data
- Use HTTP/1.1 keep-alive for agent communication
- Consider rate limiting for agent endpoints
- Leverage Go's goroutines for concurrent job processing

## Security Notes

- Implement JWT-based authentication for API access
- Use HTTPS for all communications in production
- Validate all input data from agents
- Implement proper error handling without exposing internals
- Store sensitive configuration in environment variables

## SvelteKit Frontend Features

### Real-time Capabilities
- **Live Build Logs**: High-frequency log streaming with virtual scrolling
- **Pipeline Graphs**: Interactive DAG visualization with real-time status updates
- **Resource Monitoring**: Live CPU/memory charts using Chart.js or D3.js
- **WebSocket Integration**: Real-time job status, agent health, and notifications

### Modern UI Components
- **Responsive Design**: Mobile-first with TailwindCSS
- **Dark/Light Mode**: User preference with system detection
- **Advanced Filtering**: Complex log search, job filtering, time ranges
- **Drag & Drop**: Pipeline editor and job management
- **Toast Notifications**: Real-time alerts and status updates

### Development Approach
1. **Backend First**: Complete Zig HTTP server with all endpoints
2. **API Design**: RESTful API with WebSocket support for real-time features
3. **Frontend Second**: SvelteKit app consuming the HTTP API
4. **Real-time Integration**: WebSocket/SSE for live updates
5. **Progressive Enhancement**: Start with basic UI, add real-time features

## Future Enhancements

- YAML pipeline parser for GitOps workflows
- GitHub/GitLab webhook integration
- Advanced pipeline visualization with drag-drop editor
- Agent auto-scaling based on job queue
- Build artifact caching system
- Multi-tenant support with role-based access
- WebSocket support for real-time updates
- Prometheus metrics integration
- Distributed tracing with OpenTelemetry
- Plugin system for custom step types

## Development Commands Reference

### Build Commands
```bash
# Build the control plane server
go build ./cmd/server

# Build the agent
go build ./cmd/agent

# Build everything
go build ./...

# Run the server
go run ./cmd/server

# Run an agent
go run ./cmd/agent
```

### Database Commands
```bash
# Run database migrations (using migrate tool)
migrate -path ./migrations -database "postgres://..." up

# Create new migration
migrate create -ext sql -dir ./migrations migration_name

# Reset database
migrate -path ./migrations -database "postgres://..." down
```

## Important Instruction Reminders

### Communication Protocol
- **HTTP-First**: Use HTTP/JSON for stateless agent communication
- **Job Assignment**: Kubernetes-style push scheduling via HTTP POST to agents
- **Status Updates**: Agents POST status updates back to control plane
- **Log Streaming**: Server-Sent Events (SSE) for unidirectional real-time log streaming

### Architecture Principles
- **Domain-Driven**: Each domain (server, agent, jobs, database) is self-contained
- **HTTP-First**: Stateless HTTP/JSON for agent communication, SSE for real-time updates
- **Push Scheduling**: Kubernetes-style scheduler pushes jobs to agents (no polling)
- **State Machine**: Use explicit state machines for job and agent lifecycle management
- **Concurrent by Default**: Leverage Go's goroutines for I/O operations

### Development Guidelines
- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User
- Follow Go naming conventions and best practices
- Use explicit error handling with Go's error system
- Prefer compile-time safety and clear error propagation

## Project Status
- Architecture updated to use Go + HTTP-first communication
- Domain-driven structure implemented
- State machine patterns defined
- HTTP API endpoints specified
- Database schema updated for PostgreSQL + Redis
- Ready for implementation phase
