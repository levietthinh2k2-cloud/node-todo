# GitHub Actions Workflows

This directory contains GitHub Actions workflows for the Node.js Todo application.

## Workflows

### 1. CI Pipeline (`ci.yml`)
**Trigger**: Push or PR to `main` or `develop` branches

**What it does**:
- Tests the application on Node.js versions 14.x, 16.x, and 18.x
- Runs MongoDB service for testing
- Installs dependencies and runs linter
- Executes tests (if configured)
- Performs code quality and security checks

### 2. Docker Build & Push (`docker-build.yml`)
**Trigger**: Push to `main` branch, tags starting with `v`, or manual trigger

**What it does**:
- Builds Docker image using Dockerfile
- Pushes to GitHub Container Registry (ghcr.io)
- Tags images with branch name, commit SHA, and version
- Optional: Can push to Docker Hub (disabled by default)

### 3. Deploy Application (`deploy.yml`)
**Trigger**: After successful Docker build, or manual trigger

**What it does**:
- Connects to your server via SSH
- Copies docker-compose.yml to the server
- Pulls latest images and restarts containers
- Performs health check
- Cleans up old Docker images
- Sends notifications (optional)

## Setup Instructions

### Prerequisites
1. GitHub repository with this code
2. A server for deployment (VPS, EC2, etc.)
3. Docker and Docker Compose installed on the server

### Required GitHub Secrets

Go to your repository → Settings → Secrets and variables → Actions, and add:

#### For Deployment (deploy.yml):
- `SSH_PRIVATE_KEY` - Your SSH private key for server access
- `SSH_USER` - SSH username (e.g., `ubuntu`, `root`)
- `SERVER_HOST` - Server IP address or domain
- `SSH_PORT` - (Optional) SSH port, default is 22

#### For Docker Hub (optional):
- `DOCKERHUB_USERNAME` - Your Docker Hub username
- `DOCKERHUB_TOKEN` - Docker Hub access token

### Setting up SSH Access

1. **Generate SSH key pair** (on your local machine):
   ```bash
   ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions
   ```

2. **Copy public key to your server**:
   ```bash
   ssh-copy-id -i ~/.ssh/github_actions.pub user@your-server-ip
   ```

3. **Add private key to GitHub Secrets**:
   ```bash
   cat ~/.ssh/github_actions
   ```
   Copy the entire output and add as `SSH_PRIVATE_KEY` secret

### Server Setup

1. **Install Docker and Docker Compose** on your server:
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   
   # Install Docker Compose
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

2. **Create deployment directory**:
   ```bash
   mkdir -p ~/node-todo
   ```

3. **Configure firewall** (if applicable):
   ```bash
   sudo ufw allow 8081/tcp
   sudo ufw allow 22/tcp
   ```

## Customization

### Change Node.js versions
Edit `ci.yml`, modify the matrix:
```yaml
strategy:
  matrix:
    node-version: [16.x, 18.x, 20.x]  # Add or remove versions
```

### Enable Docker Hub push
Edit `docker-build.yml`, change:
```yaml
if: false  # Change to: if: true
```

### Add notifications
Edit `deploy.yml` in the `notify` job, add:
```yaml
- name: Send Slack notification
  uses: slackapi/slack-github-action@v1.24.0
  with:
    payload: |
      {
        "text": "Deployment ${{ needs.deploy.result }}"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## Monitoring

### View workflow runs
Go to: Repository → Actions tab

### Check deployment logs
Click on any workflow run → Select job → View logs

### Manual deployment
Go to: Actions → Deploy Application → Run workflow

## Troubleshooting

### SSH connection fails
- Verify `SSH_PRIVATE_KEY` is correctly formatted (includes BEGIN/END lines)
- Check server firewall allows SSH (port 22 or custom port)
- Ensure SSH user has Docker permissions: `sudo usermod -aG docker $USER`

### Docker image pull fails
- Check if image was pushed successfully in docker-build workflow
- Verify GitHub Container Registry permissions
- Ensure server can access ghcr.io

### Application health check fails
- Check if MongoDB is running: `docker compose ps`
- View application logs: `docker compose logs app`
- Verify port 8081 is not in use: `netstat -tuln | grep 8081`

## Best Practices

1. **Use environment-specific secrets** for staging vs production
2. **Enable branch protection** on main branch
3. **Require CI to pass** before merging PRs
4. **Review security alerts** from npm audit
5. **Tag releases** with semantic versioning (v1.0.0, v1.1.0, etc.)
6. **Monitor workflow execution times** and optimize if needed
7. **Keep dependencies updated** regularly

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
