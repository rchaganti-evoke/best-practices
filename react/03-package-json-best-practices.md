## organize package json logically 
```json
{
  "name": "my-enterprise-app",
  "version": "2.1.5",
  "private": true,
  "description": "Enterprise React application for XYZ Corp",
  "author": "Engineering Team <engineering@example.com>",
  "license": "UNLICENSED",
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "scripts": {
    "// Development": "----------------------------------------",
    "dev": "vite",
    "dev:https": "vite --https",
    "// Building": "----------------------------------------",
    "build": "tsc && vite build",
    "build:staging": "vite build --mode staging",
    "build:production": "vite build --mode production",
    "// Testing": "----------------------------------------",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "// Linting & Formatting": "----------------------------------------",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,css,md}\"",
    "// Type Checking": "----------------------------------------",
    "type-check": "tsc --noEmit",
    "// Security": "----------------------------------------",
    "audit": "npm audit --audit-level=moderate",
    "audit:fix": "npm audit fix"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "@tanstack/react-query": "^5.12.0",
    "axios": "^1.6.0",
    "zustand": "^4.4.7"
  },
  "devDependencies": {
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.0.8",
    "typescript": "^5.2.2",
    "eslint": "^8.55.0",
    "prettier": "^3.1.1",
    "vitest": "^1.0.4"
  }
}
```

## use exact versions for critical dependencies 
```json
{
  "dependencies": {
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "axios": "1.6.2"
  },
  "devDependencies": {
    "vite": "5.0.8",
    "typescript": "5.3.3"
  }
}
```

## audit dependencies and regular updates
```bash
# Run security audit
npm audit
npm audit --audit-level=high  # Only high/critical

# Check for outdated packages
npm outdated

# Use automated tools
npm install -g npm-check-updates
ncu  # Check all updates
ncu -u  # Update package.json

# GitHub Dependabot (automated PRs)
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

# CI/CD pipeline - fail on vulnerabilities
- name: Security Audit
  run: npm audit --audit-level=high
```

## semantic Versioning is: MAJOR.MINOR.PATCH
```json
{
  "dependencies": {
    // Exact version - no updates
    "react": "18.2.0",
    
    // Caret (^) - allows minor and patch updates (recommended for most)
    "axios": "^1.6.0",  // Allows 1.6.0 to <2.0.0
    
    // Tilde (~) - allows only patch updates
    "lodash": "~4.17.21",  // Allows 4.17.21 to <4.18.0
    
    // Greater than or equal
    "typescript": ">=5.0.0 <6.0.0",
    
    // Latest (AVOID IN PRODUCTION)
    "some-lib": "latest"  // this is Non-deterministic never use like this
  }
}
```

## use lock files and commit them into repo i mean package-lock.json file always commit.







