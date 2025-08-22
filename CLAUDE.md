# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TypeScript-based MCP (Model Context Protocol) server that integrates with Excalidraw to provide real-time visual diagramming capabilities. It consists of three main components:

1. **MCP Server** (`src/index.ts`) - Handles MCP protocol communication with AI agents
2. **Canvas Server** (`src/server.ts`) - Express server that serves the frontend and provides REST API + WebSocket for real-time sync
3. **React Frontend** (`frontend/src/`) - Excalidraw canvas with WebSocket client for live updates

## Key Commands

### Development
```bash
# Install dependencies
npm install

# Build frontend and TypeScript backend
npm run build

# Start MCP server (after building)
npm start

# Start canvas server with frontend (production mode)
npm run canvas

# Development mode (TypeScript watch + Vite dev server)
npm run dev

# Type checking without compilation
npm run type-check
```

### Testing a Single Component
```bash
# Type check specific file
npx tsc --noEmit src/server.ts

# Run just the canvas server
node dist/server.js

# Run just the MCP server
node dist/index.js
```

## Architecture & Code Structure

### Type System
The codebase uses comprehensive TypeScript types defined in `src/types.ts`:
- `ExcalidrawElement` and its subtypes for all diagram elements
- `ServerElement` extends base elements with server metadata (createdAt, updatedAt)
- WebSocket message types for real-time synchronization
- API response types for REST endpoints

### MCP Tools Implementation
MCP tools are defined in `src/index.ts` using the `@modelcontextprotocol/sdk`. Key tools include:
- `create_element` - Creates individual Excalidraw elements
- `batch_create_elements` - Creates multiple elements in one call
- `update_element`, `delete_element` - Element CRUD operations
- `group_elements`, `align_elements` - Element organization

Each tool syncs with the canvas server via HTTP requests to `EXPRESS_SERVER_URL` (default: http://localhost:3000).

### Canvas Server Architecture
The Express server (`src/server.ts`) serves dual purposes:
1. Static file serving for the built React frontend (`dist/` directory)
2. REST API endpoints at `/api/elements` with WebSocket for real-time updates

WebSocket connections broadcast element changes to all connected clients using a `Map<string, WebSocket>` for client tracking.

### Frontend Integration
The React app (`frontend/src/App.tsx`) uses:
- Official `@excalidraw/excalidraw` package
- WebSocket client that reconnects automatically
- Dual-path element loading (API fetch + WebSocket updates)

## Important Implementation Details

- **Build Output**: TypeScript compiles to `dist/` directory. Both servers and types are compiled here.
- **Frontend Build**: Vite builds to `dist/` with entry point at `frontend/index.html`
- **Element IDs**: Generated using timestamp + random string pattern
- **Canvas Sync**: Controlled by `ENABLE_CANVAS_SYNC` environment variable
- **WebSocket Protocol**: Simple JSON messages with `type` and `data` fields
- **CORS**: Enabled on canvas server for cross-origin MCP requests

## Environment Variables

- `EXPRESS_SERVER_URL` - Canvas server URL (default: http://localhost:3000)
- `ENABLE_CANVAS_SYNC` - Enable/disable canvas sync (default: true)  
- `PORT` - Canvas server port (default: 3000)
- `DEBUG` - Enable debug logging (default: false)

## Common Development Tasks

### Adding a New MCP Tool
1. Define the tool schema in `src/index.ts` using zod
2. Implement the handler function
3. Register the tool in the server's tool list
4. Add corresponding API endpoint in `src/server.ts` if needed

### Modifying Element Types
1. Update type definitions in `src/types.ts`
2. Run `npm run type-check` to verify no type errors
3. Rebuild with `npm run build`

### Debugging Canvas Sync Issues
1. Check WebSocket connection in browser console
2. Verify `EXPRESS_SERVER_URL` is accessible
3. Check server logs for HTTP request errors
4. Ensure `ENABLE_CANVAS_SYNC=true`