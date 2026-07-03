# Part 16: Software Engineering

## Chapter 1: System Design — Requirements and Capacity

System design is the process of defining the architecture, components, modules, interfaces, and data for a system.

### Requirements Gathering

```
Functional Requirements → What the system does
Non-Functional Requirements → How the system performs
```

#### Functional Requirements

```
### URL Shortener - Functional Requirements
1. Users can create short URLs from long URLs
2. Users can access original URL via short URL
3. Short URLs expire after configurable time
4. Users can track click statistics
5. Custom alias support for premium users
```

#### Non-Functional Requirements

```
### URL Shortener - Non-Functional Requirements
- Availability: 99.99% uptime (52.56 min downtime/year)
- Latency: p99 < 100ms for redirect
- Throughput: 10M URL creations/day, 100M redirects/day
- Consistency: Eventual consistency for analytics
- Scalability: Handle 10x growth
```

### Capacity Estimation

```
### Twitter-Scale System Design

Traffic Estimates:
- 500M tweets/day, 200M active users
- Read:Write ratio = 100:1
- Peak throughput = 2x average

Storage Estimates:
- Tweet size: ~1KB
- Daily storage: 500GB/day
- Yearly storage: ~182TB/year

Bandwidth Estimates:
- Incoming writes: 500MB/s
- Outgoing reads: 50GB/s
```

### Back-of-the-Envelope Calculations

```
### Key Numbers
- 1M requests/day = ~12 requests/second
- 1M requests/second (peak) ≈ 12M/day
- 99th percentile latency is typically 5-10x median
- Cache hit ratio > 90% is good
- 1M records × 1KB ≈ 1GB
```

> **Best Practice**: Always start with requirements and capacity estimation before diving into design.

> 💡 **Pro Tip:** Memorize these key numbers for system design interviews: 1M req/day ≈ 12 req/s, 1KB per record, 1GB per million records, p99 latency ≈ 5-10x median. These back-of-the-envelope calculations let you quickly validate if a design is feasible before diving into details.

---

## Chapter 2: Low-Level Design (LLD)

Low-Level Design focuses on detailed implementation of individual components.

### Class Diagrams (UML)

```
class User {
  - id: UUID
  - email: String
  - passwordHash: String
  - createdAt: DateTime
  + register(email, password): User
  + login(email, password): Session
}

class Tweet {
  - id: UUID
  - content: String
  - userId: UUID
  - createdAt: DateTime
  - likes: int
  + create(userId, content): Tweet
  + delete(): void
  + like(): void
}

class TimelineService {
  - cache: RedisClient
  + getTimeline(userId, page, size): List<Tweet>
  + generateTimeline(userId): void
}

User "1" --> "*" Tweet : creates
User "1" --> "1" TimelineService : has
TimelineService --> Tweet : aggregates
```

### Sequence Diagrams

```
actor Client
participant "API Gateway" as GW
participant "Tweet Service" as TS
participant "Database" as DB
participant "Cache" as Cache

Client -> GW: POST /tweet
GW -> TS: Forward request
TS -> DB: INSERT tweet
TS -> Cache: Invalidate timeline
TS --> GW: 201 Created
GW --> Client: Response
```

### Database Schema

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    display_name VARCHAR(100),
    bio TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE tweets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    content VARCHAR(280) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
CREATE INDEX idx_tweets_created_at ON tweets(created_at DESC);

CREATE TABLE follows (
    follower_id UUID NOT NULL REFERENCES users(id),
    followee_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (follower_id, followee_id)
);

CREATE TABLE likes (
    user_id UUID NOT NULL REFERENCES users(id),
    tweet_id UUID NOT NULL REFERENCES tweets(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (user_id, tweet_id)
);
```

### API Contracts (OpenAPI)

```yaml
openapi: 3.0.0
info:
  title: Twitter Clone API
  version: 1.0.0
paths:
  /api/v1/tweets:
    post:
      summary: Create a new tweet
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - content
              properties:
                content:
                  type: string
                  maxLength: 280
      responses:
        '201':
          description: Tweet created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Tweet'
        '401':
          description: Unauthorized

  /api/v1/timeline:
    get:
      summary: Get user timeline
      parameters:
        - in: query
          name: page
          schema:
            type: integer
            default: 1
        - in: query
          name: size
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Timeline
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Tweet'

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    Tweet:
      type: object
      properties:
        id:
          type: string
          format: uuid
        content:
          type: string
        user:
          $ref: '#/components/schemas/User'
        created_at:
          type: string
          format: date-time
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        display_name:
          type: string
        email:
          type: string
          format: email
```

### State Machines

```python
from enum import Enum
from typing import Dict, Set

class OrderState(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    PROCESSING = "processing"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"
    REFUNDED = "refunded"

class OrderStateMachine:
    transitions: Dict[OrderState, Set[OrderState]] = {
        OrderState.PENDING: {OrderState.CONFIRMED, OrderState.CANCELLED},
        OrderState.CONFIRMED: {OrderState.PROCESSING, OrderState.CANCELLED},
        OrderState.PROCESSING: {OrderState.SHIPPED, OrderState.CANCELLED},
        OrderState.SHIPPED: {OrderState.DELIVERED, OrderState.CANCELLED},
        OrderState.DELIVERED: {OrderState.REFUNDED},
        OrderState.CANCELLED: set(),
        OrderState.REFUNDED: set(),
    }

    def __init__(self, initial_state: OrderState = OrderState.PENDING):
        self.current_state = initial_state

    def transition(self, new_state: OrderState) -> bool:
        if new_state in self.transitions[self.current_state]:
            self.current_state = new_state
            return True
        raise ValueError(f"Cannot transition from {self.current_state} to {new_state}")

> ✅ **Best Practice:** Always define API contracts (OpenAPI/Swagger) before writing implementation code. This enables frontend and backend teams to work in parallel, serves as living documentation, and catches design issues before code is written.

## Chapter 3: High-Level Design (HLD)

High-Level Design provides a bird's-eye view of the system architecture.

### Architecture Diagrams

```
                         Clients
                  (Mobile App, Web Browser)
                         |
                         v
              CDN (CloudFront/CloudFlare)
                         |
                         v
              Load Balancer (ALB/NGINX)
                         |
                         v
              API Gateway (Kong/AWS API GW)
              /        |          \
             v         v           v
     Tweet Svc   User Svc    Timeline Svc
             \         |           /
              v        v          v
         Message Queue (Kafka/SQS)
            /                  \
           v                    v
      Fan-Out Svc         Analytics Svc
           |                    |
           v                    v
      Redis Cache        Data Warehouse

      Primary DB (PostgreSQL) + Read Replicas
```

### Component Design

```python
class VideoService:
    def __init__(self):
        self.storage = S3Storage()
        self.transcoder = VideoTranscoder()
        self.cdn = CDNManager()

class RecommendationService:
    def __init__(self):
        self.feature_store = FeatureStore()
        self.model = RecommendationModel()
        self.cache = RedisCache()

class UserService:
    def __init__(self):
        self.db = UserDatabase()
        self.auth = AuthService()

class NotificationService:
    def __init__(self):
        self.push_provider = FirebasePush()
        self.email_provider = SESEmail()
```

### Data Flow

```
User Uploads Video:
1. Client -> Upload Service -> S3 (raw video)
2. Upload Service -> SQS -> Transcoder Service
3. Transcoder Service -> S3 (HLS/DASH segments)
4. Transcoder Service -> Database (update status)

User Watches Video:
1. Client -> API Gateway -> Video Service
2. Video Service -> CDN (check cache)
3. CDN miss -> Video Service -> S3
4. Video Service -> Analytics Service (log view)
```

> **Key Principle**: A good HLD should be technology-agnostic enough to be implemented with different tech stacks.

> 💡 **Pro Tip:** In system design interviews, start with a naive design, identify bottlenecks, and iterate. Don't jump to sharding, caching, and microservices immediately — show you understand when each complexity is justified.

---

## Chapter 4: Distributed Systems -- CAP Theorem

### CAP Theorem

The CAP theorem states that a distributed system can only provide two of three guarantees:
- **Consistency**: Every read receives the most recent write
- **Availability**: Every request receives a response
- **Partition Tolerance**: System continues despite network failures

### Database Choices by CAP Trade-offs

| Database | Category | Use Case |
|----------|----------|----------|
| PostgreSQL | CP (typically) | Financial transactions |
| MongoDB | CP (default) | Document storage |
| Cassandra | AP | Time-series, IoT, event logging |
| DynamoDB | AP (default) | Session management |
| Redis | CP (cluster) | Caching, leaderboards |
| CockroachDB | CP | Global distributed SQL |

### ACID vs BASE

| ACID | BASE |
|------|------|
| Atomicity | Basically Available |
| Consistency | Soft State |
| Isolation | Eventual Consistency |
| Durability | |

```sql
-- ACID example: Bank transfer
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

```javascript
// BASE example: Shopping cart (eventually consistent)
async function addToCart(userId, item) {
  await cartDB.put(`cart:${userId}:${item.id}`, item);
}
async function getCart(userId) {
  return await cartDB.get(`cart:${userId}`);  // May be stale
}
```

> ❓ **Interview Question:** When would you choose AP over CP in a distributed system? — Choose AP (e.g., Cassandra) when availability is critical and stale reads are acceptable: social media feeds, analytics dashboards, product catalogs. Choose CP (e.g., PostgreSQL) when consistency is non-negotiable: financial transactions, inventory management, distributed locks.

### Consistency Models

```python
# Strong Consistency
class StrongConsistentKV:
    def __init__(self):
        self.store = {}
        self.lock = threading.Lock()

    def put(self, key, value):
        with self.lock:
            self.store[key] = value
            self.replicate_sync(key, value)

    def get(self, key):
        with self.lock:
            return self.store[key]

# Eventual Consistency
class EventuallyConsistentKV:
    def __init__(self):
        self.store = {}
        self.version = 0

    def put(self, key, value):
        self.store[key] = value
        self.replicate_async(key, value)

    def get(self, key):
        return self.store.get(key)  # May return stale data

# Read-Your-Writes Consistency
class ReadYourWritesKV:
    def __init__(self, session_id):
        self.store = {}
        self.session_id = session_id

    def put(self, key, value):
        self.store[key] = value
        self.last_write_ts[self.session_id] = time.time()

    def get(self, key, session_id=None):
        if session_id == self.session_id:
            return self.store.get(key)  # Always latest
        return self.store.get(key)  # May be stale
```

### Consensus Algorithms

#### Raft

```python
class RaftNode:
    def __init__(self, node_id, peers):
        self.node_id = node_id
        self.peers = peers
        self.state = "follower"
        self.current_term = 0
        self.voted_for = None
        self.log = []
        self.commit_index = 0

    def request_vote(self, candidate_id, term, last_log_index, last_log_term):
        if term < self.current_term:
            return False, self.current_term
        if (self.voted_for is None or self.voted_for == candidate_id) and \
           self.is_log_up_to_date(last_log_index, last_log_term):
            self.voted_for = candidate_id
            return True, self.current_term
        return False, self.current_term

    def append_entries(self, term, leader_id, prev_log_index, prev_log_term, entries, leader_commit):
        if term < self.current_term:
            return False, self.current_term
        if not self.log_consistent(prev_log_index, prev_log_term):
            return False, self.current_term
        self.log = self.log[:prev_log_index + 1] + entries
        if leader_commit > self.commit_index:
            self.commit_index = min(leader_commit, len(self.log) - 1)
        return True, self.current_term

    def start_election(self):
        self.state = "candidate"
        self.current_term += 1
        self.voted_for = self.node_id
        votes = 1
        for peer in self.peers:
            if self.request_vote_from_peer(peer):
                votes += 1
        if votes > len(self.peers) // 2:
            self.state = "leader"
```

#### Paxos

```python
class PaxosAcceptor:
    def __init__(self):
        self.promised_ballot = None
        self.accepted_ballot = None
        self.accepted_value = None

    def prepare(self, ballot):
        if self.promised_ballot is None or ballot >= self.promised_ballot:
            self.promised_ballot = ballot
            return ("promise", ballot, self.accepted_ballot, self.accepted_value)
        return ("reject", self.promised_ballot)

    def accept(self, ballot, value):
        if ballot >= self.promised_ballot:
            self.promised_ballot = ballot
            self.accepted_ballot = ballot
            self.accepted_value = value
            return ("accepted", ballot)
        return ("reject", self.promised_ballot)

class PaxosProposer:
    def __init__(self, quorum_size, acceptors):
        self.quorum_size = quorum_size
        self.acceptors = acceptors
        self.ballot_number = 0

    def propose(self, value):
        self.ballot_number += 1
        promises = []
        highest_accepted = None
        for acceptor in self.acceptors:
            result = acceptor.prepare(self.ballot_number)
            if result[0] == "promise":
                promises.append(result)
                if result[2] is not None:
                    if highest_accepted is None or result[2] > highest_accepted[0]:
                        highest_accepted = (result[2], result[3])
        if len(promises) < self.quorum_size:
            return None
        final_value = highest_accepted[1] if highest_accepted else value
        accepts = 0
        for acceptor in self.acceptors:
            result = acceptor.accept(self.ballot_number, final_value)
            if result[0] == "accepted":
                accepts += 1
        if accepts >= self.quorum_size:
            return final_value
        return None
```

### Distributed Transactions

#### Two-Phase Commit (2PC)

```python
class TwoPhaseCommit:
    def __init__(self, participants):
        self.participants = participants

    def execute_transaction(self, transaction):
        for participant in self.participants:
            response = participant.prepare(transaction)
            if response != "READY":
                self.abort(transaction)
                return False
        for participant in self.participants:
            participant.commit(transaction)
        return True

    def abort(self, transaction):
        for participant in self.participants:
            participant.abort(transaction)
```

#### SAGA Pattern

```python
class Saga:
    def __init__(self):
        self.steps = []
        self.compensations = []

    def add_step(self, action, compensation):
        self.steps.append(action)
        self.compensations.append(compensation)

    def execute(self):
        executed = []
        for i, step in enumerate(self.steps):
            try:
                step()
                executed.append(i)
            except Exception as e:
                for idx in reversed(executed):
                    self.compensations[idx]()
                raise e

# Order Processing Saga
saga = Saga()
saga.add_step(lambda: reserve_inventory(order_id, items), lambda: release_inventory(order_id, items))
saga.add_step(lambda: charge_payment(order_id, amount), lambda: refund_payment(order_id, amount))
saga.add_step(lambda: ship_order(order_id, address), lambda: cancel_shipment(order_id))
saga.execute()
```

#### TCC (Try-Confirm/Cancel)

```python
class TCCService:
    def try_reserve(self, resource_id, amount):
        if self.available(resource_id) >= amount:
            self.hold(resource_id, amount)
            return "tentative_id_123"
        raise InsufficientResources()

    def confirm(self, tentative_id):
        self.release_hold(tentative_id)
        self.deduct(booking)

    def cancel(self, tentative_id):
        self.release_hold(tentative_id)
```

### Distributed ID Generation

#### Snowflake

```python
import time
import threading

class SnowflakeID:
    def __init__(self, datacenter_id, worker_id):
        self.datacenter_id = datacenter_id
        self.worker_id = worker_id
        self.sequence = 0
        self.last_timestamp = -1
        self.lock = threading.Lock()
        self.EPOCH = 1704067200000  # 2024-01-01
        self.SEQUENCE_BITS = 12

    def next_id(self):
        with self.lock:
            timestamp = int(time.time() * 1000)
            if timestamp < self.last_timestamp:
                raise Exception("Clock moved backwards!")
            if timestamp == self.last_timestamp:
                self.sequence = (self.sequence + 1) & 0xFFF
                if self.sequence == 0:
                    timestamp = self._wait_next_millis()
            else:
                self.sequence = 0
            self.last_timestamp = timestamp
            return ((timestamp - self.EPOCH) << 22) | (self.datacenter_id << 17) | (self.worker_id << 12) | self.sequence

    def _wait_next_millis(self):
        timestamp = int(time.time() * 1000)
        while timestamp <= self.last_timestamp:
            timestamp = int(time.time() * 1000)
        return timestamp
```

#### ULID

```python
import time
import os

class ULID:
    def generate(self):
        timestamp = int(time.time() * 1000)
        random_bytes = os.urandom(10)
        return self._encode_ulid(timestamp, random_bytes)

    def _encode_ulid(self, timestamp, random_bytes):
        charset = "0123456789ABCDEFGHJKMNPQRSTVWXYZ"
        ts_chars = []
        for _ in range(10):
            ts_chars.append(charset[timestamp & 0x1F])
            timestamp >>= 5
        rand_val = int.from_bytes(random_bytes, "big")
        rand_chars = []
        for _ in range(16):
            rand_chars.append(charset[rand_val & 0x1F])
            rand_val >>= 5
        return "".join(reversed(ts_chars)) + "".join(reversed(rand_chars))
```

---

## Chapter 5: Caching

### Caching Strategies

```python
import redis

# Cache-Aside (Lazy Loading)
class CacheAside:
    def __init__(self, cache_client, db):
        self.cache = cache_client
        self.db = db

    def get_user(self, user_id):
        user = self.cache.get(f"user:{user_id}")
        if user:
            return user
        user = self.db.query("SELECT * FROM users WHERE id = %s", user_id)
        self.cache.set(f"user:{user_id}", user, ex=3600)
        return user

    def update_user(self, user_id, data):
        self.db.execute("UPDATE users SET ... WHERE id = %s", user_id, data)
        self.cache.delete(f"user:{user_id}")

# Write-Through
class WriteThrough:
    def __init__(self, cache_client, db):
        self.cache = cache_client
        self.db = db

    def set_user(self, user_id, data):
        self.cache.set(f"user:{user_id}", data)
        self.db.execute("INSERT INTO users ...", user_id, data)

    def get_user(self, user_id):
        return self.cache.get(f"user:{user_id}")

# Write-Behind
class WriteBehind:
    def __init__(self, cache_client, db, queue):
        self.cache = cache_client
        self.db = db
        self.queue = queue

    def set_user(self, user_id, data):
        self.cache.set(f"user:{user_id}", data)
        self.queue.enqueue({"action": "update_user", "user_id": user_id, "data": data})

    def flush_pending_writes(self):
        while not self.queue.empty():
            item = self.queue.dequeue()
            self.db.execute("UPDATE users SET ... WHERE id = %s", item["data"], item["user_id"])
```

> ⚠️ **Warning:** Cache-aside is the most common and safest caching pattern. Write-through adds latency on writes. Write-behind risks data loss if the cache crashes before the write is flushed. Always set a TTL on cached data to prevent stale data from living forever.

### Cache Invalidation

```python
# TTL-based
redis_client.setex(key, 3600, value)

# LRU (Redis config)
redis_client.config_set("maxmemory", "2gb")
redis_client.config_set("maxmemory-policy", "allkeys-lru")

# LFU
redis_client.config_set("maxmemory-policy", "allkeys-lfu")

from collections import OrderedDict

class FIFOCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key):
        return self.cache.get(key)

    def put(self, key, value):
        if key not in self.cache and len(self.cache) >= self.capacity:
            self.cache.popitem(last=False)
        self.cache[key] = value
```

### Cache Stampede Prevention

```python
def get_or_compute_with_lock(key, compute_fn, ttl=3600):
    value = cache.get(key)
    if value is not None:
        return value
    lock_key = f"lock:{key}"
    if cache.setnx(lock_key, "1"):
        cache.expire(lock_key, 10)
        value = compute_fn()
        cache.setex(key, ttl, value)
        cache.delete(lock_key)
        return value
    while value is None:
        time.sleep(0.1)
        value = cache.get(key)
    return value
```

### Distributed Cache

```python
from redis.cluster import RedisCluster
rc = RedisCluster(startup_nodes=[
    {"host": "127.0.0.1", "port": 7000},
    {"host": "127.0.0.1", "port": 7001},
], decode_responses=True)
rc.set("key", "value")
rc.get("key")
```

---

## Chapter 6: Queues and Message Brokers

### Kafka

```python
from kafka import KafkaProducer, KafkaConsumer
import json

producer = KafkaProducer(
    bootstrap_servers=["localhost:9092"],
    value_serializer=lambda v: json.dumps(v).encode("utf-8"),
    acks="all",
    retries=3,
    compression_type="gzip"
)
producer.send("user-events", key=b"user_123", value={"user_id": "123", "event": "page_view"})
producer.flush()

consumer = KafkaConsumer(
    "user-events",
    bootstrap_servers=["localhost:9092"],
    group_id="analytics-group",
    auto_offset_reset="earliest",
    enable_auto_commit=False
)
for message in consumer:
    event = json.loads(message.value)
    process_event(event)
    consumer.commit()
```

### RabbitMQ

```python
import pika
import json

connection = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
channel = connection.channel()
channel.queue_declare(queue="task_queue", durable=True)

channel.basic_publish(
    exchange="",
    routing_key="task_queue",
    body=json.dumps({"task": "process_data", "id": 123}),
    properties=pika.BasicProperties(delivery_mode=2)
)

def callback(ch, method, properties, body):
    task = json.loads(body)
    process_task(task)
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue="task_queue", on_message_callback=callback)
channel.start_consuming()
```

> ✅ **Best Practice:** Use Kafka for event streaming (ordered, replayable, multi-consumer) and RabbitMQ for task queues (direct routing, priority, dead-lettering). Kafka excels at high-throughput event pipelines; RabbitMQ is better for complex routing and immediate task processing.

### Exactly-Once vs At-Least-Once

```python
# At-Least-Once (manual ack)
def callback(ch, method, properties, body):
    try:
        process_message(body)
        ch.basic_ack(method.delivery_tag)
    except Exception:
        ch.basic_nack(method.delivery_tag, requeue=True)

# Exactly-Once (Kafka idempotent + transactions)
producer = KafkaProducer(
    bootstrap_servers=["localhost:9092"],
    enable_idempotence=True,
    transactional_id="prod-1"
)
producer.init_transactions()
producer.begin_transaction()
try:
    producer.send("topic1", value=data1)
    producer.send("topic2", value=data2)
    producer.commit_transaction()
except Exception:
    producer.abort_transaction()
```

---

## Chapter 7: Event-Driven Architecture

### Events vs Commands

```python
# Command: Direct instruction
class PlaceOrderCommand:
    def __init__(self, order_id, user_id, items, total):
        self.type = "PlaceOrder"
        self.order_id = order_id
        self.user_id = user_id
        self.items = items
        self.total = total

# Event: Something that happened (fact)
class OrderPlacedEvent:
    def __init__(self, order_id, user_id, items, total):
        self.type = "OrderPlaced"
        self.order_id = order_id
        self.user_id = user_id
        self.items = items
        self.total = total

# Multiple handlers react independently
class OrderEventHandler:
    def handle(self, event):
        if isinstance(event, OrderPlacedEvent):
            self.inventory_service.reserve_items(event.items)
            self.notification_service.send_confirmation(event.user_id)
            self.analytics_service.log_order(event.order_id, event.total)
```

### Event Sourcing

```python
from abc import ABC, abstractmethod

class Event(ABC):
    @abstractmethod
    def apply(self, aggregate): pass

class AccountCreated(Event):
    def __init__(self, account_id, owner, initial_balance):
        self.account_id = account_id
        self.owner = owner
        self.initial_balance = initial_balance
    def apply(self, account):
        account.account_id = self.account_id
        account.owner = self.owner
        account.balance = self.initial_balance
        account.version += 1

class MoneyDeposited(Event):
    def __init__(self, account_id, amount):
        self.account_id = account_id
        self.amount = amount
    def apply(self, account):
        account.balance += self.amount
        account.version += 1

class BankAccount:
    def __init__(self):
        self.account_id = None
        self.owner = None
        self.balance = 0
        self.version = 0

    @staticmethod
    def from_events(events):
        account = BankAccount()
        for event in events:
            event.apply(account)
        return account
```

> 💡 **Pro Tip:** Event-driven architecture decouples services, making them independently deployable and scalable. However, it introduces complexity: eventual consistency, debugging difficulty, and event schema evolution. Start synchronous, extract async boundaries only when needed.

### CQRS

```python
# Command side (writes)
class CreateOrderCommand:
    def __init__(self, user_id, items, shipping_address):
        self.user_id = user_id
        self.items = items
        self.shipping_address = shipping_address

class OrderCommandHandler:
    def handle_create_order(self, command):
        events = [OrderCreatedEvent(order_id=uuid.uuid4(), user_id=command.user_id, items=command.items)]
        self.event_store.save(events)
        self.event_bus.publish(events)

# Query side (reads)
class OrderQueryHandler:
    def __init__(self, read_db):
        self.read_db = read_db

    def get_order_summary(self, order_id):
        return self.read_db.query("SELECT * FROM order_summaries WHERE id = %s", order_id)

    def get_user_orders(self, user_id, page, size):
        return self.read_db.query(
            "SELECT * FROM order_summaries WHERE user_id = %s ORDER BY created_at DESC LIMIT %s OFFSET %s",
            user_id, size, (page - 1) * size)

# Projection
class OrderProjection:
    def project_order_created(self, event):
        self.read_db.execute(
            "INSERT INTO order_summaries (id, user_id, item_count, total, status, created_at) VALUES (%s, %s, %s, %s, %s, %s)",
            event.order_id, event.user_id, len(event.items), event.total, "created", event.timestamp)
```

### Outbox Pattern

```python
class OutboxPattern:
    def __init__(self, db, message_broker):
        self.db = db
        self.message_broker = message_broker

    def create_order(self, order_data):
        with self.db.transaction():
            self.db.execute("INSERT INTO orders (id, user_id, items, total, created_at) VALUES (%s, %s, %s, %s, %s)", order_data)
            self.db.execute(
                "INSERT INTO outbox (id, aggregate_type, aggregate_id, event_type, payload, created_at, processed) "
                "VALUES (%s, %s, %s, %s, %s, %s, false)",
                order_data["id"], "order", order_data["id"], "OrderCreated", json.dumps(order_data), now)

    def process_outbox(self):
        messages = self.db.query("SELECT * FROM outbox WHERE processed = false ORDER BY created_at ASC LIMIT 100")
        for message in messages:
            try:
                self.message_broker.publish(message["aggregate_type"], message["payload"])
                self.db.execute("UPDATE outbox SET processed = true WHERE id = %s", message["id"])
            except Exception as e:
                print(f"Failed: {e}")
```

### Dead Letter Queues

```python
class DeadLetterQueue:
    def __init__(self, primary_queue, dlq):
        self.primary_queue = primary_queue
        self.dlq = dlq

    def process_with_dlq(self):
        def callback(ch, method, properties, body):
            try:
                process_message(body)
                ch.basic_ack(method.delivery_tag)
            except Exception as e:
                if self.should_move_to_dlq(method):
                    self.dlq.publish({"original_message": body, "error": str(e)})
                    ch.basic_ack(method.delivery_tag)
                else:
                    ch.basic_nack(method.delivery_tag, requeue=True)
        self.primary_queue.consume(callback)

    def should_move_to_dlq(self, method):
        retry_count = method.headers.get("retry-count", 0)
        return retry_count >= 3
```

---

## Chapter 8: Scalability

### Horizontal vs Vertical Scaling

```
Vertical Scaling (Scale Up):
  Server (8GB RAM, 4 CPU) -> Server (64GB RAM, 32 CPU)

Horizontal Scaling (Scale Out):
  Server 1, Server 2, Server 3 -> Add more servers
```

### Sharding Strategies

```python
# Hash-Based Sharding
class HashShard:
    def __init__(self, num_shards):
        self.num_shards = num_shards
        self.shards = [{} for _ in range(num_shards)]

    def get_shard(self, key):
        return hash(key) % self.num_shards

    def put(self, key, value):
        self.shards[self.get_shard(key)][key] = value

# Range-Based Sharding
class RangeShard:
    def __init__(self, ranges):
        self.ranges = ranges
        self.shards = [{} for _ in ranges]

    def get_shard(self, user_id):
        for i, (start, end) in enumerate(self.ranges):
            if start <= user_id <= end:
                return i
        raise ValueError("User ID out of range")

# Directory-Based Sharding
class DirectoryShard:
    def __init__(self):
        self.directory = {}
        self.shards = [{} for _ in range(4)]

    def get_shard(self, key):
        if key not in self.directory:
            self.directory[key] = len(self.shards) - 1
        return self.directory[key]
```

> ✅ **Best Practice:** Always scale vertically first before adding horizontal complexity. Upgrading to a larger server is simpler, cheaper, and avoids distributed system trade-offs. Only shard when a single node can't handle the workload after vertical scaling has been exhausted.

### Replication

```python
# Leader-Follower
class LeaderNode:
    def __init__(self):
        self.data = {}
        self.followers = []
        self.write_ahead_log = []

    def write(self, key, value):
        self.data[key] = value
        entry = {"key": key, "value": value}
        self.write_ahead_log.append(entry)
        for follower in self.followers:
            follower.replicate(entry)

class FollowerNode:
    def __init__(self, leader):
        self.leader = leader
        self.data = {}
    def replicate(self, entry):
        self.data[entry["key"]] = entry["value"]
    def read(self, key):
        return self.data.get(key)
    def write(self, key, value):
        return self.leader.write(key, value)
```

### Load Balancing

```python
class LoadBalancer:
    def __init__(self, backends):
        self.backends = backends
        self.current = 0

    # Round Robin
    def round_robin(self):
        backend = self.backends[self.current]
        self.current = (self.current + 1) % len(self.backends)
        return backend

    # Least Connections
    def least_connections(self):
        return min(self.backends, key=lambda b: b.active_connections)
```

### Rate Limiting

```python
import time
from collections import defaultdict

# Token Bucket
class TokenBucket:
    def __init__(self, rate, capacity):
        self.rate = rate
        self.capacity = capacity
        self.tokens = capacity
        self.last_refill = time.time()

    def allow_request(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.rate)
        self.last_refill = now
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False

# Sliding Window
class SlidingWindow:
    def __init__(self, limit, window_size):
        self.limit = limit
        self.window_size = window_size
        self.requests = defaultdict(list)

    def allow_request(self, client_id):
        now = time.time()
        window_start = now - self.window_size
        self.requests[client_id] = [t for t in self.requests[client_id] if t > window_start]
        if len(self.requests[client_id]) < self.limit:
            self.requests[client_id].append(now)
            return True
        return False
```

### Microservices Patterns

```python
# Circuit Breaker
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.state = "CLOSED"
        self.last_failure_time = None

    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time >= self.recovery_timeout:
                self.state = "HALF_OPEN"
            else:
                raise CircuitBreakerOpen()
        try:
            result = func(*args, **kwargs)
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
            raise e

# Bulkhead
class Bulkhead:
    def __init__(self, max_concurrent):
        self.semaphore = threading.Semaphore(max_concurrent)
    def execute(self, func, *args, **kwargs):
        with self.semaphore:
            return func(*args, **kwargs)
```

---

## Part 16 Revision Notes

### Key Concepts
- System Design: requirements, capacity estimation, HLD, LLD
- CAP Theorem: Consistency, Availability, Partition Tolerance
- ACID vs BASE, Consistency Models
- Consensus: Paxos, Raft, Zab
- Distributed Transactions: 2PC, SAGA, TCC
- Caching: cache-aside, write-through, write-behind
- Queues: Kafka, RabbitMQ, ordering guarantees
- Event-Driven: CQRS, Event Sourcing, Outbox
- Scalability: sharding, replication, load balancing
- Microservices: circuit breaker, bulkhead, saga

### Common Mistakes
- Not defining non-functional requirements early
- Assuming strong consistency when eventual works
- Cache stampede without protection
- Not planning for partition tolerance
- Over-engineering before understanding scale

### Interview Questions
1. Design a URL shortener (requirements to architecture)
2. When would you choose Kafka over RabbitMQ?
3. Explain the CAP theorem with real-world examples
4. How does Raft leader election work?
5. Design a distributed rate limiter

---
