
# Udiddit - A Social News Aggregator

## Introduction

Udiddit is a social news aggregation web content rating and discussion website currently using a risky and unreliable Postgres database schema to store forum posts, discussions, and votes made by users about various topics.

The schema allows posts to be created by registered users on certain topics and can include a URL or text content. It also allows registered users to cast an upvote (like) or downvote (dislike) for any forum post and add comments.

Here is the DDL used to create the schema:

```sql
CREATE TABLE bad_posts (
    id SERIAL PRIMARY KEY,
    topic VARCHAR(50),
    username VARCHAR(50),
    title VARCHAR(150),
    url VARCHAR(4000) DEFAULT NULL,
    text_content TEXT DEFAULT NULL,
    upvotes TEXT,
    downvotes TEXT
);

CREATE TABLE bad_comments (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    post_id BIGINT,
    text_content TEXT
);
```

## Part I: Investigate the Existing Schema

Investigate this schema and some sample data in the project’s SQL workspace. In your own words, outline three (3) specific things that could be improved:

1. **Normalization of Username and Topic Fields**: The current schema is partially denormalized, especially with the `username` field appearing in both the `bad_posts` and `bad_comments` tables. The `topic` field in the `bad_posts` table is likely to have repeated values.

2. **Improved Use of Constraints and Data Types**: The schema lacks constraints like `FOREIGN KEY` relationships and could use optimized data types. For example, `upvotes` and `downvotes` are stored as `TEXT`, which is inefficient. The `username` in both tables should be set to `NOT NULL` to enforce data integrity.

3. **Lack of Indexing for Optimizing Queries**: The schema does not use indexing, leading to inefficient queries as the dataset grows. Frequently queried columns like `username`, `topic`, and `post_id` are prime candidates for indexing.

## Part II: Create the DDL for Your New Schema

Based on the initial investigation, create a new schema for Udiddit. The new schema should address the shortcomings outlined earlier. Guidelines include:

- Allow users to register with unique usernames (max 25 characters, not empty).
- Allow users to create new topics with unique names (max 30 characters, optional description).
- Posts must have a title (max 100 characters, either URL or text content, not both).
- Allow users to comment on posts with threaded comments and associate votes with users and posts.

Here is the DDL for the new schema:

```sql
-- Create users table
CREATE TABLE users (
    id SERIAL,
    username VARCHAR(25) UNIQUE NOT NULL,
    last_logon TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_users_id PRIMARY KEY (id),
    CONSTRAINT ck_users_username CHECK (LENGTH(TRIM(username)) > 0)
);

-- Create topics table
CREATE TABLE topics (
    id SERIAL,
    topic_name VARCHAR(30) UNIQUE NOT NULL,
    topic_description VARCHAR(500),
    user_id INT NOT NULL,
    CONSTRAINT pk_topics_id PRIMARY KEY (id),
    CONSTRAINT fk_topics_user_id FOREIGN KEY (user_id)
        REFERENCES users (id),
    CONSTRAINT ck_topics_topic_name CHECK (LENGTH(TRIM(topic_name)) > 0)
);

-- Create posts table
CREATE TABLE posts (
    id SERIAL,
    post_title VARCHAR(100) NOT NULL,
    url TEXT,
    post_content TEXT,
    topic_id INT NOT NULL,
    user_id INT NOT NULL,
    created_on TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_posts PRIMARY KEY (id),
    CONSTRAINT fk_posts_topic_id FOREIGN KEY (topic_id)
        REFERENCES topics (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_posts_user_id FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE SET NULL,
    CONSTRAINT ck_posts_url_xor_post_content 
        CHECK ((url IS NOT NULL AND post_content IS NULL)
        OR (post_content IS NOT NULL AND url IS NULL)),
    CONSTRAINT ck_posts_post_title CHECK (LENGTH(TRIM(post_title)) > 0)
);

-- Create comments table
CREATE TABLE comments (
    id SERIAL,
    comment_content TEXT NOT NULL,
    parent_id INT,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    created_on TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_comments PRIMARY KEY (id),
    CONSTRAINT fk_comments_parent_id FOREIGN KEY (parent_id)
        REFERENCES comments (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_comments_post_id FOREIGN KEY (post_id)
        REFERENCES posts (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_comments_user_id FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE SET NULL,
    CONSTRAINT ck_comments_comment_content 
        CHECK (LENGTH(TRIM(comment_content)) > 0)
);

-- Create votes table
CREATE TABLE votes (
    id SERIAL,
    up_vote INT,
    down_vote INT,
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    CONSTRAINT pk_votes PRIMARY KEY (id),
    CONSTRAINT fk_votes_user_id FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE SET NULL,
    CONSTRAINT fk_votes_post_id FOREIGN KEY (post_id)
        REFERENCES posts (id)
        ON DELETE CASCADE,
    CONSTRAINT vote_value CHECK ((up_vote = 1 AND down_vote IS NULL)
        OR (down_vote = -1 AND up_vote IS NULL)),
    CONSTRAINT unique_votes UNIQUE (user_id, post_id)
);
```

## Part III: Migrate the Provided Data

To migrate the data from the provided schema, use the following DML queries:

```sql
-- All users who've made posts
INSERT INTO users (username)
    SELECT DISTINCT username
    FROM bad_posts;

-- All users who've only commented
INSERT INTO users (username)
    SELECT DISTINCT bc.username
    FROM bad_comments bc
    LEFT JOIN users u
    ON bc.username = u.username
    WHERE u.username IS NULL;

-- Users who've only upvoted
WITH table1 AS (SELECT REGEXP_SPLIT_TO_TABLE(upvotes, ',') AS upvote FROM bad_posts)
INSERT INTO users (username)
    SELECT DISTINCT upvote
    FROM table1
    LEFT JOIN users u
    ON table1.upvote = u.username
    WHERE u.username IS NULL;

-- Users who've only downvoted
WITH table1 AS (SELECT REGEXP_SPLIT_TO_TABLE(downvotes, ',') AS downvote FROM bad_posts)
INSERT INTO users (username)
    SELECT DISTINCT downvote
    FROM table1
    LEFT JOIN users u
    ON table1.downvote = u.username
    WHERE u.username IS NULL;

-- Inserting data into topics table
INSERT INTO topics (topic_name, user_id)
    SELECT DISTINCT ON (topic) bp.topic, u.id
    FROM bad_posts AS bp
    JOIN users AS u ON u.username = bp.username;

-- Inserting data into posts table
INSERT INTO posts (post_title, url, post_content, topic_id, user_id)
    SELECT LEFT(bp.title, 100), bp.url, bp.text_content, t.id, u.id
    FROM bad_posts AS bp
    JOIN topics AS t ON bp.topic = t.topic_name
    JOIN users AS u ON bp.username = u.username;

-- Inserting data into comments table
INSERT INTO comments (comment_content, post_id, user_id)
    SELECT bc.text_content, p.id, u.id
    FROM bad_comments AS bc
    JOIN bad_posts AS bp ON bc.post_id = bp.id
    JOIN posts AS p ON bp.id = p.id
    JOIN users AS u ON bc.username = u.username;

-- Inserting data into votes table
WITH up_vote_list AS (
    SELECT id AS post_id, regexp_split_to_table(upvotes, ',') AS username
    FROM bad_posts
)
INSERT INTO votes (up_vote, user_id, post_id)
    SELECT 1, u.id, p.id
    FROM up_vote_list AS uvl
    JOIN posts AS p ON uvl.post_id = p.id
    JOIN users AS u ON LOWER(TRIM(u.username)) = LOWER(TRIM(uvl.username));

WITH down_vote_list AS (
    SELECT id AS post_id, regexp_split_to_table(downvotes, ',') AS username
    FROM bad_posts
)
INSERT INTO votes (down_vote, user_id, post_id)
    SELECT -1, u.id, p.id
    FROM down_vote_list AS dvl
    JOIN posts AS p ON dvl.post_id = p.id
    JOIN users AS u ON LOWER(TRIM(u.username)) = LOWER(TRIM(dvl.username));
```
