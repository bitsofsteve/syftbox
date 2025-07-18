# Development

Get your development environment ready:

#### 1. Install Go (1.20+) & `just`

| Platform | Commands                                                                                                                                                 |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| macOS    | `brew install go just`                                                                                                                                   |
| Windows  | `winget install -e --id GoLang.Go --id Casey.Just`                                                                                                  |
| Linux    | Go: [Official Downloads](https://go.dev/dl/) <br> just: [Installation Guide](https://github.com/casey/just?tab=readme-ov-file#linux) |

Make sure to export `GOPATH` binaries correctly

```
export PATH="$(go env GOPATH)/bin:$PATH"
```

#### 2. Setup Project Tools

``` sh
just setup-toolchain
```

This will install the following CLI tools
- swaggo
- mkcert
- wgo

#### 3. Cross compiling CGO_ENABLED=1 (optional)

Install platform-specific build tools for cross-compiling CGO binaries.

Note: It's easier to cross compile from macOS -> other platforms

```sh
brew install FiloSottile/musl-cross/musl-cross mingw-w64
```

### IDE Setup (VSCode/Cursor Recommended)

Install the official Go extension. Then, add the following to your user `settings.json`:
```json
{
    "go.toolsManagement.autoUpdate": true,
    "gopls": {
        "ui.semanticTokens": true
    }
}
```

### Common Development Commands

List all available commands:
```sh
just
```

Run the client:
```sh
just run-client -e <email> -d <datadir> -s <server> -c <configpath>

# Or directly:
go run ./cmd/client -e <email> -d <datadir> -s <server> -c <configpath>
```

Run the server (using `./config/server.dev.yaml`):
```sh
just run-server -f ./config/server.dev.yaml

# Or directly:
go run ./cmd/server -f ./config/server.dev.yaml
```

Run the server with TLS (generates certs if needed):
```sh
just run-server-tls
# Or directly (ensure certs exist, e.g., via `just gen-certs`):

just gen-certs
go run ./cmd/server --cert certs/cert.pem --key certs/cert.key
```

Run a local server with two separate clients
```sh
go run ./cmd/server

go run ./cmd/client -e user1@email.com -s http://localhost:8080 -d ~/SyftBox.user1 -c ~/.syftbox/config.user1.json

go run ./cmd/client -e user2@email.com -s http://localhost:8080 -d ~/SyftBox.user2 -c ~/.syftbox/config.user2.json
```

Run tests:
```sh
just test
# (The `just test` command also generates an HTML coverage report)

# Or directly (basic tests):
go test ./...
```

### Working with MinIO (Local S3 Storage)

Start MinIO container:
```sh
just run-minio
```

Stop and delete MinIO container and data:
```sh
just destroy-minio # Use with caution!
```

SSH into the running MinIO container:
```sh
just ssh-minio
```

### Building Binaries

Build client binaries using GoReleaser (for configured targets):
```sh
just build-client
```

Build server binary using GoReleaser:
```sh
just build-server
```

Build all binaries using GoReleaser:
```sh
just build-all
```

Build client binary without GoReleaser:
```sh
# for your current OS/Arch
just build-client-target

# for a specific OS/Arch
just build-client-target linux amd64

# Or directly (non-prod build for current platform):
go build -o .out/syftbox_client ./cmd/client
```

### Release and Deployment

#### Production Release

**Prerequisites:**
- Repository write access
- SSH keys configured for production

**Steps:**
1. **Prepare**
   ```bash
   just show-version
   git status  # ensure clean
   git pull origin main
   ```

2. **Release**
   - Go to [GitHub Actions](https://github.com/OpenMined/syftbox/actions)
   - Select "Syftbox Release" workflow
   - Choose version type: `patch`, `minor`, or `major`
   - Click "Run workflow"

3. **Verify**
   - Check [releases page](https://github.com/OpenMined/syftbox/releases)
   - Monitor production environment

#### Development/Staging Deployment

**Steps:**
1. **Prepare**
   ```bash
   just test
   ```

2. **Deploy**
   - Go to [GitHub Actions](https://github.com/OpenMined/syftbox/actions)
   - Select "Syftbox Deploy" workflow
   - Choose environment: `dev` or `stage`
   - Click "Run workflow"

#### Version Management

**Version Types:**
- **Patch**: Bug fixes (1.2.3 → 1.2.4)
- **Minor**: New features (1.2.3 → 1.3.0)
- **Major**: Breaking changes (1.2.3 → 2.0.0)

**Local Commands:**
```bash
just show-version          # Show current version
just bump patch|minor|major # Update version files
just release patch|minor|major # Bump, commit, and tag
```

**Manual Deployment:**
```bash
just deploy user@hostname
```
