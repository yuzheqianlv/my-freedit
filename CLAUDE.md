# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Freedit is a forum application written in Rust that emphasizes safety and simplicity. It's designed to be the "safest and lightest forum, powered by rust" with no JavaScript dependencies for maximum security. The application uses an embedded database (sled) for easy deployment and includes features like:

- User management with roles and authentication
- Inn (subreddit-like) communities with posts and comments
- Solo (Twitter-like) personal posts
- End-to-end encrypted private messaging
- RSS feed reader
- File uploads and image galleries
- Full-text search via Tantivy
- Markdown and LaTeX support

## Common Development Commands

### Building and Running
```bash
# Development build and run
cargo run

# Release build
cargo build -r

# Run the release binary
./target/release/freedit
```

### Documentation
```bash
# Generate and open local documentation
cargo doc --no-deps --document-private-items --open
```

### Testing
Note: This project does not appear to have a traditional test suite. Testing is likely done manually or through integration testing.

## Architecture Overview

### Core Components

**Database Layer (`src/lib.rs`):**
- Uses sled embedded database for all data storage
- Global `DB` instance initialized via LazyLock
- Database trees for different data types (users, posts, sessions, etc.)

**Web Framework (`src/app_router.rs`):**
- Built with Axum web framework
- RESTful routing with clear URL patterns
- Middleware stack includes compression, timeouts, and tracing
- Static file serving for avatars, uploads, and assets

**Configuration (`src/config.rs`):**
- TOML-based configuration with sensible defaults
- Configurable paths for data storage, uploads, and database
- Auto-creates required directories on startup
- Supports custom config file via command line argument

**Controllers (`src/controller/`):**
- Modular design with separate controllers for each feature area:
  - `admin.rs` - Admin panel functionality
  - `user.rs` - User management, authentication, profiles
  - `inn.rs` - Community posts, comments, voting
  - `solo.rs` - Personal posts (Twitter-like)
  - `feed.rs` - RSS reader functionality
  - `message.rs` - End-to-end encrypted messaging
  - `upload.rs` - File upload and gallery management
  - `tantivy.rs` - Full-text search implementation

### Background Tasks (`src/main.rs`)

The application runs several background tasks:
- Session and captcha cleanup (every 5 minutes)
- RSS feed updates and audio downloads (every 4 hours)
- Full-text search index updates (real-time via sled watchers)
- Database snapshots (release mode only)

### Key Design Patterns

**Embedded Database Trees:**
- Different data types stored in separate sled trees
- Real-time indexing via sled event watchers
- No SQL migrations needed due to key-value nature

**Template Rendering:**
- Uses Askama for type-safe HTML templates
- Templates located in `templates/` directory
- Consistent layout with `layout.html` base template

**Static Assets:**
- CSS framework: Bulma
- No JavaScript for security (mentioned in README)
- Embedded static assets via `include_dir!` macro
- User uploads stored in configurable directories

## Configuration

The application reads from `config.toml` (or file specified as first argument):
- Database path and server address
- Upload directories for avatars, inn icons, files
- Search index and podcast storage paths
- Optional proxy settings and index rebuild flag

## Internationalization

The project supports multiple languages via TOML files in `i18n/`:
- English (`en.toml`)
- French (`fr.toml`) 
- Japanese (`ja.toml`)
- Chinese (`zh_cn.toml`)

## Security Features

- No client-side JavaScript by design
- End-to-end encryption for private messages
- CAPTCHA system for spam prevention
- Role-based access control
- Content sanitization via ammonia crate