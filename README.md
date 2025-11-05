# OpenCode DevContainer Base Image

A security-hardened, production-ready DevContainer base image featuring OpenCode CLI for AI-assisted development. Built on Node.js 25 with network isolation, essential development tools, and persistent configuration support.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Hub-blue.svg)](https://hub.docker.com/r/vegastyle/opencode-devcontainer-base)
[![OpenCode](https://img.shields.io/badge/OpenCode-AI-purple.svg)](https://github.com/opencode-ai/opencode)

## Overview

This project provides a **customizable base DevContainer image** that integrates OpenCode, an open-source AI coding assistant, into a secure Linux development environment. Unlike using DevContainer features, this approach gives you explicit control over your environment configuration through Docker, making it easier to:

- Customize dependencies and tools
- Mount your own OpenCode settings to `/home/node/.opencode`
- Extend the base image with additional runtimes or services
- Control security policies and firewall rules
- Version your development environment


## Key Features

- **OpenCode CLI** - AI-assisted coding with support for multiple providers (Anthropic, OpenAI, OpenRouter)
- **Network Isolation** - Custom firewall with default-deny and explicit whitelist
- **Node.js 25** - Latest LTS with optimized memory settings
- **Development Tools** - git, GitHub CLI, fzf, nano, vim, jq, and more
- **Enhanced Shell** - zsh with powerline10k theme and git-delta for better diffs
- **VS Code Extensions** - Pre-configured with OpenCode, ESLint, Prettier, and GitLens
- **Persistent Storage** - Command history and OpenCode configuration preserved across rebuilds
- **Security Hardened** - Non-root user, capability-based permissions, monitored outbound connections

## Prerequisites

Before you begin, ensure you have:

1. **Docker Desktop** or Docker CLI installed and running
   - [Install Docker Desktop](https://www.docker.com/products/docker-desktop/)

2. **Visual Studio Code** with Dev Containers extension
   - [Install VS Code](https://code.visualstudio.com/)
   - [Install Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

3. **OpenCode API Keys** - At least one of:
   - Anthropic API key ([Get one here](https://console.anthropic.com/))
   - OpenAI API key ([Get one here](https://platform.openai.com/api-keys))
   - OpenRouter API key ([Get one here](https://openrouter.ai/))

4. **Basic Understanding** of DevContainers
   - [DevContainers Tutorial](https://code.visualstudio.com/docs/devcontainers/tutorial)

## Quick Start

### Step 1: Create Your Project

```bash
mkdir my-opencode-project
cd my-opencode-project
```

### Step 2: Create DevContainer Configuration

Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "My OpenCode Project",
  "image": "vegastyle/opencode-devcontainer-base:latest",

  "capAdd": ["NET_ADMIN", "NET_RAW"],
  "remoteUser": "node",

  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh",

  "customizations": {
    "vscode": {
      "extensions": [
        "sst-dev.opencode"
      ]
    }
  }
}
```

### Step 3: Configure OpenCode

Create your OpenCode authentication file locally:

```bash
# Linux/macOS
mkdir -p ~/.local/share/opencode
nano ~/.local/share/opencode/auth.json
```

Add your API keys:

```json
{
  "anthropic": "your-anthropic-api-key",
  "openai": "your-openai-api-key"
}
```

**Important:** Never commit `auth.json` to version control!

### Step 4: Open in Container

1. Open VS Code
2. Press `F1` or `Ctrl+Shift+P` (Windows/Linux) / `Cmd+Shift+P` (macOS)
3. Type "Dev Containers: Reopen in Container"
4. Wait for the container to build and start

### Step 5: Verify Installation

Once inside the container, open the integrated terminal and run:

```bash
opencode --version
```

You should see the OpenCode version number, confirming successful installation.

## Installation Methods

### Method 1: Using Pre-built Image (Recommended)

This is the fastest way to get started. The image is already built and hosted on Docker Hub.

**Basic Configuration** (`.devcontainer/devcontainer.json`):

```json
{
  "name": "My Project",
  "image": "vegastyle/opencode-devcontainer-base:latest",

  "capAdd": ["NET_ADMIN", "NET_RAW"],
  "remoteUser": "node",

  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh",

  "mounts": [
    "source=${localEnv:HOME}${localEnv:USERPROFILE}/.local/share/opencode,target=/home/node/.local/share/opencode,type=bind,consistency=cached"
  ],

  "customizations": {
    "vscode": {
      "extensions": [
        "sst-dev.opencode",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "eamodio.gitlens"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "terminal.integrated.defaultProfile.linux": "zsh"
      }
    }
  },

  "containerEnv": {
    "DEVCONTAINER": "true",
    "NODE_OPTIONS": "--max-old-space-size=4096"
  }
}
```

### Method 2: Building from Source

If you want to customize the base image itself, you can build from the Dockerfile.

**Step 1:** Clone this repository

```bash
git clone https://github.com/vega-style/dev-containers.git
cd dev-containers
```

**Step 2:** Create your devcontainer configuration (`.devcontainer/devcontainer.json`):

```json
{
  "name": "My Custom OpenCode",
  "build": {
    "dockerfile": "../path/to/opencode/.devcontainer/dockerfile",
    "args": {
      "OPENCODE_VERSION": "latest"
    }
  },

  "capAdd": ["NET_ADMIN", "NET_RAW"],
  "remoteUser": "node",

  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh"
}
```

**Step 3:** Customize the Dockerfile

You can modify `opencode/.devcontainer/dockerfile` to:
- Install additional tools
- Change the Node.js version
- Add language runtimes (Python, Go, Rust, etc.)
- Modify the OpenCode version

**Step 4:** Build and open

VS Code will automatically build the image when you reopen in the container.

## Configuration Guide

### Customizing OpenCode Settings

OpenCode stores user settings in `/home/node/.opencode`. To customize:

**Option 1: Mount your local settings**

```json
{
  "mounts": [
    "source=${localWorkspaceFolder}/.opencode,target=/home/node/.opencode,type=bind"
  ]
}
```

Create `.opencode/config.json` in your project:

```json
{
  "defaultProvider": "anthropic",
  "model": "claude-sonnet-4-5-20250929",
  "maxTokens": 8000
}
```

**Option 2: Copy settings in a custom Dockerfile**

```dockerfile
FROM vegastyle/opencode-devcontainer-base:latest

COPY .opencode /home/node/.opencode
RUN sudo chown -R node:node /home/node/.opencode
```

### Adding VS Code Extensions

Extend the `customizations.vscode.extensions` array:

```json
{
  "customizations": {
    "vscode": {
      "extensions": [
        "sst-dev.opencode",
        "dbaeumer.vscode-eslint",
        "ms-python.python",
        "golang.go",
        "rust-lang.rust-analyzer"
      ]
    }
  }
}
```

### Modifying Environment Variables

Add or override environment variables:

```json
{
  "containerEnv": {
    "NODE_OPTIONS": "--max-old-space-size=8192",
    "EDITOR": "vim",
    "MY_CUSTOM_VAR": "value"
  }
}
```

### Customizing Firewall Rules

The firewall script is located at `/usr/local/bin/init-firewall.sh`. To modify it:

**Option 1: Override in custom Dockerfile**

```dockerfile
FROM vegastyle/opencode-devcontainer-base:latest

COPY my-custom-firewall.sh /usr/local/bin/init-firewall.sh
RUN sudo chmod +x /usr/local/bin/init-firewall.sh
```

**Option 2: Extend the existing script**

Create `custom-firewall.sh`:

```bash
#!/bin/bash
# Run the base firewall first
sudo /usr/local/bin/init-firewall.sh

# Add your custom rules
sudo iptables -A OUTPUT -d $(dig +short my-custom-domain.com) -j ACCEPT
```

Then use it in `postStartCommand`:

```json
{
  "postStartCommand": "bash custom-firewall.sh"
}
```

### Extending the Base Image

Create a custom Dockerfile that extends the base:

```dockerfile
FROM vegastyle/opencode-devcontainer-base:latest

# Install Python
USER root
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
USER node
RUN pip3 install --user numpy pandas matplotlib

# Copy custom OpenCode settings
COPY --chown=node:node .opencode /home/node/.opencode

# Add custom shell aliases
RUN echo "alias py='python3'" >> /home/node/.zshrc
```

## Security Features

### Network Isolation Firewall

This DevContainer includes a **default-deny firewall** that only allows outbound connections to:

- **GitHub** - Web, API, and Git operations
- **npm Registry** - Package downloads
- **Anthropic API** - OpenCode AI requests
- **VS Code Services** - Extension marketplace, updates
- **Monitoring** - Sentry, Statsig (error reporting)
- **DNS & SSH** - For resolution and Git operations
- **Local Network** - Host access for Docker functionality
- **Localhost** - Loopback traffic

### Security Considerations

> **Warning:** This container requires elevated network capabilities (`NET_ADMIN`, `NET_RAW`) to configure the firewall. While this provides strong network isolation, it also means:
>
> - Malicious code in the container could modify firewall rules
> - The container has more privileges than a standard container
> - Only use this with **trusted repositories**

### Why These Capabilities Are Needed

- `NET_ADMIN` - Allows the firewall script to configure iptables rules
- `NET_RAW` - Enables packet filtering and network monitoring

### Best Practices

1. **Never commit API keys** - Use mounted volumes for `auth.json`
2. **Review the firewall script** - Understand what connections are allowed
3. **Use with trusted code** - Don't run untrusted code in this container
4. **Keep the base image updated** - Pull the latest version regularly
5. **Monitor outbound connections** - Check container logs for unexpected traffic

## Project Structure

```
dev-containers/
├── .github/
│   └── workflows/
│       └── on_push.yml           # Semantic versioning workflow
├── opencode/
│   └── .devcontainer/
│       ├── devcontainer.json     # Base DevContainer configuration
│       ├── dockerfile            # Base image build instructions
│       └── init-firewall.sh      # Network isolation script
├── readme-examples/              # Example README files for reference
├── LICENSE                       # Apache 2.0 License
└── README.md                     # This file
```

### File Descriptions

- **`dockerfile`** - Defines the base image with Node.js, OpenCode, and development tools
- **`devcontainer.json`** - Reference configuration for the DevContainer
- **`init-firewall.sh`** - Configures iptables for network isolation
- **`on_push.yml`** - Automates version bumping using semantic versioning

## How It Works

### Container Startup Sequence

1. **Image Pull/Build** - Docker pulls `vegastyle/opencode-devcontainer-base` or builds from Dockerfile
2. **Container Creation** - VS Code creates the container with specified capabilities
3. **Volume Mounting** - Persistent volumes for bash history and OpenCode config are attached
4. **User Switch** - Container runs as non-root `node` user
5. **Post-Start Command** - Firewall script initializes network rules
6. **Extension Installation** - VS Code installs specified extensions
7. **Ready** - Environment is ready for development

### Firewall Initialization

The firewall script (`init-firewall.sh`) runs at startup and:

1. **Preserves DNS** - Saves Docker DNS rules before flushing
2. **Creates IP Sets** - Uses ipset for efficient CIDR matching
3. **Resolves Domains** - Dynamically gets IPs for allowed domains
4. **Aggregates Ranges** - Combines IPs into efficient CIDR blocks
5. **Applies Rules** - Sets iptables to default-deny with whitelist
6. **Verifies** - Tests that blocked domains fail and allowed domains succeed

### OpenCode Integration

OpenCode CLI is installed globally via npm and configured to:

- Store global settings in `/home/node/.config/opencode`
- Store user settings in `/home/node/.opencode` (customizable)
- Use environment variables for API keys (optional)
- Integrate with VS Code through the `sst-dev.opencode` extension

## Customization Examples

### Example 1: Adding Python Support

Create a custom Dockerfile:

```dockerfile
FROM vegastyle/opencode-devcontainer-base:latest

USER root

# Install Python and common tools
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    && rm -rf /var/lib/apt/lists/*

USER node

# Install common Python packages
RUN pip3 install --user \
    numpy \
    pandas \
    matplotlib \
    jupyter \
    pytest
```

Update `.devcontainer/devcontainer.json`:

```json
{
  "build": {
    "dockerfile": "Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "sst-dev.opencode",
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  }
}
```

### Example 2: Full-Stack Development (Node + PostgreSQL)

Use Docker Compose (`.devcontainer/docker-compose.yml`):

```yaml
version: '3.8'

services:
  app:
    image: vegastyle/opencode-devcontainer-base:latest
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Update `.devcontainer/devcontainer.json`:

```json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace"
}
```

### Example 3: Mounting Custom OpenCode Configuration

Create `.opencode/config.json` in your project:

```json
{
  "defaultProvider": "anthropic",
  "model": "claude-sonnet-4-5-20250929",
  "maxTokens": 8000,
  "temperature": 0.7,
  "streaming": true
}
```

Mount it in `.devcontainer/devcontainer.json`:

```json
{
  "mounts": [
    "source=${localWorkspaceFolder}/.opencode,target=/home/node/.opencode,type=bind,consistency=cached"
  ]
}
```

### Example 4: Adding Custom Domain to Firewall

Create `custom-firewall-additions.sh`:

```bash
#!/bin/bash

# Get IP for custom domain
CUSTOM_DOMAIN="myapi.example.com"
CUSTOM_IPS=$(dig +short "$CUSTOM_DOMAIN" | grep -E '^[0-9.]+$')

# Add to firewall
for ip in $CUSTOM_IPS; do
    sudo iptables -A OUTPUT -d "$ip" -j ACCEPT
    echo "Allowed outbound to $CUSTOM_DOMAIN ($ip)"
done
```

Update `.devcontainer/devcontainer.json`:

```json
{
  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh && bash .devcontainer/custom-firewall-additions.sh"
}
```

## Troubleshooting

### Issue: Firewall Verification Failed

**Symptom:**
```
Verification failed! example.com should be blocked but was accessible.
```

**Solution:**
- The firewall script couldn't properly block traffic
- Check Docker network settings
- Ensure `NET_ADMIN` and `NET_RAW` capabilities are granted
- Try running `sudo /usr/local/bin/init-firewall.sh` manually in the container

### Issue: OpenCode Command Not Found

**Symptom:**
```
bash: opencode: command not found
```

**Solution:**
- OpenCode may not be installed properly
- Check if you're using the correct base image
- Verify the Dockerfile includes the OpenCode installation step
- Try running `npm list -g opencode-ai` to check installation

### Issue: API Authentication Failed

**Symptom:**
```
Error: API key not found or invalid
```

**Solution:**
- Verify your `auth.json` file exists and contains valid keys
- Check the file is mounted correctly to `/home/node/.local/share/opencode/auth.json`
- Ensure the JSON syntax is correct (no trailing commas)
- Test your API key directly at the provider's website

### Issue: Permission Denied Errors

**Symptom:**
```
Error: EACCES: permission denied
```

**Solution:**
- The container runs as the `node` user (non-root)
- Files created on the host may have incorrect permissions
- Fix with: `sudo chown -R node:node /path/to/files` inside the container
- Or on host: `chmod -R 755 /path/to/files`

### Issue: Cannot Connect to External Service

**Symptom:**
```
Error: connect ETIMEDOUT
```

**Solution:**
- The firewall blocks connections by default
- Check if the domain is in the whitelist in `init-firewall.sh`
- Add the domain to the firewall whitelist (see Customization Examples)
- Verify DNS resolution: `dig +short domain.com`

### Issue: Container Won't Start

**Symptom:**
Container fails to start or exits immediately

**Solution:**
- Check Docker logs: `docker logs <container-id>`
- Verify Docker has enough resources (memory, disk space)
- Try removing old containers: `docker container prune`
- Check for conflicting ports
- Ensure Docker Desktop is running

### Issue: Extensions Not Installing

**Symptom:**
VS Code extensions listed in `devcontainer.json` don't install

**Solution:**
- Check internet connection and firewall rules
- Verify `marketplace.visualstudio.com` is allowed in firewall
- Try installing manually: `Ctrl+Shift+X` → search → install
- Check VS Code output panel for error messages

## Related Resources

### OpenCode
- [OpenCode GitHub Repository](https://github.com/opencode-ai/opencode)
- [OpenCode Documentation](https://opencode.dev/docs)
- [OpenCode VS Code Extension](https://marketplace.visualstudio.com/items?itemName=sst-dev.opencode)

### DevContainers
- [VS Code Dev Containers Documentation](https://code.visualstudio.com/docs/devcontainers/containers)
- [DevContainer Specification](https://containers.dev/)
- [DevContainers Tutorial](https://code.visualstudio.com/docs/devcontainers/tutorial)

### Docker
- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub - Base Image](https://hub.docker.com/r/vegastyle/opencode-devcontainer-base)
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)

### Claude Code (Official)
- [Claude Code Official DevContainer](https://github.com/anthropics/claude-code-devcontainer)
- [Claude API Documentation](https://docs.anthropic.com/)

### Security
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Linux Capabilities Explained](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [iptables Tutorial](https://www.netfilter.org/documentation/)

## Contributing

Contributions are welcome! Here's how you can help:

1. **Report Issues** - Found a bug or have a feature request? [Open an issue](https://github.com/your-username/dev-containers/issues)

2. **Submit Pull Requests**
   - Fork the repository
   - Create a feature branch: `git checkout -b feature/my-feature`
   - Commit your changes: `git commit -m "Add my feature"`
   - Push to the branch: `git push origin feature/my-feature`
   - Open a Pull Request

3. **Improve Documentation** - Help make this README better by fixing typos, adding examples, or clarifying instructions

4. **Share Examples** - Created a cool customization? Share it in the discussions or add it to the examples section

### Development Guidelines

- Follow existing code style and conventions
- Test your changes in a clean DevContainer environment
- Update documentation for any new features
- Add comments for complex logic
- Ensure the firewall script still works after modifications

## License

This project is licensed under the **Apache License 2.0** - see the [LICENSE](LICENSE) file for details.

---

**Made with ❤️ for developers who want secure, AI-assisted development environments**

*Questions? Issues? [Open an issue](https://github.com/your-username/dev-containers/issues) or reach out to the community!*
