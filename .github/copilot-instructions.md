# Strails API Documentation Repository

Strails is a GitBook-based documentation repository for Nigerian banking integration API documentation. This repository contains API documentation and a comprehensive Postman collection for testing wallet funding scenarios.

**ALWAYS** reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Repository Structure and Purpose
- **Documentation Site**: This is a GitBook documentation repository, NOT a traditional software project
- **No Build Process**: GitBook handles documentation rendering - no local build required
- **Primary Content**: API documentation for Strails (Nigerian banking integration)
- **Testing Tools**: Comprehensive Postman collection with 1781 lines of webhook test scenarios

### Essential Commands (All Fast - Under 1 Second)
- **View repository structure**: `ls -la` 
- **Read documentation**: `cat README.md` and `cat SUMMARY.md`
- **List markdown files**: `find . -name "*.md" -exec basename {} \;`
- **File statistics**: `wc -l *.md *.json`

### Repository Contents Overview
```bash
# Repository root contents (verified working):
ls -la
# Output:
# README.md (26 lines) - Main GitBook documentation
# SUMMARY.md (16 lines) - GitBook table of contents  
# postman-collection.json (1781 lines) - Webhook testing collection
# .keep (1 line) - Git placeholder file
```

## Testing with Postman Collection

### Prerequisites
- Newman CLI is available: `newman --version` (tested: v6.2.1)
- Postman collection exists but has formatting issue (starts with `@copilot` comment line)

### Running Postman Tests
**CRITICAL**: The postman-collection.json file has a malformed first line that must be removed before use:

```bash
# View the malformed first line:
head -1 postman-collection.json
# Output: @copilot

# ALWAYS use this command to run the collection (creates temporary file):
sed '1d' postman-collection.json > /tmp/clean-collection.json && newman run /tmp/clean-collection.json --bail --timeout 10000

# To validate collection structure (takes ~0.5 seconds):
sed '1d' postman-collection.json | python -m json.tool > /dev/null && echo "Collection structure valid"
```

### Expected Test Behavior
- **Local Server Required**: Tests expect Firebase Functions emulator at `http://localhost:5001/demo-cngn-dex-local/us-central1`
- **Without Local Server**: Tests will fail with connection errors (expected behavior)
- **Test Categories**: Legacy format tests and new virtual account funding format tests
- **Authentication**: Optional HMAC signature generation for testing scenarios

## Validation and Quality Assurance

### Documentation Changes
- **ALWAYS** validate markdown syntax after editing documentation files
- **Check file line counts** after changes: `wc -l *.md *.json`
- **Verify GitBook structure** by ensuring README.md and SUMMARY.md are properly formatted
- **Test content** by reading both documentation files: `cat README.md SUMMARY.md`

### Postman Collection Changes  
- **NEVER** remove the `@copilot` first line - it's part of the repository structure
- **Test collection parsing**: `sed '1d' postman-collection.json | newman run - --list`
- **Validate JSON structure** after editing (excluding first line): `sed '1d' postman-collection.json | python -m json.tool > /dev/null`

### Content Validation Scenarios
**CRITICAL**: After making any documentation changes, ALWAYS perform these validation steps:

1. **Documentation Consistency Check**:
   ```bash
   # Verify both main documentation files are readable
   cat README.md SUMMARY.md > /dev/null && echo "Documentation files valid"
   ```

2. **Postman Collection Integrity Check**:
   ```bash
   # Test collection structure is valid JSON (takes ~0.5 seconds)
   sed '1d' postman-collection.json | python -m json.tool > /dev/null && echo "Collection valid"
   ```

3. **Repository Structure Verification**:
   ```bash
   # Ensure all expected files exist
   test -f README.md && test -f SUMMARY.md && test -f postman-collection.json && echo "All files present"
   ```

## GitBook Platform Integration

### Local Development Limitations
- **NO LOCAL PREVIEW**: GitBook CLI has compatibility issues with Node.js v20.19.4
- **Deployment Method**: Changes are reflected on GitBook platform when repository is synced
- **File Format**: Standard GitBook markdown with YAML frontmatter
- **Navigation**: SUMMARY.md defines the documentation structure

### GitBook CLI Issues (Documented Problems)
```bash
# These commands WILL FAIL due to Node.js compatibility:
gitbook init    # TypeError: cb.apply is not a function
npx gitbook     # Same compatibility error
npm install gitbook-cli  # Installation succeeds but runtime fails
```

**Alternative**: Use the GitBook web platform for preview and publishing instead of local CLI.

## Common Tasks and File Locations

### Frequently Used File Paths
- **Main Documentation**: `/README.md` (GitBook homepage)
- **Table of Contents**: `/SUMMARY.md` (GitBook navigation)
- **API Testing**: `/postman-collection.json` (webhook test scenarios)
- **Git Configuration**: `/.git/config` (repository settings)

### Common Development Workflows

#### Editing Documentation
1. **Edit content**: Modify `README.md` or `SUMMARY.md` directly
2. **Validate syntax**: Check markdown is well-formed
3. **Test readability**: `cat README.md SUMMARY.md`
4. **Commit changes**: Use standard git workflow

#### Testing API Changes
1. **Review collection**: `sed '1d' postman-collection.json | newman run - --list`
2. **Validate JSON**: `sed '1d' postman-collection.json | python -m json.tool > /dev/null`
3. **Test scenarios**: Requires local Firebase Functions emulator setup (outside this repository)

#### Repository Maintenance
1. **Check status**: `git status`
2. **View structure**: `ls -la && wc -l *.md *.json`
3. **Verify integrity**: Run all validation scenarios above

## Key Projects and Focus Areas

### Documentation Content
- **Strails API**: Nigerian banking integration documentation

- **Smart Wallet Funding**: Legacy and new format payload scenarios

### Testing Infrastructure  
- **Postman Collection**: Comprehensive webhook testing scenarios
- **HMAC Signatures**: Optional security testing with configurable secrets
- **Firebase Integration**: Local emulator testing setup references

### Repository Characteristics
- **Minimal Dependencies**: No package.json or build tools required
- **GitBook Standard**: Follows GitBook documentation conventions
- **API-Focused**: Designed for API documentation and testing workflows
- **Banking Domain**: Specialized for Nigerian financial services integration

**REMEMBER**: This is a documentation repository, not a software project. Focus on content quality, API accuracy, and testing scenario coverage rather than traditional software development practices.