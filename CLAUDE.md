# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open-CICD is a modern, HTTP-first CI/CD tool written in Zig that provides an alternative to Jenkins. It follows a control plane/data plane architecture where the control plane is a Zap-based HTTP server and data plane consists of ephemeral Docker-based agents.

## Architecture

### Control Plane
- **HTTP Server**: Zap (Zig) for blazing fast HTTP performance
- **Database**: PostgreSQL for metadata, Redis for caching
- **Storage**: S3-compatible (MinIO) for artifacts and logs
- **UI**: SvelteKit for modern, reactive web interface

### Data Plane
- **Agents**: Docker containers with HTTP endpoints
- **Communication**: HTTP/JSON API between control plane and agents
- **Registration**: Agents self-register via `POST /register`
- **Health**: Control plane monitors agents via `GET /health`

## Development Commands

### Building and Running
```bash
# Build the project
zig build

# Run the application
zig build run

# Run tests
zig build test
```

### Dependencies
- Add Zap HTTP server: `zig fetch --save https://github.com/zigzap/zap/archive/v0.8.0.tar.gz`
- PostgreSQL client library for database integration
- JSON parsing library for HTTP API

## Project Structure

```
src/
├── main.zig              # Entry point
├── server/               # Control plane HTTP server
│   ├── server.zig       # Zap HTTP server setup
│   ├── handlers/        # HTTP request handlers
│   │   ├── register.zig # Agent registration endpoint
│   │   ├── jobs.zig     # Job management endpoints
│   │   └── health.zig   # Health check endpoints
│   └── middleware/      # HTTP middleware
├── agent/               # Agent-related code
│   ├── client.zig      # HTTP client for agent communication
│   └── types.zig       # Agent data structures
├── database/            # Database integration
│   ├── postgres.zig    # PostgreSQL connection and queries
│   └── redis.zig       # Redis caching layer
├── types/              # Common data structures
│   ├── job.zig         # Job-related types
│   ├── agent.zig       # Agent-related types
│   └── api.zig         # API request/response types
└── utils/              # Utility functions
    ├── json.zig        # JSON serialization helpers
    └── http.zig        # HTTP utility functions
```

## Key API Endpoints

### Control Plane Endpoints
- `POST /register` - Agent registration with capabilities
- `GET /agents` - List all active agents
- `POST /jobs` - Create new job
- `GET /jobs/{id}` - Get job status and logs
- `POST /jobs/{id}/status` - Receive status updates from agents
- `GET /health` - Control plane health check

### Agent Endpoints
- `GET /health` - Agent health status
- `POST /jobs` - Receive job assignment from control plane
- `GET /jobs/{id}` - Query specific job status
- `DELETE /jobs/{id}` - Cancel running job

## Database Schema

### Key Tables
- `agents` - Registered agents with capabilities and status
- `jobs` - Job definitions and current status
- `job_logs` - Streaming logs from job execution
- `artifacts` - Metadata for build artifacts stored in S3

## Development Workflow

### Backend Development (Zig)
1. **Agent Registration**: Agents POST to `/register` with their capabilities
2. **Health Monitoring**: Control plane polls agent `/health` endpoints
3. **Job Assignment**: Control plane POSTs jobs to selected agents
4. **Status Updates**: Agents POST status updates to control plane
5. **Log Streaming**: Real-time log collection via HTTP endpoints

### Frontend Development (SvelteKit)
1. **Real-time Updates**: WebSocket/SSE integration for live job status
2. **Pipeline Visualization**: Interactive DAG graphs with real-time status
3. **Log Streaming**: Live log viewer with syntax highlighting
4. **Resource Monitoring**: Real-time CPU/memory charts and metrics
5. **Modern UI Components**: Responsive design with TailwindCSS

## Testing

### Unit Tests
```bash
# Run all tests
zig build test

# Run specific test file
zig test src/server/handlers/register.zig
```

### Integration Tests
- Test agent registration flow
- Test job assignment and execution
- Test health monitoring
- Test API endpoints with mock agents

## Performance Considerations

- Zap HTTP server provides excellent performance (microsecond response times)
- Use connection pooling for PostgreSQL
- Implement caching with Redis for frequently accessed data
- Use HTTP/1.1 keep-alive for agent communication
- Consider rate limiting for agent endpoints

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