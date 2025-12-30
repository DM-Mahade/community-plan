
---

# 🧩 COMMUNITY WEBSITE – COMPLETE MYSQL DATABASE SCHEMA

---

## **DATABASE**
###**`myboq_community`**
---

## **1️⃣ USERS**

### **Table Name**

`cm_users`

---

### **Schema (Table Format)**

| Column         | Type         | Description                                            |
| -------------- | ------------ | ------------------------------------------------------ |
| id             | BIGINT (PK)  | Community user identifier                              |
| public_handle  | VARCHAR(100) | Public community username (used for mentions, tagging) |
| fk_sts_user_id | BIGINT (FK)  | Reference to `sts_ssrpro.sts_users.user_id`            |
| email          | VARCHAR(100) | Email address (optional, synced if needed)             |
| mobile         | VARCHAR(20)  | Mobile number (optional, synced if needed)             |
| full_name      | VARCHAR(200) | Full name                                              |
| profile_image  | VARCHAR(255) | Profile image                                          |
| bio            | TEXT         | User bio                                               |
| status         | ENUM         | Account status                                         |
| created_at     | TIMESTAMP    | Created date                                           |
| updated_at     | TIMESTAMP    | Updated date                                           |

---

### **Query**

```sql
CREATE TABLE cm_users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    public_handle VARCHAR(100) UNIQUE NOT NULL,
    fk_sts_user_id BIGINT NOT NULL,
    email VARCHAR(100),
    mobile VARCHAR(20),
    full_name VARCHAR(200),
    profile_image VARCHAR(255),
    bio TEXT,
    status ENUM('active','blocked') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### **Purpose**

Stores user identity, authentication, and profile information for the platform.

---

## **2️⃣ CATEGORIES**

### **Table Name**

`categories`

### **Schema (Table Format)**

| Column      | Type         | Description          |
| ----------- | ------------ | -------------------- |
| id          | BIGINT (PK)  | Category ID          |
| name        | VARCHAR(200) | Category name        |
| description | TEXT         | Category description |
| created_by  | BIGINT (FK)  | Creator              |
| created_at  | TIMESTAMP    | Created date         |

### **Query**

```sql
CREATE TABLE categories (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) UNIQUE NOT NULL,
    description TEXT,
    created_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES cm_users(id)
);
```

### **Purpose**

Defines high-level grouping of topics to organize community discussions.

---


## **3️⃣ TAGS**

### **Table Name**

`tags`

### **Schema**

| Column     | Type         | Description  |
| ---------- | ------------ | ------------ |
| id         | BIGINT (PK)  | Tag ID       |
| name       | VARCHAR(100) | Tag name     |
| created_by | BIGINT (FK)  | Creator      |
| created_at | TIMESTAMP    | Created date |

### **Query**

```sql
CREATE TABLE tags (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) UNIQUE NOT NULL,
    created_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES cm_users(id)
);
```

### **Purpose**

Used for filtering, searching, and classifying topics across categories.

---

## **4️⃣ TOPICS**

### **Table Name**

`topics`

### **Schema**

| Column        | Type         | Description          |
| ------------- | ------------ | -------------------- |
| id            | BIGINT (PK)  | Topic ID             |
| user_id       | BIGINT (FK)  | Topic creator        |
| category_id   | BIGINT (FK)  | Category             |
| title         | VARCHAR(500) | Topic title          |
| content       | TEXT         | Topic content        |
| is_answered   | BOOLEAN      | First reply exists   |
| is_solved     | BOOLEAN      | Best answer selected |
| best_reply_id | BIGINT       | Best reply           |
| created_at    | TIMESTAMP    | Created date         |
| updated_at    | TIMESTAMP    | Updated date         |

### **Query**

```sql
CREATE TABLE topics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    category_id BIGINT NOT NULL,
    title VARCHAR(500) NOT NULL,
    content TEXT NOT NULL,
    is_answered BOOLEAN DEFAULT FALSE,
    is_solved BOOLEAN DEFAULT FALSE,
    best_reply_id BIGINT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES cm_users(id),
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

### **Purpose**

Stores all user-created discussion topics.

### **Business Logic**

* `is_answered = TRUE` only when the **first reply** is added
* `is_solved = TRUE` only when **best answer** is selected

---


## **5️⃣ TOPIC ATTACHMENTS**

### **Table Name**

`topic_attachments`

### **Query**

```sql
CREATE TABLE topic_attachments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT NOT NULL,
    file_url VARCHAR(500) NOT NULL,
    file_type ENUM('image','pdf','doc') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (topic_id) REFERENCES topics(id)
);
```

### **Purpose**

Stores files attached to topics such as images or documents.

---


## 6️⃣ TOPIC TAG MAPPING (Many-to-Many)

### **Table Name**

`topic_tags`

### **Schema (Table Format)**

| Column Name | Type            | Description        |
| ----------- | --------------- | ------------------ |
| topic_id    | BIGINT (PK, FK) | Reference to topic |
| tag_id      | BIGINT (PK, FK) | Reference to tag   |


### **Query**

```sql
CREATE TABLE topic_tags (
    topic_id BIGINT,
    tag_id BIGINT,
    PRIMARY KEY (topic_id, tag_id),
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);
```

### **Purpose**

Enables many-to-many mapping between topics and tags.

---

### **Table Name**

`topic_replies`


### **Schema (Table Format)**

| Column Name     | Type        | Description            |
| --------------- | ----------- | ---------------------- |
| id              | BIGINT (PK) | Reply identifier       |
| topic_id        | BIGINT (FK) | Parent topic           |
| user_id         | BIGINT (FK) | Reply author           |
| parent_reply_id | BIGINT (FK) | Nested reply reference |
| content         | TEXT        | Reply content          |
| is_best_answer  | BOOLEAN     | Best answer flag       |
| is_reported     | BOOLEAN     | Reported flag          |
| created_at      | TIMESTAMP   | Created timestamp      |


### **Query**

```sql
CREATE TABLE topic_replies (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    parent_reply_id BIGINT NULL,
    content TEXT NOT NULL,
    is_best_answer BOOLEAN DEFAULT FALSE,
    is_reported BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (user_id) REFERENCES cm_users(id),
    FOREIGN KEY (parent_reply_id) REFERENCES topic_replies(id)
);
```

### **Purpose**

Stores all replies for topics with nested reply support.

### **Business Logic**

* Only **topic owner** can mark `is_best_answer = TRUE`
* Only **one reply** can be best answer per topic

---

## 8️⃣ REPLY ATTACHMENTS
### **Table Name**

`reply_attachments`

### **Schema (Table Format)**

| Column Name | Type         | Description     |
| ----------- | ------------ | --------------- |
| id          | BIGINT (PK)  | Attachment ID   |
| reply_id    | BIGINT (FK)  | Reply reference |
| file_url    | VARCHAR(500) | File location   |
| file_type   | ENUM         | Attachment type |
| created_at  | TIMESTAMP    | Uploaded date   |


### **Query**

```sql
CREATE TABLE reply_attachments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    reply_id BIGINT NOT NULL,
    file_url VARCHAR(500) NOT NULL,
    file_type ENUM('image','pdf','doc') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (reply_id) REFERENCES topic_replies(id)
);
```

### **Purpose**

Stores files attached to replies.

---


## 9️⃣ TOPIC LIKES
### **Table Name**

`topic_likes`

### **Schema (Table Format)**

| Column Name | Type        | Description     |
| ----------- | ----------- | --------------- |
| id          | BIGINT (PK) | Like ID         |
| topic_id    | BIGINT (FK) | Topic reference |
| user_id     | BIGINT (FK) | Liked by user   |
| created_at  | TIMESTAMP   | Like date       |


### **Query**

```sql
CREATE TABLE topic_likes (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (topic_id, user_id),
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Tracks likes received on topics.

---


## **🔟 REPLY LIKES**

### **Table Name**

`reply_likes`

### **Schema (Table Format)**

| Column Name | Type        | Description     |
| ----------- | ----------- | --------------- |
| id          | BIGINT (PK) | Like ID         |
| reply_id    | BIGINT (FK) | Reply reference |
| user_id     | BIGINT (FK) | Liked by user   |
| created_at  | TIMESTAMP   | Like date       |


### **Query**

```sql
CREATE TABLE reply_likes (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    reply_id BIGINT,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (reply_id, user_id),
    FOREIGN KEY (reply_id) REFERENCES topic_replies(id),
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Tracks likes received on replies.

---


## **1️⃣1️⃣ TOPIC VIEWS**

### **Table Name**

`topic_views`

### **Schema (Table Format)**

| Column Name | Type        | Description       |
| ----------- | ----------- | ----------------- |
| id          | BIGINT (PK) | View ID           |
| topic_id    | BIGINT (FK) | Topic reference   |
| user_id     | BIGINT (FK) | Viewer (nullable) |
| viewed_at   | TIMESTAMP   | View time         |


### **Query**

```sql
CREATE TABLE topic_views (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT,
    user_id BIGINT NULL,
    viewed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Stores topic view events for analytics.

---


## **1️⃣2️⃣ BOOKMARK TOPICS**

### **Table Name**

`bookmark_topics`

### **Schema (Table Format)**

| Column Name | Type        | Description     |
| ----------- | ----------- | --------------- |
| id          | BIGINT (PK) | Bookmark ID     |
| topic_id    | BIGINT (FK) | Topic reference |
| user_id     | BIGINT (FK) | Bookmark owner  |
| created_at  | TIMESTAMP   | Bookmark date   |


### **Query**

```sql
CREATE TABLE bookmark_topics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (topic_id, user_id),
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Allows users to save topics for later reference.

---

## **1️⃣3️⃣ TOPIC REPORTS**

### **Table Name**

`topic_reports`

### **Schema (Table Format)**

| Column Name | Type         | Description    |
| ----------- | ------------ | -------------- |
| id          | BIGINT (PK)  | Report ID      |
| topic_id    | BIGINT (FK)  | Reported topic |
| reported_by | BIGINT (FK)  | Reporting user |
| reason      | VARCHAR(800) | Report reason  |
| created_at  | TIMESTAMP    | Report date    |


### **Query**

```sql
CREATE TABLE topic_reports (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT,
    reported_by BIGINT,
    reason VARCHAR(800),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (reported_by) REFERENCES cm_users(id)
);
```

### **Purpose**

Stores reports raised against topics by users.

---

## **1️⃣4️⃣ REPLY REPORTS**

### **Table Name**

`reply_reports`

### **Schema (Table Format)**

| Column Name | Type         | Description    |
| ----------- | ------------ | -------------- |
| id          | BIGINT (PK)  | Report ID      |
| reply_id    | BIGINT (FK)  | Reported reply |
| reported_by | BIGINT (FK)  | Reporting user |
| reason      | VARCHAR(800) | Report reason  |
| created_at  | TIMESTAMP    | Report date    |


### **Query**

```sql
CREATE TABLE reply_reports (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    reply_id BIGINT,
    reported_by BIGINT,
    reason VARCHAR(800),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (reply_id) REFERENCES topic_replies(id),
    FOREIGN KEY (reported_by) REFERENCES cm_users(id)
);
```

### **Purpose**

Stores reports raised against replies.

---

## **1️⃣5️⃣ TOPIC STATS**

### **Table Name**

`topic_stats`

### **Schema (Table Format)**

| Column Name   | Type            | Description     |
| ------------- | --------------- | --------------- |
| topic_id      | BIGINT (PK, FK) | Topic reference |
| total_likes   | INT             | Total likes     |
| total_shares  | INT             | Total shares    |
| total_views   | INT             | Total views     |
| total_replies | INT             | Total replies   |
| total_reports | INT             | Total reports   |


### **Query**

```sql
CREATE TABLE topic_stats (
    topic_id BIGINT PRIMARY KEY,
    total_likes INT DEFAULT 0,
    total_shares INT DEFAULT 0,
    total_views INT DEFAULT 0,
    total_replies INT DEFAULT 0,
    total_reports INT DEFAULT 0,
    FOREIGN KEY (topic_id) REFERENCES topics(id)
);
```

### **Purpose**

Stores aggregated topic analytics for fast reads.

---

## **1️⃣6️⃣ REPLY STATS**

### **Table Name**

`reply_stats`

### **Schema (Table Format)**

| Column Name   | Type            | Description     |
| ------------- | --------------- | --------------- |
| reply_id      | BIGINT (PK, FK) | Reply reference |
| total_likes   | INT             | Total likes     |
| total_reports | INT             | Total reports   |
| total_replies | INT             | Nested replies  |


### **Query**

```sql
CREATE TABLE reply_stats (
    reply_id BIGINT PRIMARY KEY,
    total_likes INT DEFAULT 0,
    total_reports INT DEFAULT 0,
    total_replies INT DEFAULT 0,
    FOREIGN KEY (reply_id) REFERENCES topic_replies(id)
);
```

### **Purpose**

Stores aggregated reply analytics.

---

## **1️⃣7️⃣ USER ACTIVITY STATS**

### **Table Name**

`user_activity_stats`

### **Schema (Table Format)**

| Column Name          | Type            | Description         |
| -------------------- | --------------- | ------------------- |
| user_id              | BIGINT (PK, FK) | User reference      |
| topics_created       | INT             | Topics count        |
| replies_posted       | INT             | Replies count       |
| topics_solved        | INT             | Solved topics       |
| likes_received       | INT             | Likes received      |
| conversations_joined | INT             | Participated topics |


### **Query**

```sql
CREATE TABLE user_activity_stats (
    user_id BIGINT PRIMARY KEY,
    topics_created INT DEFAULT 0,
    replies_posted INT DEFAULT 0,
    topics_solved INT DEFAULT 0,
    likes_received INT DEFAULT 0,
    conversations_joined INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Provides fast dashboard statistics for users.

---

## **1️⃣8️⃣ USER POINTS**

### **Table Name**

`user_points`

### **Schema (Table Format)**

| Column Name | Type            | Description    |
| ----------- | --------------- | -------------- |
| user_id     | BIGINT (PK, FK) | User reference |
| points      | INT             | Total points   |
| batch       | VARCHAR(200)    | User batch     |
| updated_at  | TIMESTAMP       | Updated date   |


### **Query**

```sql
CREATE TABLE user_points (
    user_id BIGINT PRIMARY KEY,
    points INT DEFAULT 0,
    batch VARCHAR(200),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES cm_users(id)
);
```

### **Purpose**

Tracks gamification points and user ranking level.

---

## **1️⃣9️⃣ NOTIFICATIONS**

### **Table Name**

`notifications`

### **Schema (Table Format)**

| Column Name | Type        | Description       |
| ----------- | ----------- | ----------------- |
| id          | BIGINT (PK) | Notification ID   |
| receiver_id | BIGINT (FK) | Receiver          |
| sender_id   | BIGINT (FK) | Sender            |
| topic_id    | BIGINT (FK) | Topic reference   |
| reply_id    | BIGINT (FK) | Reply reference   |
| type        | ENUM        | Notification type |
| is_read     | BOOLEAN     | Read status       |
| created_at  | TIMESTAMP   | Created date      |


### **Query**

```sql
CREATE TABLE notifications (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    receiver_id BIGINT,
    sender_id BIGINT NULL,
    topic_id BIGINT NULL,
    reply_id BIGINT NULL,
    type ENUM('LIKE','REPLY','MENTION','BEST_ANSWER','REPORT'),
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (receiver_id) REFERENCES cm_users(id),
    FOREIGN KEY (sender_id) REFERENCES cm_users(id),
    FOREIGN KEY (topic_id) REFERENCES topics(id),
    FOREIGN KEY (reply_id) REFERENCES topic_replies(id)
);
```

### **Purpose**

Stores in-app notifications for user activities.

---

# 🧠 BUSINESS LOGIC

### ✔ `is_answered`

* `TRUE` **only when first reply is added**

```sql
UPDATE topics
SET is_answered = TRUE
WHERE id = :topicId;
```

---

### ✔ BEST ANSWER LOGIC (As Requested – Exact)

```sql
UPDATE topic_replies
SET is_best_answer = TRUE
WHERE id = :replyId;

UPDATE topics
SET
    is_solved = TRUE,
    best_reply_id = :replyId
WHERE id = :topicId;
```

---

# 🔥 IMPORTANT INDEXES (Performance)

```sql
CREATE INDEX idx_topics_category ON topics(category_id);
CREATE INDEX idx_topic_replies_topic ON topic_replies(topic_id);
CREATE INDEX idx_notifications_receiver ON notifications(receiver_id, is_read);
CREATE INDEX idx_topic_stats_views ON topic_stats(total_views);
```

---

## **2️⃣0️⃣ user_email_preferences**

### **Schema (Table Format)**

| Column Name       | Type        | Description    |
| ----------------- | ----------- | -------------- |
| id                | BIGINT (PK) | Preference ID  |
| user_id           | BIGINT (FK) | User reference |
| notification_type | ENUM        | Activity type  |
| is_email_enabled  | BOOLEAN     | Email toggle   |
| updated_at        | TIMESTAMP   | Updated date   |


```sql
CREATE TABLE user_email_preferences (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    notification_type ENUM('LIKE','REPLY','MENTION','BEST_ANSWER','REPORT') NOT NULL,
    is_email_enabled BOOLEAN DEFAULT TRUE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_user_notification (user_id, notification_type),
    CONSTRAINT fk_uep_user
        FOREIGN KEY (user_id) REFERENCES cm_users(id)
);

```

### **Purpose**

Stores each user’s email notification preferences.
Allows users to control which activity types should trigger email notifications, while still receiving in-app notifications.

---

## **2️⃣1️⃣ report_issue**

### **Schema (Table Format)**

| Column Name | Type         | Description       |
| ----------- | ------------ | ----------------- |
| id          | BIGINT (PK)  | Issue ID          |
| user_id     | BIGINT (FK)  | Reporting user    |
| full_name   | VARCHAR(200) | Reporter name     |
| category    | VARCHAR(200) | Issue category    |
| issue       | TEXT         | Issue description |
| attachment  | VARCHAR(300) | Attachment        |
| source      | VARCHAR(300) | Source            |
| created_at  | TIMESTAMP    | Created date      |


```sql
CREATE TABLE report_issue (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    full_name VARCHAR(200),
    category VARCHAR(200),
    issue TEXT NOT NULL,
    attachment VARCHAR(300),
    source VARCHAR(300),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_report_issue_user
        FOREIGN KEY (user_id) REFERENCES cm_users(id)
);

```

### **Purpose**

Used for platform-level issue reporting such as bugs, abuse, payment issues, or UI problems.
This is separate from topic/reply reports and is meant for support and admin handling.

---

## **2️⃣2️⃣ notification_delivery_logs**

### **Schema (Table Format)**

| Column Name     | Type        | Description      |
| --------------- | ----------- | ---------------- |
| id              | BIGINT (PK) | Log ID           |
| notification_id | BIGINT (FK) | Notification     |
| channel         | ENUM        | Delivery channel |
| status          | ENUM        | Delivery status  |
| error_message   | TEXT        | Failure reason   |
| created_at      | TIMESTAMP   | Created date     |


```sql
CREATE TABLE notification_delivery_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    notification_id BIGINT NOT NULL,
    channel ENUM('EMAIL','IN_APP','PUSH') NOT NULL,
    status ENUM('SENT','FAILED','PENDING') DEFAULT 'PENDING',
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_ndl_notification
        FOREIGN KEY (notification_id) REFERENCES notifications(id)
);

```

### **Purpose**

Tracks the delivery status of notifications.
Helps monitor email failures, retries, and debugging notification delivery in production.

---

## **2️⃣3️⃣ topic_shares**

### **Schema (Table Format)**

| Column Name | Type        | Description     |
| ----------- | ----------- | --------------- |
| id          | BIGINT (PK) | Share ID        |
| topic_id    | BIGINT (FK) | Topic reference |
| user_id     | BIGINT (FK) | Shared by       |
| shared_via  | ENUM        | Platform        |
| created_at  | TIMESTAMP   | Share date      |


```sql
CREATE TABLE topic_shares (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    topic_id BIGINT NOT NULL,
    user_id BIGINT NULL,
    shared_via ENUM('LINK','WHATSAPP','EMAIL','TWITTER','OTHER') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_topic_shares_topic
        FOREIGN KEY (topic_id) REFERENCES topics(id),
    CONSTRAINT fk_topic_shares_user
        FOREIGN KEY (user_id) REFERENCES cm_users(id)
);

```

### **Purpose**

Stores topic sharing activity.
Used to calculate total shares, understand traffic sources, and improve engagement analytics.

---

## **2️⃣4️⃣ user_blocks**

### **Schema (Table Format)**

| Column Name  | Type        | Description   |
| ------------ | ----------- | ------------- |
| id           | BIGINT (PK) | Block ID      |
| blocked_by   | BIGINT (FK) | Blocking user |
| blocked_user | BIGINT (FK) | Blocked user  |
| created_at   | TIMESTAMP   | Block date    |


```sql
CREATE TABLE user_blocks (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    blocked_by BIGINT NOT NULL,
    blocked_user BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_blocked_pair (blocked_by, blocked_user),
    CONSTRAINT fk_user_blocks_by
        FOREIGN KEY (blocked_by) REFERENCES cm_users(id),
    CONSTRAINT fk_user_blocks_user
        FOREIGN KEY (blocked_user) REFERENCES cm_users(id)
);

```

### **Purpose**

Allows users to block or mute other users.
Blocked users’ content, replies, and notifications can be hidden for better community safety.

---

## **2️⃣5️⃣ moderation_logs**

### **Schema (Table Format)**

| Column Name | Type        | Description      |
| ----------- | ----------- | ---------------- |
| id          | BIGINT (PK) | Log ID           |
| admin_id    | BIGINT (FK) | Admin user       |
| entity_type | ENUM        | Entity type      |
| entity_id   | BIGINT      | Entity reference |
| action      | ENUM        | Action taken     |
| reason      | TEXT        | Action reason    |
| created_at  | TIMESTAMP   | Created date     |


```sql
CREATE TABLE moderation_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    admin_id BIGINT NOT NULL,
    entity_type ENUM('TOPIC','REPLY','USER') NOT NULL,
    entity_id BIGINT NOT NULL,
    action ENUM('DELETE','HIDE','WARN','BLOCK') NOT NULL,
    reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_moderation_admin
        FOREIGN KEY (admin_id) REFERENCES cm_users(id)
);

```
### **Purpose**

Tracks all admin or moderator actions performed on users, topics, or replies.
Essential for moderation transparency, audits, and future dispute handling.

---
