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
