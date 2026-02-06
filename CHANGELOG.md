# Changelog

## [1.0.1] - 2026-02-06

### 🔒 Security Fixes
- **CRITICAL**: Fixed command injection vulnerability in callback handlers
  - Removed dangerous `exec({ command: \`rm ${file}\` })` pattern
  - Replaced with safe Node.js `fs.promises.unlink()` API
  - Added path validation to prevent directory traversal attacks
  - Added comprehensive security best practices documentation

### 📚 Documentation
- Added `references/security-best-practices.md` with detailed security guidelines
- Updated `SKILL.md` with security warnings and safe code examples
- Updated `references/gateway-integration.md` with secure callback handling patterns
- Added security comments to `examples/confirmation-card.json`

### ✅ Security Checklist
- [x] Input validation for all user-controlled data
- [x] No direct shell command execution with user input
- [x] Path normalization and workspace boundary checks
- [x] Error handling without information leakage
- [x] Security documentation and examples

## [1.0.0] - 2026-02-05

### 🎉 Initial Release
- Interactive card creation and sending
- Long-polling callback server
- Gateway integration
- Multiple card templates (confirmation, todo, poll, form)
- Comprehensive documentation
