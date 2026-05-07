# inode — Claude Code Guide

inode is a CLI-first, AI-powered personal knowledge base and secret vault.
Users save notes, secrets, commands, and decisions — then retrieve them later
with natural language queries using RAG.

---

## Current Phase

**Phase 1 — Local MVP.** Everything runs locally. No server, no auth, single user.

- Language: Go
- SQLite + sqlite-vec for storage and vector search
- Claude API (`anthropic-sdk-go`) for LLM tasks
- Voyage AI for embeddings (custom HTTP client)
- AES-256-GCM encryption for sensitive notes (Go stdlib `crypto/aes`)

---

## Project Structure

```
inode/
├── main.go                        # entry point
├── go.mod / go.sum
│
├── cmd/                           # Cobra CLI commands
│   ├── root.go                    # root command, global flags
│   ├── add.go                     # inode add
│   ├── ask.go                     # inode ask
│   ├── list.go                    # inode list
│   ├── note.go                    # inode note get/edit/delete
│   └── config.go                  # inode config set/show
│
└── internal/
    ├── core/
    │   ├── notes.go               # NoteService
    │   ├── search.go              # SearchService (RAG pipeline)
    │   ├── tagger.go              # TaggerService (LLM auto-tagging)
    │   └── encryption.go          # AES-256-GCM + Argon2id key derivation
    ├── adapters/
    │   ├── llm/
    │   │   ├── adapter.go         # LLMAdapter interface
    │   │   └── claude.go          # Claude API implementation
    │   ├── embedding/
    │   │   ├── adapter.go         # EmbeddingAdapter interface
    │   │   └── voyage.go          # Voyage AI HTTP client
    │   └── db/
    │       ├── adapter.go         # DBAdapter interface
    │       └── sqlite.go          # SQLite + sqlite-vec implementation
    ├── config/
    │   └── settings.go            # Read/write ~/.inode/config.toml
    └── model/
        └── note.go                # Note struct, shared types
```

---

## Key Architecture Decisions

### Adapter Pattern
All external dependencies (LLM, embeddings, database) are behind Go interfaces
in `internal/adapters/*/adapter.go`. Never call `anthropic-sdk-go`, the Voyage AI
HTTP client, or `go-sqlite3` directly from `internal/core/` or `cmd/` — always go
through the adapter. This is what makes Phase 3 (Ollama) and Phase 2 (PostgreSQL)
possible without rewriting business logic.

### Encryption
Sensitive notes are encrypted with AES-256-GCM (Go stdlib) before being written to
the database. The key is derived via Argon2id (`golang.org/x/crypto/argon2`) from a
secret stored in the OS keychain (`go-keyring`). The key is never stored — rederived
per process from the keychain secret.

The `content_enc` column stores a single BLOB: `iv[12] + gcm_sealed[N]` where
`gcm_sealed` includes both the ciphertext and the 16-byte GCM auth tag.
The `content_plain` column is empty for sensitive notes. Do not change this layout.

**Linux headless fallback:** When `go-keyring` cannot reach a keyring service
(WSL, headless SSH), fall back to `~/.inode/.key` (chmod 600) and warn the user.

### RAG Pipeline
1. At `inode add`: call `EmbeddingAdapter.Embed(content)`, store vector in DB.
2. At `inode ask`: embed the query, call `DBAdapter.SearchSimilar()` for top-K notes,
   decrypt sensitive ones in memory, pass to `LLMAdapter.Answer()`.

The LLM receives only the top-K retrieved notes, not the full database.

### Config File
User config lives at `~/.inode/config.toml`. File permissions must be 600.
API keys stored in config are encrypted with the same keychain-derived key.
Never write plaintext secrets to the config file.

---

## Adapter Interfaces (do not change signatures)

```go
// internal/adapters/llm/adapter.go
type LLMAdapter interface {
    Classify(ctx context.Context, content string, categories []Category, tags []string) (ClassifyResult, error)
    Answer(ctx context.Context, query string, notes []model.Note) (string, error)
    Summarize(ctx context.Context, content string) (string, error)
}

// internal/adapters/embedding/adapter.go
type EmbeddingAdapter interface {
    Embed(ctx context.Context, text string) ([]float32, error)
}

// internal/adapters/db/adapter.go
type DBAdapter interface {
    Save(ctx context.Context, note *model.Note) (string, error)
    Get(ctx context.Context, id string) (*model.Note, error)
    Delete(ctx context.Context, id string) error
    SearchSimilar(ctx context.Context, vec []float32, topK int, filters Filters) ([]*model.Note, error)
    List(ctx context.Context, filters Filters, limit, offset int) ([]*model.Note, error)
}
```

---

## CLI Behavior Rules

- Sensitive values are always masked in output unless `--reveal` is passed.
- `--reveal` requires a `y/N` confirmation prompt before printing plaintext.
- `inode list --sensitive` also requires a confirmation prompt.
- Default for `IsSensitive` comes from `[defaults] sensitive` in config (default: `true`).
- `--sensitive` and `--no-sensitive` flags override the config default.

---

## What Not to Do

- Do not add features beyond Phase 1 scope (see `spec.md`).
- Do not call `anthropic-sdk-go`, Voyage AI, or SQLite directly from `cmd/` or `internal/core/` — always use adapters.
- Do not store plaintext sensitive content in the database.
- Do not store or cache the encryption key beyond the current process.
- Do not skip the confirmation prompt on `--reveal`.
- Do not use bcrypt — Argon2id is required (`golang.org/x/crypto/argon2`).
- Do not add a `server/` directory until Phase 2 work begins.

---

## Reference Documents

- `spec.md` — full product spec, CLI reference, data model, security model, tech stack
- `architecture.md` — layer diagram, folder structure, Go interfaces, data flows,
  encryption design, SQLite schema, Go module dependencies
