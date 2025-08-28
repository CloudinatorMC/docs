# Management API

The Management API provides a WebSocket-based JSON-RPC interface for dynamically managing mc-router's routing configuration without restarting the service.

## Configuration

To enable the Management API, add the `management_listen` option to your `mc-router.toml` configuration file:

```toml
management_listen = "0.0.0.0:8080"
```

This will start the API server on the specified address and port.

## Connection

The API uses WebSocket connections for communication. Connect to the configured address using any WebSocket client:

```
ws://localhost:8080
```

## Protocol

The API implements the JSON-RPC 2.0 specification. All requests and responses are JSON objects.

### Request Format

```json
{
  "jsonrpc": "2.0",
  "method": "method_name",
  "params": { ... },
  "id": 1
}
```

### Response Format

```json
{
  "jsonrpc": "2.0",
  "result": { ... },
  "id": 1
}
```

### Error Response Format

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found"
  },
  "id": 1
}
```

## Methods

### ping

Tests connectivity to the API.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "ping",
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": "pong",
  "id": 1
}
```

### get_routes

Retrieves the current routing configuration.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "get_routes",
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "routes": {
      "lobby.example.com": {
        "address": "127.0.0.1:25566",
        "use_haproxy": true
      },
      "survival.example.com": {
        "address": "127.0.0.1:25567",
        "use_haproxy": false
      }
    },
    "default": {
      "address": "127.0.0.1:25566",
      "use_haproxy": true
    }
  },
  "id": 1
}
```

### replace_routes

Replaces the entire routing configuration with new routes.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "replace_routes",
  "params": {
    "routes": {
      "newserver.example.com": {
        "address": "127.0.0.1:25568",
        "use_haproxy": false
      }
    },
    "default": {
      "address": "127.0.0.1:25569",
      "use_haproxy": true
    }
  },
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": "ok",
  "id": 1
}
```

### merge_routes

Merges new routes into the existing configuration. Existing routes with the same key will be overwritten.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "merge_routes",
  "params": {
    "routes": {
      "newserver.example.com": {
        "address": "127.0.0.1:25568",
        "use_haproxy": false
      }
    }
  },
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": "ok",
  "id": 1
}
```

### clear_routes

Clears all routing rules. Only the default backend (if configured) will remain.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "clear_routes",
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": "ok",
  "id": 1
}
```

## Data Types

### BackendConfig

```json
{
  "address": "127.0.0.1:25566",
  "use_haproxy": true
}
```

- `address`: String - The backend server address in "host:port" format
- `use_haproxy`: Boolean - Whether to send HAProxy PROXY protocol headers

## Error Codes

- `-32700`: Parse error - Invalid JSON
- `-32600`: Invalid Request - Not a valid JSON-RPC 2.0 request
- `-32601`: Method not found - Unknown method
- `-32602`: Invalid params - Parameters don't match expected format

## Usage Examples

### Using WebSocket in JavaScript

```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  // Ping test
  ws.send(JSON.stringify({
    jsonrpc: '2.0',
    method: 'ping',
    id: 1
  }));
};

ws.onmessage = (event) => {
  const response = JSON.parse(event.data);
  console.log('Response:', response);
};
```

### Using curl with websocat

```bash
# Install websocat first
# Then send a ping request
echo '{"jsonrpc": "2.0", "method": "ping", "id": 1}' | websocat ws://localhost:8080
```

### Programmatic Configuration Update

```javascript
// Get current routes
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'get_routes',
  id: 1
}));

// Add a new route
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'merge_routes',
  params: {
    routes: {
      'newserver.example.com': {
        address: '127.0.0.1:25570',
        use_haproxy: false
      }
    }
  },
  id: 2
}));
```

## Notes

- All route keys are automatically converted to lowercase
- Changes take effect immediately without restarting mc-router
- The API is designed for programmatic access and automation
- Consider securing the WebSocket endpoint in production environments using a reverse proxy