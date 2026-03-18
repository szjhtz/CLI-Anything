# Security Policy

## Threat Model

CLI-Anything bridges AI agents and real desktop software. Unlike traditional
CLIs where a human reviews every command, **an AI agent may autonomously
construct and execute commands** based on untrusted input (user prompts,
uploaded files, or other agents' output). This makes input validation
critical — a prompt-injected agent could pass crafted arguments to software
backends (melt, GIMP Script-Fu, LibreOffice, etc.).

Key attack surfaces:

| Surface | Risk | Mitigation |
|---------|------|------------|
| Subprocess arguments | Malicious codec names, paths, or filter params passed to melt/ffmpeg/gimp | Allowlist validation before subprocess calls |
| Script-Fu injection | User-controlled strings embedded in GIMP batch scripts | `_script_fu_escape()` in gimp_backend.py |
| XML/SVG content | User text injected into MLT XML, SVG, or Draw.io files | `xml_escape()` / ElementTree auto-escaping |
| File path traversal | Agent-controlled output paths writing to arbitrary locations | `os.path.abspath()` normalization |
| Credential exposure | API keys stored in plaintext config files | File permissions set to `0o600` |

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly:

1. **Do NOT open a public GitHub issue.**
2. Email the maintainers or use GitHub's private vulnerability reporting:
   **Security** tab > **Report a vulnerability**.
3. Include: affected file(s), reproduction steps, and potential impact.
4. We aim to acknowledge reports within 48 hours and release a fix within
   7 days for critical issues.

## Security Guidelines for Harness Developers

When building a new CLI harness, follow these rules:

1. **Never use `shell=True`** in `subprocess.run()` — always pass arguments
   as a list.
2. **Validate all subprocess arguments** against an allowlist. Do not pass
   user-controlled strings directly to external tools.
3. **Escape user content** before embedding it in scripts (Script-Fu, Python,
   Lua) or structured formats (XML, SVG, HTML).
4. **Use `os.path.abspath()`** on all file paths and consider resolving
   symlinks with `os.path.realpath()` for sensitive write operations.
5. **Never log or echo API keys** in error messages or JSON output.
