---
name: twitter-query
description: Query Twitter/X data via RapidAPI twitter-api45.p.rapidapi.com. Supports user profiles, timelines, followers, following, replies, and tweet search. Triggered when user mentions "查询推特", "查推特", "twitter", or shares a Twitter/X link.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos]
prerequisites:
  commands: [curl]
  env_vars: [TWITTER_RAPIDAPI_KEY, TWITTER_RAPIDAPI_HOST]
metadata:
  hermes:
    tags: [twitter, x, social-media, rapidapi]
    homepage: https://github.com/Infatoshi/x-cli
---

# Twitter Query Skill

Query Twitter/X data via the RapidAPI `twitter-api45.p.rapidapi.com` API.

## Setup

On first use, the skill will prompt the user for:
- **RapidAPI Key** (`TWITTER_RAPIDAPI_KEY`)
- **RapidAPI Host** (`TWITTER_RAPIDAPI_HOST`) — defaults to `twitter-api45.p.rapidapi.com`

Credentials are stored in `~/.config/twitter-query/.env`.

## API Capabilities

| Endpoint | Function |
|----------|----------|
| `screenname.php` | Get user profile info |
| `timeline.php` | Get user's recent tweets |
| `followers.php` | Get user's followers list |
| `following.php` | Get user's following list |
| `search.php` | Search tweets (replies to a user, conversation threads) |
| `retweets.php` | Get retweets of a tweet |

**Not supported:** likes/favorites history, bookmarks, quote tweets.

## API Base

```
https://twitter-api45.p.rapidapi.com
```

Headers:
- `x-rapidapi-host: {TWITTER_RAPIDAPI_HOST}`
- `x-rapidapi-key: {TWITTER_RAPIDAPI_KEY}`

## Workflow

1. **Check/setup credentials** — run setup commands below, prompt user for RapidAPI key if missing
2. **Load credentials** — source `~/.config/twitter-query/.env`
3. **Parse user intent** — determine which endpoint to call based on the query
4. **Execute curl** — make the API call
5. **Parse JSON response** — extract relevant fields and format for the user

## Credential Setup (must run before any query)

Before using this skill, check if credentials exist:

```bash
mkdir -p ~/.config/twitter-query
if [ ! -f ~/.config/twitter-query/.env ]; then
  echo "TWITTER_RAPIDAPI_HOST=twitter-api45.p.rapidapi.com" > ~/.config/twitter-query/.env
fi
```

**If `TWITTER_RAPIDAPI_KEY` is missing or empty, use `clarify` to prompt the user:**

> 请提供你的 RapidAPI Key（用于 Twitter API）：
> 1. 登录 [RapidAPI](https://rapidapi.com)
> 2. 订阅 `twitter-api45` 包（免费层可用）
> 3. 复制你的 RapidAPI Key 并发送给我

After user provides the key, save it:
```bash
sed -i "s|TWITTER_RAPIDAPI_KEY=.*|TWITTER_RAPIDAPI_KEY=$NEW_KEY|" ~/.config/twitter-query/.env
chmod 600 ~/.config/twitter-query/.env
```

## Load Credentials

```bash
source ~/.config/twitter-query/.env 2>/dev/null
```

## Commands

### 1. Query User Profile

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/screenname.php?screenname=USERNAME"
```

Returns: name, bio, follower/following count, join date, verified status, etc.

### 2. Query User Timeline (recent tweets)

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/timeline.php?screenname=USERNAME"
```

Returns: list of recent tweets with IDs, text, timestamps, engagement metrics.

### 3. Get Followers

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/followers.php?screenname=USERNAME"
```

### 4. Get Following

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/following.php?screenname=USERNAME"
```

### 5. Get Replies to a User (search)

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/search.php?query=to:USERNAME&count=20"
```

### 6. Get Replies to a Tweet (conversation)

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/search.php?query=conversation_id:TWEET_ID&count=20"
```

### 7. Get Retweets of a Tweet

```bash
curl -s -H "x-rapidapi-host: $TWITTER_RAPIDAPI_HOST" -H "x-rapidapi-key: $TWITTER_RAPIDAPI_KEY" \
  "https://twitter-api45.p.rapidapi.com/retweets.php?tweet_id=TWEET_ID"
```

## Tweet ID / Link Extraction

To extract handle and tweet ID from a Twitter link like `https://x.com/GameFiLabs/status/123456`:

- Handle: `GameFiLabs` (between `/` and `/status/`)
- Tweet ID: `123456` (after `/status/`)

Regex: `x\.com/([^/]+)/status/([0-9]+)`

## Pitfalls

- **Endpoint not found (404):** That endpoint doesn't exist in this API — check the capability table
- **"Invalid credentials" or 403:** Check RapidAPI key is correct and hasn't expired
- **Rate limits:** RapidAPI twitter-api45 has usage limits; space out queries
- **No likes/favorites endpoint:** This API does not support reading user likes — inform the user
- **search.php returns `{"status":"failed"}`:** Some query types aren't supported; try `conversation_id:` or `to:` syntax
