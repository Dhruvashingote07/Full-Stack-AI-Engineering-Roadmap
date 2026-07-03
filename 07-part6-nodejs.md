# Part 6: Node.js

---

## Chapter 46: Node.js Runtime

### Introduction

Node.js is a JavaScript runtime built on Chrome's V8 engine. It enables server-side JavaScript with an event-driven, non-blocking I/O model.

> 🏛️ **Real World Analogy:** Node.js is like a restaurant with one waiter (single thread) who doesn't wait for food to cook — he takes orders, hands them to the kitchen (async I/O), and serves them when ready. This event-driven model handles thousands of concurrent diners efficiently without needing a waiter per table (multi-threading).

### Why It Matters

Node.js unified frontend and backend development under one language. Its package ecosystem (npm) is the largest software registry in the world.

### Architecture

- **V8** — JavaScript engine (compiles JS to machine code)
- **libuv** — Asynchronous I/O (event loop, thread pool)
- **Node.js API** — s, http, path, crypto, etc.

### Event Loop Phases

`
   +---------------------------+
+->|          timers           |
|  +-------------+------------+
|  +-------------+------------+
|  |     pending callbacks     |
|  +-------------+------------+
|  +-------------+------------+
|  |       idle, prepare       |
|  +-------------+------------+
|  +-------------+------------+
|  |           poll            |
|  +-------------+------------+
|  +-------------+------------+
|  |          check            |
|  +-------------+------------+
|  +-------------+------------+
|  |      close callbacks      |
|  +---------------------------+
+------------------------------+
`

> ⚠️ **Warning:** Never block the event loop with synchronous `fs` methods like `fs.readFileSync()` in production. Synchronous methods freeze the entire server — no requests can be processed until the file read completes. Always use the promise-based `fs/promises` API for non-blocking I/O.

### Core Modules

`javascript
import { readFile, writeFile } from \"node:fs/promises\";
const data = await readFile(\"./data.json\", \"utf-8\");
await writeFile(\"./output.txt\", \"Hello World\");

import { join, resolve } from \"node:path\";
const fullPath = join(\"src\", \"index.js\");

import { createServer } from \"node:http\";
const server = createServer((req, res) => {
  res.writeHead(200, { \"Content-Type\": \"application/json\" });
  res.end(JSON.stringify({ status: \"ok\" }));
});
server.listen(3000);
`

### npm

`ash
npm init -y
npm install express
npm install -D typescript @types/node
npm ci
npx tsc --init
npm run dev
`

### package.json Fields

| Field | Purpose |
|-------|---------|
| 
ame | Package name, lowercase + hyphens |
| ersion | Semver |
| main | Entry point |
| 	ype | \"module\" for ESM, \"commonjs\" (default) |
| scripts | Run commands with 
pm run |
| dependencies | Production packages |
| devDependencies | Build/dev-only packages |

### Common Mistakes

- Blocking the event loop with synchronous methods
- Not handling promise rejections (unhandledRejection)
- Using process.exit() without cleanup
- Ignoring backpressure in streams

> ✅ **Best Practice:** Always handle unhandled promise rejections with a process-level handler: `process.on('unhandledRejection', (err) => { logger.error(err); process.exit(1); })`. In Node.js 15+, unhandled rejections crash the process by default.

### Best Practices

- Always use the promise-based s/promises API
- Handle all errors with proper try/catch
- Use pino or winston for structured logging
- Set NODE_ENV=production for production deployments

### 📝 Revision Notes

- Node.js runs on V8 + libuv with a single-threaded event loop
- The event loop phases: timers → pending callbacks → poll → check → close callbacks
- Always use `fs/promises` over `fs` sync methods to avoid blocking
- npm is the largest package registry; prefer `npm ci` over `npm install` in CI
- Set `NODE_ENV=production` and use structured logging in production

### 🛠️ Practical Exercises

1. **Event loop experiment** — Write a program that demonstrates each phase of the event loop using `setTimeout`, `setImmediate`, `process.nextTick`, and `Promise.resolve()` — observe the execution order
2. **File reader with streams** — Build a program that reads a large file using streams instead of `readFile` and measures the memory difference
3. **CLI tool with npm** — Create a simple CLI tool, publish it to a local npm registry (Verdaccio), and practice `npm link` for local development
4. **HTTP server from scratch** — Build a simple HTTP server using only `node:http` without Express, handling routing and JSON responses manually

---

## Chapter 47: Express.js

### Introduction

Express.js is the most widely used web framework for Node.js. It provides minimal, flexible features for building web applications and APIs.

> 🏛️ **Real World Analogy:** Express.js is like the highway system for your Node.js traffic. It provides the on-ramps (routes), toll booths (middleware), and exits (responses). Without it, you'd be building dirt roads with `node:http` — functional but much more work.

### Basic Server

`javascript
import express from \"express\";

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());
app.use(cors());

app.get(\"/\", (req, res) => {
  res.json({ message: \"Hello World\" });
});

app.listen(PORT, () => console.log(\Server running on port \\));
`

> 💡 **Pro Tip:** Use `express-async-errors` library or create a wrapper function for async route handlers to catch rejected promises automatically. Without this, an unhandled promise rejection in an async route will crash the process. Express 5 will include native async error handling, but for Express 4 you need the wrapper.

### Routing

`javascript
import { Router } from \"express\";
const userRouter = Router();

userRouter.get(\"/\", async (req, res) => {
  const users = await db.listUsers(req.query);
  res.json(users);
});

userRouter.get(\"/:id\", async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) return res.status(404).json({ error: \"Not found\" });
  res.json(user);
});

userRouter.post(\"/\", async (req, res) => {
  const user = await db.createUser(req.body);
  res.status(201).json(user);
});

userRouter.put(\"/:id\", async (req, res) => {
  const updated = await db.updateUser(req.params.id, req.body);
  res.json(updated);
});

userRouter.delete(\"/:id\", async (req, res) => {
  await db.deleteUser(req.params.id);
  res.status(204).end();
});

app.use(\"/api/users\", userRouter);
`

### Middleware System

`javascript
const logger = (req, res, next) => {
  const start = Date.now();
  res.on(\"finish\", () => {
    console.log(\\ \ \ \ms\);
  });
  next();
};

const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  if (err.name === \"ValidationError\") return res.status(400).json({ error: err.message });
  res.status(500).json({ error: \"Internal server error\" });
};

app.use(logger);
app.use(errorHandler);
`

> ⚠️ **Warning:** Middleware order matters in Express. Error-handling middleware (with 4 parameters) must be registered LAST. If you place it before routes, errors won't be caught. Always put `app.use(errorHandler)` after all routes and other middleware.

### Async Handler

`javascript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get(\"/api/users/:id\", asyncHandler(async (req, res) => {
  const user = await db.findUser(req.params.id);
  if (!user) return res.status(404).json({ error: \"Not found\" });
  res.json(user);
}));
`

> ⚠️ **Warning:** Exposing stack traces in production error responses is a security vulnerability — it reveals internal application structure to attackers. Always check `app.get('env') !== 'production'` before including stack traces, or use a dedicated error-handling middleware that strips them in production.

### Response Methods

| Method | Purpose |
|--------|---------|
| 
es.json(body) | JSON response |
| 
es.status(code) | Set status code |
| 
es.send(body) | String/Buffer/JSON auto-detect |
| 
es.redirect(url) | 302 redirect |
| 
es.sendFile(path) | File stream |
| 
es.type(type) | Set Content-Type |

> ❓ **Interview Question:** How does Express middleware work? Middleware functions have access to `req`, `res`, and `next`. They can execute code, modify `req`/`res`, end the request-response cycle, or call `next()` to pass control to the next middleware. Middleware runs in the order it's registered — this is why error handlers must be last.

### Common Mistakes

- Not ordering middleware correctly (error handler last)
- Forgetting express.json() and receiving undefined body
- Mixing sync and async error handling
- Exposing stack traces in production

### Best Practices

- Use the router pattern for organized routes
- Implement centralized error handling middleware
- Validate input with zod or joi
- Use helmet for security headers
- Rate-limit with express-rate-limit

### 📝 Revision Notes

- Express middleware runs in registration order; error handlers must be last
- Wrap async route handlers to catch rejected promises (Express 5 handles this natively)
- Use `Router()` for modular, organized route definitions
- Always parse JSON bodies with `express.json()` and validate input
- Secure Express apps with `helmet`, `cors`, and `express-rate-limit`

### 🛠️ Practical Exercises

1. **REST API with Express** — Build a complete CRUD API for a resource (e.g., books) with Express Router, input validation with Zod, and centralized error handling
2. **Middleware chain** — Create custom middleware for request logging, rate limiting (in-memory), and authentication, then compose them in the correct order
3. **Express error handling** — Build an Express app with custom error classes, an error-handling middleware that formats errors consistently, and async wrappers
4. **File server with Express** — Create an Express server that serves static files, handles file uploads with multer, and streams large files

---

## Chapter 48: REST APIs

### Introduction

REST (Representational State Transfer) is an architectural style for designing networked applications using HTTP methods.

> 🏛️ **Real World Analogy:** REST APIs are like a library catalog. Each book (resource) has a unique shelf location (URL /books/:id). You browse the catalog (GET), add a new book (POST), update a book's description (PATCH), replace a book entirely (PUT), or remove it (DELETE). The librarian (server) doesn't remember who you are between visits — that's statelessness.

### REST Principles

1. **Stateless** — Each request contains all necessary information
2. **Resource-based** — URLs represent resources, not actions
3. **HTTP methods** — GET, POST, PUT, PATCH, DELETE
4. **Uniform interface** — Consistent patterns across the API

> ❓ **Interview Question:** What is the difference between PUT and PATCH? PUT replaces the entire resource — if you omit a field, it gets reset to default or removed. PATCH applies a partial update — only the fields you send are modified. Use PUT for full replacements and PATCH for partial updates. In practice, many APIs use PATCH exclusively for updates to avoid accidental data loss.

### RESTful URL Design

| HTTP Method | URL | Action | Status Code |
|-------------|-----|--------|-------------|
| GET | /api/users | List users | 200 |
| GET | /api/users/:id | Get user by ID | 200 |
| POST | /api/users | Create user | 201 |
| PUT | /api/users/:id | Replace user | 200 |
| PATCH | /api/users/:id | Partial update | 200 |
| DELETE | /api/users/:id | Delete user | 204 |

### Status Codes by Category

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid authentication |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate, version conflict |
| 422 | Unprocessable Entity | Semantic errors |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server failure |

> ⚠️ **Warning:** Never return 500 for validation errors — use 400 Bad Request. Similarly, don't return 401 when the user is authenticated but lacks permissions; that's 403 Forbidden. Using correct HTTP status codes makes your API predictable and easier for clients to handle programmatically.

### Pagination, Filtering, Sorting

`javascript
app.get(\"/api/users\", async (req, res) => {
  const { page = 1, pageSize = 10, sort = \"createdAt\", order = \"desc\" } = req.query;
  const skip = (Number(page) - 1) * Number(pageSize);

  const [users, total] = await Promise.all([
    db.find({}).sort({ [sort]: order }).skip(skip).limit(Number(pageSize)),
    db.countDocuments({}),
  ]);

  res.json({
    data: users,
    meta: { page: Number(page), pageSize: Number(pageSize), total },
  });
});
`

### Common Mistakes

- Using GET for mutating operations
- Returning 500 for validation errors (use 400)
- Not implementing proper pagination
- Inconsistent error response format

> ✅ **Best Practice:** Always use plural nouns for collection endpoints (`/api/users`, `/api/posts`), not verbs (`/api/getUsers`). Use HTTP methods to express actions. For nested resources, use `/api/users/:id/posts` rather than query parameters. Consistency across your API is more important than which convention you choose.

### Best Practices

- Use plural nouns for collection endpoints
- Nest related resources logically
- Provide consistent error responses
- Implement rate limiting and pagination from day one
- Document with OpenAPI/Swagger

### 📝 Revision Notes

- REST uses HTTP methods as verbs: GET (read), POST (create), PUT (replace), PATCH (update), DELETE (delete)
- Use correct status codes: 200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Error
- Always paginate collection endpoints; use cursor-based pagination for large datasets
- Consistent error responses with a standard format make API clients simpler
- Document APIs with OpenAPI/Swagger for developer experience

### 🛠️ Practical Exercises

1. **RESTful API design** — Design and implement a REST API for a blogging platform with users, posts, comments, and tags using proper URL patterns and status codes
2. **Pagination implementation** — Implement cursor-based pagination for a large dataset and compare it with offset-based pagination (trade-offs)
3. **API error standardization** — Create a standardized error response format (`{ error: { code, message, details } }`) and implement it across all endpoints
4. **OpenAPI documentation** — Write an OpenAPI 3.0 spec for an existing API and generate interactive Swagger UI documentation

---

## Chapter 49: Authentication — JWT & Passport.js

### Introduction

Authentication verifies who a user is; authorization determines what they can do.

> 🏛️ **Real World Analogy:** JWT tokens are like a digitally signed ID card. The ID card contains your name and photo (claims like `sub`, `role`), and it's stamped with an official seal (digital signature). Anyone can read the card (Base64-decoded), but only the issuing authority can create valid ones. The card expires (15 min for access tokens) so you must periodically get a new one.

### JWT Structure

A JWT consists of three Base64URL-encoded segments separated by dots: header.payload.signature

### JWT Implementation

`javascript
import jwt from \"jsonwebtoken\";
import bcrypt from \"bcrypt\";

const SECRET = process.env.JWT_SECRET;

function generateAccessToken(user) {
  return jwt.sign({ sub: user.id, role: user.role }, SECRET, { expiresIn: \"15m\" });
}

function generateRefreshToken(user) {
  return jwt.sign({ sub: user.id }, process.env.JWT_REFRESH_SECRET, { expiresIn: \"7d\" });
}

function verifyAccessToken(token) {
  try {
    return jwt.verify(token, SECRET);
  } catch (error) {
    if (error.name === \"TokenExpiredError\") throw new Error(\"Token expired\");
    throw new Error(\"Invalid token\");
  }
}

const hash = await bcrypt.hash(password, 12);
const valid = await bcrypt.compare(password, hash);
`

### Passport.js

`javascript
import passport from \"passport\";
import { Strategy as JwtStrategy, ExtractJwt } from \"passport-jwt\";
import { Strategy as LocalStrategy } from \"passport-local\";

const jwtOptions = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
};

passport.use(new JwtStrategy(jwtOptions, async (payload, done) => {
  try {
    const user = await db.findUser(payload.sub);
    return done(null, user || false);
  } catch (error) {
    return done(error, false);
  }
}));

passport.use(new LocalStrategy({ usernameField: \"email\" }, async (email, password, done) => {
  try {
    const user = await db.findUserByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.password))) {
      return done(null, false, { message: \"Invalid credentials\" });
    }
    return done(null, user);
  } catch (error) {
    return done(error);
  }
}));
`

> ⚠️ **Warning:** Never store plain-text passwords. Always hash with bcrypt (cost 12+) or argon2id. A data breach with plain-text passwords compromises every user — especially those who reuse passwords across services. Also, never include sensitive data (like passwords or credit cards) in JWT payloads — they're only Base64-encoded, not encrypted.

> 💡 **Pro Tip:** Use separate secrets for access tokens (`JWT_SECRET`) and refresh tokens (`JWT_REFRESH_SECRET`). This limits damage if one secret is compromised. Store them in environment variables, never in code. Generate strong secrets with: `openssl rand -base64 64` or use a secrets manager like AWS Secrets Manager or HashiCorp Vault.

### Authentication Best Practices

| Practice | Why |
|----------|-----|
| Hash passwords with bcrypt (cost 12+) | Prevent rainbow table attacks |
| Use short-lived access tokens (15 min) | Limit damage if stolen |
| Implement refresh token rotation | Invalidate old refresh tokens |
| Store refresh tokens securely | httpOnly + Secure + SameSite cookies |
| Rate-limit login endpoints | Prevent brute force |

### Common Mistakes

- Storing plain-text passwords
- Using weak JWT secrets
- Not validating token expiration server-side
- Including sensitive data in JWT payload

> ❓ **Interview Question:** How does JWT authentication work end-to-end? The user logs in with credentials; the server validates them and issues a signed JWT (access token + refresh token). The client sends the access token in the `Authorization: Bearer <token>` header on subsequent requests. The server verifies the signature and expiration on every request. When the access token expires, the client uses the refresh token to get a new one without re-authentication.

### Best Practices

- Use well-vetted libraries
- Store JWT secret securely
- Implement token blacklisting for immediate revocation
- Use rgon2id over bcrypt for new projects
- Apply principle of least privilege to roles

### 📝 Revision Notes

- JWT has three parts: header (algorithm), payload (claims), signature (verification)
- Access tokens are short-lived (15 min); refresh tokens last longer (7 days)
- Always hash passwords with bcrypt (cost 12+) or argon2id
- Passport.js strategies: JWT (Bearer token), Local (username/password), OAuth
- Implement refresh token rotation to detect and prevent token theft

### 🛠️ Practical Exercises

1. **JWT auth server** — Build an Express server with register, login, refresh, and logout endpoints using JWT access + refresh tokens
2. **Passport.js integration** — Add Passport.js with JWT and Local strategies to an existing Express app, including protected routes
3. **Token blacklist** — Implement a token blacklist using Redis that allows immediate revocation of tokens before their expiration
4. **Role-based authorization** — Create a middleware that checks user roles and restricts access to admin-only endpoints, with proper 403 responses

---

## Chapter 50: NestJS

### Introduction

NestJS is a progressive Node.js framework for building efficient, scalable server-side applications using TypeScript by default.

> 🏛️ **Real World Analogy:** NestJS is like the architect-designed blueprint of a house, while Express is a pile of lumber and tools. NestJS imposes structure (modules, controllers, services, decorators) that makes large codebases maintainable. Express gives you freedom but requires you to create your own architecture. Choose NestJS for enterprise applications with multiple developers.

### Core Concepts

`	ypescript
import { NestFactory } from \"@nestjs/core\";
import { ValidationPipe } from \"@nestjs/common\";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }));
  app.enableCors();
  app.setGlobalPrefix(\"api/v1\");
  await app.listen(3000);
}
bootstrap();

@Module({
  imports: [UsersModule, DatabaseModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
`

> 💡 **Pro Tip:** Use class-validator decorators (`@IsString()`, `@IsEmail()`, `@MinLength()`) on your DTO classes combined with NestJS's `ValidationPipe` to automatically validate incoming requests. With `whitelist: true` and `forbidNonWhitelisted: true`, any extra properties are stripped or rejected — this prevents mass-assignment vulnerabilities.

### Controllers

`	ypescript
@Controller(\"users\")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() { return this.usersService.findAll(); }

  @Get(\":id\")
  @UseGuards(JwtAuthGuard)
  findOne(@Param(\"id\") id: string) { return this.usersService.findOne(id); }

  @Post()
  create(@Body() createUserDto: CreateUserDto) { return this.usersService.create(createUserDto); }
}
`

### Providers & Services

`	ypescript
@Injectable()
export class UsersService {
  constructor(@InjectModel(User.name) private userModel: Model<User>) {}

  async findAll(): Promise<User[]> { return this.userModel.find().exec(); }

  async findOne(id: string): Promise<User> {
    const user = await this.userModel.findById(id).exec();
    if (!user) throw new NotFoundException(\User #\ not found\);
    return user;
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    return new this.userModel(createUserDto).save();
  }
}
`

> ✅ **Best Practice:** Always validate file types on the server using both the MIME type and file extension check. Even better, check the file's magic bytes (first few bytes of the file) to confirm the actual content type. A user can rename `virus.exe` to `photo.jpg` and bypass extension-only checks.

### Key Decorators

| Decorator | Purpose |
|-----------|---------|
| @Module | Organize code into modules |
| @Injectable | Mark class as provider |
| @Controller | Define route handler |
| @Get, @Post, @Put, @Delete | HTTP method handlers |
| @Body, @Param, @Query | Extract request data |
| @UseGuards | Apply guards |
| @UsePipes | Apply validation pipes |
| @UseInterceptors | Apply interceptors |

> ❓ **Interview Question:** Explain NestJS's dependency injection system. Providers decorated with `@Injectable()` are registered in the DI container. When a controller or service declares a constructor parameter, NestJS resolves the dependency from the container. Providers can be scoped as DEFAULT (singleton — shared across requests), REQUEST (new instance per request), or TRANSIENT (new instance per injection). This enables clean, testable architecture.

### Common Mistakes

- Making all providers singletons when request-scoped is needed
- Not handling circular dependencies between modules
- Bypassing DI container with 
ew Service()

> 💡 **Pro Tip:** Use `@nestjs/testing` for integration tests. It creates a NestJS testing module with mocked dependencies, allowing you to test controllers and services in isolation without spinning up a full server. This catches DI configuration issues early.

### Best Practices

- Keep modules focused on a single domain
- Use DTOs with class-validator for request validation
- Configure global pipes, filters, and interceptors
- Write integration tests with @nestjs/testing

### 📝 Revision Notes

- NestJS uses modules, controllers, providers, and decorators for a structured architecture
- DI container manages provider instantiation and lifecycle
- DTOs with `class-validator` and `ValidationPipe` provide automatic request validation
- Guards handle authentication, interceptors handle cross-cutting concerns (logging, transformation)
- Use `@nestjs/testing` for integration tests with mocked dependencies

### 🛠️ Practical Exercises

1. **NestJS CRUD module** — Generate a complete CRUD module for a resource using `@nestjs/cli` with DTO validation, a service, and a controller
2. **Custom guard** — Create a custom `@Roles()` decorator and `RolesGuard` that restricts access based on user roles
3. **NestJS with Prisma** — Integrate Prisma ORM with NestJS, create a module with dependency injection, and write integration tests
4. **Global exception filter** — Implement a global exception filter that catches all exceptions and returns a standardized error response format

---

## Chapter 51: Socket.IO

### Introduction

Socket.IO enables real-time, bidirectional, event-based communication between web clients and servers.

> 🏛️ **Real World Analogy:** Socket.IO is like a walkie-talkie network. When you press the talk button (emit an event), everyone tuned to your channel (room) hears you immediately. Unlike HTTP request-response (which is like mailing a letter), WebSockets maintain an open line so messages flow both ways in real time.

### Server Setup

`javascript
import { Server } from \"socket.io\";
import http from \"http\";

const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: process.env.CLIENT_URL, methods: [\"GET\", \"POST\"] },
});

io.on(\"connection\", (socket) => {
  console.log(\User connected: \\);

  socket.on(\"disconnect\", (reason) => {
    console.log(\User disconnected: \ (\)\);
  });
});

server.listen(3000);
`

### Client Setup

`javascript
import { io } from \"socket.io-client\";

const socket = io(\"http://localhost:3000\", {
  auth: { token: \"jwt_token_here\" },
  autoConnect: false,
});

socket.on(\"connect\", () => console.log(\"Connected:\", socket.id));
socket.on(\"connect_error\", (error) => console.error(\"Connection error:\", error.message));
socket.connect();
`

> ⚠️ **Warning:** Socket.IO connections can rapidly exhaust server resources if not limited. Always implement connection limits per IP and authentication middleware. Monitor open socket counts and set `maxHttpBufferSize` to prevent malicious clients from sending large payloads that consume memory.

### Events & Rooms

`javascript
io.on(\"connection\", (socket) => {
  socket.join(\user:\\);
  socket.join(\chat:\\);

  socket.on(\"chat:message\", async ({ chatId, text }) => {
    const message = await saveMessage({ userId: socket.userId, chatId, text });
    io.to(\chat:\\).emit(\"chat:message\", message);
  });

  socket.on(\"chat:typing\", ({ chatId }) => {
    socket.to(\chat:\\).emit(\"chat:typing\", { userId: socket.userId });
  });
});

// Broadcasting
io.emit(\"system:announcement\", { message: \"Server maintenance in 5 min\" });
socket.broadcast.emit(\"user:online\", { userId: socket.userId });
io.to(\"room:general\").emit(\"message\", data);
`

### Authentication Middleware

`javascript
io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token;
    if (!token) throw new Error(\"Authentication required\");
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.userId = decoded.sub;
    next();
  } catch (error) {
    next(new Error(\"Authentication failed\"));
  }
});
`

### Common Socket.IO Patterns

| Pattern | Description | Server API |
|---------|-------------|------------|
| Emit | Send event | socket.emit(event, data) |
| On | Receive event | socket.on(event, callback) |
| Broadcast | Send to all except sender | socket.broadcast.emit(event, data) |
| To Room | Send to room members | io.to(room).emit(event, data) |
| Volatile | Fire-and-forget | socket.volatile.emit(event, data) |

> ⚠️ **Warning:** Don't rely on socket IDs for identifying users — they change on every reconnection. Instead, store a mapping of userId → socketId in Redis (or your database). When a user reconnects, update the mapping. This ensures you can send messages to the correct socket after reconnections or when running multiple server instances.

### Scaling Socket.IO

`javascript
import { createAdapter } from \"@socket.io/redis-adapter\";
import Redis from \"ioredis\";

const pubClient = new Redis(process.env.REDIS_URL);
const subClient = pubClient.duplicate();
io.adapter(createAdapter(pubClient, subClient));
`

### Common Mistakes

- Not handling disconnection / reconnection properly
- Emitting too many events per second
- Storing socket IDs that don't survive reconnection
- Forgetting error handling middleware

> 💡 **Pro Tip:** Debounce server-side events emitted at high frequency, like typing indicators. Instead of emitting on every keystroke, buffer updates and emit every 300ms. This reduces server load and network traffic dramatically without degrading the user experience. Also use `socket.volatile.emit()` for non-critical events that can be dropped under load.

### Best Practices

- Authenticate on connection via middleware
- Use namespaces for logical separation
- Store socket-to-user mapping in Redis for horizontal scaling
- Debounce high-frequency events (typing indicators)
- Monitor and limit event rate per socket

### 📝 Revision Notes

- Socket.IO adds real-time bidirectional communication over WebSocket with fallbacks
- Use rooms for group communication: `socket.join('room')`, `io.to('room').emit()`
- Authenticate connections via middleware using JWT tokens
- For horizontal scaling, use Redis adapter with `@socket.io/redis-adapter`
- Debounce high-frequency events and use volatile events for non-critical data

### 🛠️ Practical Exercises

1. **Real-time chat app** — Build a chat application with Socket.IO supporting multiple rooms, typing indicators, message history, and online user lists
2. **Live notifications** — Implement a real-time notification system where the server pushes notifications to specific users via Socket.IO rooms
3. **Socket.IO with Redis** — Scale a Socket.IO app horizontally using the Redis adapter with two server instances and test cross-instance communication
4. **Collaborative editing** — Build a collaborative text editor where multiple users can edit the same document in real time with operational transforms

---

## Chapter 52: File Upload

### Introduction

File upload is handled through middleware like multer for multipart form data.

> 🏛️ **Real World Analogy:** File upload with multer is like airport security screening. The file goes through a scanner (file filter), gets weighed (size limit), tagged with a unique ID (UUID filename), and sent to a secure storage area (uploads/ folder or cloud storage). Always validate what comes through — treat every file as potentially dangerous until inspected.

> ✅ **Best Practice:** For production file uploads, use cloud storage (S3, Cloud Storage, Azure Blob) instead of local disk storage. Local storage doesn't scale across multiple server instances, requires backup management, and can fill up disks. Use `multer-s3` or `multer-storage-cloudinary` to stream uploads directly to cloud storage.

### Multer

`javascript
import multer from \"multer\";
import path from \"path\";
import { v4 as uuidv4 } from \"uuid\";

const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, \"uploads/\"),
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname);
    cb(null, \\\\);
  },
});

const fileFilter = (req, file, cb) => {
  const allowed = [\".jpg\", \".jpeg\", \".png\", \".gif\", \".webp\"];
  const ext = path.extname(file.originalname).toLowerCase();
  if (allowed.includes(ext)) cb(null, true);
  else cb(new Error(\"Only image files are allowed\"), false);
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024, files: 5 },
});
`

### Upload Endpoints

`javascript
// Single file
app.post(\"/api/upload/avatar\", upload.single(\"avatar\"), async (req, res) => {
  if (!req.file) return res.status(400).json({ error: \"No file uploaded\" });
  const fileUrl = \/uploads/\\;
  res.json({ url: fileUrl, filename: req.file.originalname, size: req.file.size });
});

// Multiple files
app.post(\"/api/upload/photos\", upload.array(\"photos\", 10), async (req, res) => {
  const files = req.files.map((f) => ({ url: \/uploads/\\, size: f.size }));
  res.json({ files });
});
`

### Cloud Upload (S3)

`javascript
import { S3Client, PutObjectCommand } from \"@aws-sdk/client-s3\";

const s3 = new S3Client({ region: process.env.AWS_REGION });

app.post(\"/api/upload/s3\", upload.single(\"file\"), async (req, res) => {
  const key = \uploads/\-\\;
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET, Key: key,
    Body: req.file.buffer, ContentType: req.file.mimetype,
  }));
  res.json({ url: \https://\.s3.amazonaws.com/\\ });
});
`

### Common Mistakes

- Not validating file types on the server
- Allowing arbitrary file paths (path traversal attack)
- Storing files without size limits
- Using sequential or user-provided filenames

> ❓ **Interview Question:** How would you handle file upload for a production application? Validate file type, size, and magic bytes server-side. Generate UUID filenames to prevent collisions and path traversal. Store files in cloud storage (S3) rather than the server filesystem. For large files, use multipart upload or presigned URLs. Scan all uploads for malware. Add rate limiting per user to prevent abuse.

### Best Practices

- Always validate on the server (MIME type, size, magic bytes)
- Generate unique filenames (UUID)
- Store files in cloud storage (S3, Cloud Storage)
- Scan uploaded files for malware
- Use presigned URLs for direct client-to-cloud uploads
- Add rate limiting on upload endpoints

### 📝 Revision Notes

- Multer handles multipart/form-data uploads in Express
- Validate file type by extension, MIME type, and magic bytes
- Generate UUID-based filenames to prevent collisions and path traversal
- Cloud storage (S3, Cloud Storage) is preferred over local filesystem for production
- Presigned URLs enable direct client-to-cloud uploads, reducing server load
- Always set file size limits and rate limit upload endpoints

### 🛠️ Practical Exercises

1. **Avatar upload endpoint** — Build an Express endpoint that uploads a single avatar image with validation (type, size, dimensions) and returns a URL
2. **Multer with S3** — Modify the upload endpoint to store files directly in S3 using `multer-s3` instead of local disk storage
3. **Presigned URL upload** — Implement a flow where the client requests a presigned URL from the server and uploads directly to S3
4. **File upload with progress** — Build an upload endpoint that emits progress events via Socket.IO so the client can show a progress bar

---

## Chapter 53: Node.js Deployment

### Introduction

Deploying a Node.js application involves process management, reverse proxy, environment configuration, and production-grade reliability.

> 🏛️ **Real World Analogy:** Deploying a Node.js app is like launching a restaurant. PM2 is the restaurant manager who keeps the kitchen running (auto-restart crashes, cluster mode for multiple chefs). Nginx is the host/hostess who greets customers, handles SSL (security), and directs traffic to the right chef. Docker is the standardized kitchen blueprint that works the same in any building.

### Environment Configuration

`javascript
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:pass@host:5432/db
REDIS_URL=redis://:password@host:6379
JWT_SECRET=<long-random-string>
`

### Process Management with PM2

`ash
npm install -g pm2
pm2 start dist/index.js --name my-app -i max
pm2 list
pm2 logs
pm2 monit
pm2 reload my-app          # zero-downtime reload
pm2 save
pm2 startup                # restart on server boot
`

`javascript
module.exports = {
  apps: [{
    name: \"my-app\",
    script: \"dist/index.js\",
    instances: \"max\",
    exec_mode: \"cluster\",
    env: { NODE_ENV: \"production\" },
    max_memory_restart: \"1G\",
  }],
};
`

> ✅ **Best Practice:** Always set `NODE_ENV=production` in your production environment. This enables Express's built-in optimizations — stack traces are hidden in error responses, view caching is enabled, and some libraries switch to optimized production paths. Forgetting this is one of the most common deployment mistakes.

### Reverse Proxy with Nginx

`
ginx
upstream node_app {
    least_conn;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    location / {
        proxy_pass http://node_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \;
        proxy_set_header Connection \"upgrade\";
        proxy_set_header Host \System.Management.Automation.Internal.Host.InternalHost;
        proxy_set_header X-Real-IP \;
        proxy_set_header X-Forwarded-For \;
        proxy_read_timeout 60s;
    }

    location /socket.io/ {
        proxy_pass http://node_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \;
        proxy_set_header Connection \"upgrade\";
    }

    client_max_body_size 10M;
}
`

### Docker Deployment

`dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:22-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
ENV NODE_ENV=production
CMD [\"node\", \"dist/index.js\"]
`

> ✅ **Best Practice:** Containerize your Node.js application with Docker for consistent deployment across environments. Use multi-stage builds (builder stage + runtime stage) to keep the final image small — the runtime image needs only `dist/` and `node_modules/`, not the TypeScript source, dev dependencies, or build tools.

### Health Checks

`javascript
app.get(\"/health\", async (req, res) => {
  const health = {
    status: \"ok\",
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
  };
  health.database = await checkDatabase() ? \"ok\" : \"error\";
  res.status(health.database === \"ok\" ? 200 : 503).json(health);
});
`

> ✅ **Best Practice:** Implement graceful shutdown in your Node.js app. Listen for the `SIGTERM` signal (sent by PM2, Docker, and cloud orchestrators), stop accepting new requests, finish in-flight requests within a timeout (e.g., 30s), then close database connections and exit. This prevents dropped requests during deployment rollouts and scaling events.

### Deployment Platforms Comparison

| Platform | Type | Scaling | Difficulty | Cost |
|----------|------|---------|------------|------|
| VPS (DigitalOcean) | VM | Manual | Medium | $ |
| PaaS (Heroku, Railway) | PaaS | Auto | Low |  |
| Container (ECS, EKS) | Container | Auto | High | $ |
| Serverless (Lambda) | FaaS | Auto | Low | $ (usage) |
| Edge (Vercel, Netlify) | Edge/PaaS | Auto | Very Low |  |

> ⚠️ **Warning:** Never run Node.js as root — it's a major security risk. If your app is compromised, the attacker gains root access. Create a dedicated user with minimal permissions. In Docker, use `USER appuser` after creating a non-root user. On VPS, use `adduser` and run PM2 under that user.

### Common Mistakes

- Running Node.js as root
- Not setting NODE_ENV=production
- Using 
pm install instead of 
pm ci
- Not configuring log rotation
- Using console.log instead of structured logging

> ✅ **Best Practice:** Use `npm ci` instead of `npm install` in CI/CD pipelines. It installs exact versions from `package-lock.json` (no version resolution), fails if the lock file is out of sync, and is significantly faster. This ensures reproducible builds across environments.

### Best Practices

- Always run as non-root user
- Use a process manager (PM2) in cluster mode
- Set up health check and readiness endpoints
- Configure reverse proxy (Nginx) for SSL and caching
- Use structured logging (JSON format)
- Implement graceful shutdown (SIGTERM handler)

### 📝 Revision Notes

- Use PM2 in cluster mode for multi-core utilization and zero-downtime reloads
- Nginx handles SSL termination, load balancing, static file caching, and WebSocket upgrades
- Multi-stage Docker builds produce smaller, more secure images
- Always implement health check endpoints for container orchestrators
- Use `npm ci` for deterministic builds; set `NODE_ENV=production` for performance optimizations
- Implement graceful shutdown: listen for SIGTERM, close connections, finish in-flight requests

### 🛠️ Practical Exercises

1. **PM2 cluster deployment** — Deploy a Node.js app with PM2 in cluster mode, configure `ecosystem.config.js`, and verify zero-downtime reload works
2. **Docker multi-stage build** — Create a multi-stage Dockerfile that builds and runs a Node.js app as a non-root user, then deploy it with docker-compose
3. **Nginx reverse proxy** — Configure Nginx as a reverse proxy in front of a Node.js app with SSL certificates (Let's Encrypt), caching, and WebSocket proxying
4. **CI/CD pipeline** — Set up a GitHub Actions workflow that runs tests, builds the Docker image, and deploys to a VPS or cloud provider
5. **Production monitoring** — Add structured logging (pino), application metrics (prom-client), and health checks to an existing Node.js app

---
