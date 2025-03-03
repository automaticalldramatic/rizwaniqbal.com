[![firebase-deploy](https://github.com/automaticalldramatic/rizwaniqbal.com/actions/workflows/main.yml/badge.svg)](https://github.com/automaticalldramatic/rizwaniqbal.com/actions/workflows/main.yml)

# rizwaniqbal.com

Static website for rizwaniqbal.com built on Hugo

## Local Development

### Prerequisites
- [Hugo Extended](https://gohugo.io/installation/) (latest version)
- Git

### Setup
1. Clone the repository
   ```bash
   git clone https://github.com/automaticalldramatic/rizwaniqbal.com.git
   cd rizwaniqbal.com
   ```

2. Initialize and update submodules (for themes)
   ```bash
   git submodule init
   git submodule update
   ```

### Running Locally
Start the Hugo development server:
```bash
hugo server -D
```

This will start a local server at http://localhost:1313/ with live reloading enabled. The `-D` flag includes draft posts.

### Creating New Content
To create a new blog post:
```bash
hugo new posts/my-new-post.md
```

## Deployment

This site is automatically deployed to Firebase Hosting using GitHub Actions. The deployment workflow is triggered when:

1. A new tag with the format `v*` is pushed to the repository
2. A pull request is made to the `master` branch (build only, no deployment)

The deployment process:
1. Builds the site using Hugo with minification
2. Authenticates with Google Cloud using Firebase service credentials
3. Deploys to Firebase Hosting on the live channel

To deploy a new version:
```bash
git tag v1.x.x
git push origin v1.x.x
```

See `.github/workflows/main.yml` for the complete workflow configuration.
