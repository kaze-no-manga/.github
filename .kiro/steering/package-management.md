# Package Management

## NPM Public Packages

All shared packages are published as **public packages** on NPM under the `@kaze-no-manga` scope.

### Published Packages

- `@kaze-no-manga/brand` - Design tokens + Tailwind preset
- `@kaze-no-manga/models` - TypeScript types + Zod schemas + GraphQL SDL
- `@kaze-no-manga/scraper` - Manga source scrapers

### Why Public?

1. **Open Source**: Contribute to the community
2. **Simplicity**: No authentication required for installation
3. **Reusability**: Other manga projects can use our packages
4. **Visibility**: Showcase our work
5. **Zero Cost**: Free on NPM

## Package Structure

Every NPM package MUST follow this structure:

```
@kaze-no-manga/package-name/
├── src/                    # Source code (TypeScript)
│   ├── index.ts           # Main entry point
│   └── ...                # Other source files
├── dist/                   # Compiled output (gitignored, npm-included)
│   ├── index.js           # Compiled JavaScript
│   ├── index.d.ts         # Type definitions
│   └── ...
├── tests/                  # Tests (optional, not published)
├── package.json            # Package manifest
├── tsconfig.json           # TypeScript configuration
├── README.md               # Package documentation
├── LICENSE                 # MIT License
├── .gitignore              # Git exclusions
└── .npmignore              # NPM publish exclusions
```

## Package.json Configuration

### Required Fields

```json
{
  "name": "@kaze-no-manga/package-name",
  "version": "0.1.0",
  "description": "Brief description of the package",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "files": [
    "dist",
    "README.md",
    "LICENSE"
  ],
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:watch": "vitest",
    "prepublishOnly": "npm run build && npm test"
  },
  "keywords": [
    "manga",
    "tracker",
    "kaze-no-manga"
  ],
  "author": "Kaze no Manga Team",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/kaze-no-manga/package-name"
  },
  "bugs": {
    "url": "https://github.com/kaze-no-manga/package-name/issues"
  },
  "homepage": "https://github.com/kaze-no-manga/package-name#readme"
}
```

### Optional Fields

```json
{
  "engines": {
    "node": ">=20.0.0"
  },
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

## Versioning Strategy

### Semantic Versioning (semver)

Format: `MAJOR.MINOR.PATCH`

- **MAJOR** (1.0.0): Breaking changes (incompatible API changes)
- **MINOR** (0.1.0): New features (backward compatible)
- **PATCH** (0.0.1): Bug fixes (backward compatible)

### Pre-1.0 Versions

During MVP development, use `0.x.y`:
- `0.x.y` - API is not stable yet
- Breaking changes allowed in MINOR version
- Once stable, bump to `1.0.0`

### Version Bumping

```bash
# Patch (0.1.0 → 0.1.1)
npm version patch

# Minor (0.1.1 → 0.2.0)
npm version minor

# Major (0.2.0 → 1.0.0)
npm version major

# Pre-release (0.1.0 → 0.1.1-beta.0)
npm version prerelease --preid=beta
```

This automatically:
1. Updates `package.json` version
2. Creates a git commit
3. Creates a git tag

### Version Tags

- `latest` - Stable release (default)
- `beta` - Beta releases
- `alpha` - Alpha releases
- `next` - Upcoming features

```bash
# Publish with tag
npm publish --tag beta
```

## Publishing Workflow

### Manual Publishing

```bash
# 1. Ensure clean working directory
git status

# 2. Run tests
npm test

# 3. Build package
npm run build

# 4. Bump version
npm version patch  # or minor/major

# 5. Publish to NPM
npm publish --access public

# 6. Push changes and tags
git push && git push --tags
```

### Automated Publishing (GitHub Actions)

```yaml
# .github/workflows/publish.yml
name: Publish to NPM

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
      
      - run: npm ci
      - run: npm test
      - run: npm run build
      
      - run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**Trigger:**
```bash
# Create and push tag
npm version patch
git push && git push --tags
```

## Dependency Management

### Dependency Types

**dependencies**: Required at runtime
```json
{
  "dependencies": {
    "zod": "^3.22.0"
  }
}
```

**devDependencies**: Required for development only
```json
{
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  }
}
```

**peerDependencies**: Required by consumer (not bundled)
```json
{
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0"
  }
}
```

### Version Ranges

- `^1.2.3` - Compatible with 1.x.x (recommended)
- `~1.2.3` - Compatible with 1.2.x
- `1.2.3` - Exact version (avoid unless necessary)
- `>=1.2.3 <2.0.0` - Range

### Updating Dependencies

```bash
# Check for outdated packages
npm outdated

# Update all dependencies (respecting semver)
npm update

# Update to latest (ignoring semver)
npm install package@latest
```

## Package Documentation

### README.md Template

```markdown
# @kaze-no-manga/package-name

> Brief description

## Installation

\`\`\`bash
npm install @kaze-no-manga/package-name
\`\`\`

## Usage

\`\`\`typescript
import { something } from '@kaze-no-manga/package-name'

// Example usage
\`\`\`

## API

### `functionName(param: Type): ReturnType`

Description of function.

**Parameters:**
- `param` - Description

**Returns:**
- Description of return value

**Example:**
\`\`\`typescript
const result = functionName('value')
\`\`\`

## License

MIT © Kaze no Manga Team
```

### Changelog

Keep a `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/):

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

## [0.2.0] - 2024-01-15

### Added
- New feature X

### Changed
- Updated API for Y

### Fixed
- Bug in Z

## [0.1.0] - 2024-01-01

### Added
- Initial release
```

## Cross-Package Dependencies

### Dependency Graph

```
@kaze-no-manga/brand (no dependencies)
  ↓
@kaze-no-manga/models (depends on brand)
  ↓
@kaze-no-manga/scraper (depends on models)
```

### Version Pinning

For internal packages, use exact versions:

```json
{
  "dependencies": {
    "@kaze-no-manga/models": "0.1.0"
  }
}
```

**Why?** Ensures consistency across the organization.

### Updating Internal Dependencies

When publishing a new version of a dependency:

1. Publish the dependency (e.g., `models@0.2.0`)
2. Update dependents (e.g., `scraper`)
3. Test dependents
4. Publish dependents

## NPM Scripts

### Standard Scripts

Every package SHOULD have these scripts:

```json
{
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "biome check .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit",
    "prepublishOnly": "npm run build && npm test"
  }
}
```

### Custom Scripts

Add package-specific scripts as needed:

```json
{
  "scripts": {
    "codegen": "graphql-codegen",
    "migrate": "drizzle-kit migrate"
  }
}
```

## TypeScript Configuration

### Base Config

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "moduleResolution": "bundler",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Extending Base Config

```json
{
  "extends": "@kaze-no-manga/tsconfig/base.json",
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

## .npmignore

Exclude files from NPM package:

```
# Source files
src/
tests/
*.test.ts

# Config files
tsconfig.json
vitest.config.ts
.eslintrc.js

# Development files
.git/
.github/
.vscode/
node_modules/

# Documentation (keep README.md)
docs/

# Build artifacts (keep dist/)
*.log
.DS_Store
```

## Package Size Optimization

### Check Package Size

```bash
# Before publishing
npm pack --dry-run

# Analyze bundle
npx bundlephobia @kaze-no-manga/package-name
```

### Reduce Size

1. **Tree-shaking**: Use ES modules
2. **Exclude dev files**: Use `.npmignore`
3. **Minimize dependencies**: Only include what's needed
4. **Avoid bundling**: Let consumers bundle

### Size Targets

- `brand`: < 50 KB
- `models`: < 100 KB
- `scraper`: < 200 KB

## Security

### Audit Dependencies

```bash
# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix
```

### Publish Verification

Before publishing:
1. ✅ All tests pass
2. ✅ No security vulnerabilities
3. ✅ Version bumped correctly
4. ✅ Changelog updated
5. ✅ README up to date

### NPM Token Security

**Never commit NPM tokens!**

Store in GitHub Secrets:
- `NPM_TOKEN` - For automated publishing

## Troubleshooting

### Package Not Found

```bash
# Check if package exists
npm view @kaze-no-manga/package-name

# Check if you're logged in
npm whoami

# Login to NPM
npm login
```

### Version Conflict

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Publish Failed

```bash
# Check if version already exists
npm view @kaze-no-manga/package-name versions

# Bump version and try again
npm version patch
npm publish --access public
```

## Best Practices

1. **Semantic Versioning**: Follow semver strictly
2. **Changelog**: Document all changes
3. **Tests**: 100% test coverage before publishing
4. **Documentation**: Keep README up to date
5. **Small Packages**: Single responsibility per package
6. **Minimal Dependencies**: Only include what's necessary
7. **Type Definitions**: Always include `.d.ts` files
8. **Backward Compatibility**: Avoid breaking changes when possible
9. **Deprecation**: Warn before removing features
10. **Security**: Audit dependencies regularly

## Checklist for New Package

- [ ] Create repository
- [ ] Initialize npm package (`npm init`)
- [ ] Configure `package.json` (name, version, exports)
- [ ] Setup TypeScript (`tsconfig.json`)
- [ ] Add build script (`npm run build`)
- [ ] Add tests (`npm test`)
- [ ] Write README.md
- [ ] Add LICENSE (MIT)
- [ ] Configure `.npmignore`
- [ ] Test locally (`npm link`)
- [ ] Publish to NPM (`npm publish --access public`)
- [ ] Create GitHub release
- [ ] Update dependent packages

## Resources

- [NPM Documentation](https://docs.npmjs.com/)
- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
