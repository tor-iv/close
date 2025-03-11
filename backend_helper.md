# Primary Database: PostgreSQL

## Database Schema

### Users Tables

#### users
- `user_id` (primary key, uuid)
- `phone_number` (indexed, unique)
- `email` (optional, unique)
- `username` (unique)
- `display_name`
- `profile_picture_url`
- `date_joined` (timestamp)
- `status` (enum: active, pending, suspended)
- `last_active` (timestamp)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### user_settings
- `user_id` (primary key, foreign key to users)
- `notification_preferences` (jsonb)
- `privacy_settings` (jsonb)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### authentication
- `auth_id` (primary key)
- `user_id` (foreign key to users)
- `phone_verified` (boolean)
- `verification_code` (encrypted)
- `verification_expiry` (timestamp)
- `password_hash` (if using password authentication)
- `auth_method` (enum: phone, social, etc.)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### refresh_tokens
- `token_id` (primary key)
- `user_id` (foreign key to users, indexed)
- `token_hash` (indexed)
- `expires_at` (timestamp)
- `created_at` (timestamp)
- `device_info` (jsonb)

### Content Tables

#### posts
- `post_id` (primary key, uuid)
- `user_id` (foreign key to users, indexed)
- `image_url` (text)
- `image_metadata` (jsonb - thumbnails, dimensions, etc.)
- `caption` (text, optional)
- `posted_at` (timestamp, indexed)
- `comment_count` (integer)
- `status` (enum: active, deleted, flagged)
- `visibility` (enum: friends-only, specific-friends)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### comments
- `comment_id` (primary key, uuid)
- `post_id` (foreign key to posts, indexed)
- `user_id` (foreign key to users, indexed)
- `content` (text)
- `commented_at` (timestamp, indexed)
- `status` (enum: active, deleted, flagged)
- `created_at` (timestamp)
- `updated_at` (timestamp)

### Social Relationship Tables

#### contacts
- `contact_id` (primary key)
- `user_id` (foreign key to users, indexed)
- `contact_phone_number` (indexed)
- `contact_name` (text)
- `status` (enum: friend, pending, not-on-platform)
- `date_added` (timestamp)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### friendships
- `friendship_id` (primary key)
- `user_id_1` (foreign key to users, indexed)
- `user_id_2` (foreign key to users, indexed)
- `status` (enum: pending, active)
- `requested_at` (timestamp)
- `accepted_at` (timestamp, nullable)
- `initiated_by` (foreign key to users)
- `created_at` (timestamp)
- `updated_at` (timestamp)
- UNIQUE CONSTRAINT on (user_id_1, user_id_2)

## Secondary Database Tables

### Notification System

#### notifications
- `notification_id` (primary key, uuid)
- `user_id` (foreign key to users, indexed)
- `type` (enum: comment, friend_request, system)
- `reference_id` (uuid, could be post_id, comment_id, user_id)
- `reference_type` (enum: post, comment, user)
- `content` (text)
- `created_at` (timestamp, indexed)
- `read_at` (timestamp, nullable)
- `delivered` (boolean)

### Analytics System

#### user_activities (Time-series optimized table or dedicated TimescaleDB)
- `activity_id` (primary key, uuid)
- `user_id` (foreign key to users, indexed)
- `event_type` (enum: login, post, comment, browse)
- `occurred_at` (timestamp, indexed)
- `metadata` (jsonb)
- `ip_address` (inet)
- `device_info` (jsonb)

## Server Requirements

### Compute Resources

#### API Servers:
- Initial: 2-4 medium instances (4 vCPUs, 8GB RAM)
- Auto-scaling configuration based on request load
- Multi-region deployment for reliability

#### Image Processing:
- 2 computation-optimized instances (8 vCPUs, 16GB RAM)
- GPU support for image optimization (optional for MVP)

#### Background Workers:
- 2 small instances (2 vCPUs, 4GB RAM)
- For handling notifications, analytics, and other asynchronous tasks

### Storage Requirements

#### User Data:
- Initial: 50-100GB database storage
- Growth estimate: ~10MB per 1000 users (excluding images)

#### Image Storage:
- Initial: 500GB-1TB blob/object storage
- Growth estimate: ~5GB per 1000 posts
- CDN integration for image delivery

#### Backup Storage:
- Equivalent to primary storage with retention policy
- Point-in-time recovery capability

### Network Requirements

#### Bandwidth:
- Initial: 10-20TB monthly transfer
- Growth estimate: ~500GB per 1000 daily active users

#### Latency:
- API responses: <100ms target
- Image delivery: <200ms target

### Security Requirements
- SSL/TLS encryption for all connections
- Rate limiting to prevent abuse
- DDoS protection
- Data encryption at rest and in transit
- Secure phone verification system
- Regular security audits

## Recommended Tech Stack

### Primary Recommendation: Python Stack
- Language: Python 3.11+
- API Framework: FastAPI 
- Databases:
  - PostgreSQL for primary data storage
  - Redis for caching and real-time features
- ORM: SQLAlchemy with Alembic migrations (or Django ORM)
- Authentication: JWTs with Pydantic validation
- Image Processing: Pillow/OpenCV
- Async Support: asyncio and uvicorn
- API Documentation: Automatic OpenAPI/Swagger via FastAPI
- Cloud Services: AWS or Google Cloud Platform

# Python Stack Implementation Details

## FastAPI Setup
- **ASGI Server:** Uvicorn with Gunicorn workers
- **API Structure:** Router-based organization with dependency injection
- **Request Validation:** Pydantic models for request/response validation
- **Background Tasks:** FastAPI background tasks + Celery for longer processes
- **Middleware:** Rate limiting, authentication, logging middleware
- **Testing:** pytest with async support

## Database Integration
- **SQLAlchemy Setup:**
  - Async SQLAlchemy for non-blocking database operations
  - Repository pattern for database operations
  - Alembic for migrations management
  - PostgreSQL connection pooling
- **Database Access Patterns:**
  - Unit of Work pattern for transactions
  - Query objects for complex queries
  - Optimistic concurrency control

## Authentication Flow
- **Phone Verification:**
  - Twilio or similar service for SMS verification
  - Rate-limited verification attempts
  - Short-lived verification tokens
- **Session Management:**
  - JWT tokens with short expiry
  - Redis-backed refresh tokens
  - Secure cookie approach for web clients

## Real-time Features Implementation
- **WebSockets with FastAPI:**
  - Integrated WebSocket support for real-time updates
  - Authentication for WebSocket connections
  - Channel-based subscription model
- **Alternative Push Mechanisms:**
  - Server-Sent Events (SSE) for one-way updates
  - Push notifications via Firebase Cloud Messaging for mobile
  - Polling fallback with proper caching

## Image Processing Pipeline
- **Upload Flow:**
  - Direct-to-S3 uploads with presigned URLs
  - Asynchronous processing via Celery tasks
  - Multi-size image generation (thumbnails, etc.)
  - Metadata extraction and validation
- **Optimization:**
  - Content-based image resizing
  - Progressive loading support
  - WebP/AVIF format conversion for web delivery
  - Image CDN integration

## Deployment Architecture
- **Container-based:**
  - Docker for containerization
  - Docker Compose for local development
  - Kubernetes for production orchestration
- **CI/CD Pipeline:**
  - GitHub Actions for CI/CD
  - Automated testing and deployment
  - Blue/Green deployment strategy
- **Monitoring:**
  - Prometheus for metrics
  - OpenTelemetry for tracing
  - Structured logging with ELK stack