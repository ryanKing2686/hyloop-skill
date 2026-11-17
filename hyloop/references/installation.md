# Installing HyLoop for a local Mac assistant

Core connection needs macOS 14+, the native `scripts/hyloop` binary for the Mac architecture, and a host able to execute it **on the Mac**, not only inside a cloud/Linux sandbox. Do not install models for pairing.

Use the host's native skill installer/importer when available. Install the entire `hyloop/` folder, not just SKILL.md. Resolve the absolute installed root and invoke its `scripts/hyloop` directly; adding it to PATH or copying it into `~/.local/bin` is unnecessary.

- **Codex:** use its provided Skill installer and the configured skill root. The existing local integration uses `$CODEX_HOME/skills/hyloop` (default `~/.codex/skills/hyloop`); respect a host that reports a different supported root. Do not install duplicate copies into several roots.
- **Claude Code:** personal filesystem skills use `~/.claude/skills/hyloop`; a project-scoped install can use `.claude/skills/hyloop` when that scope is requested.
- **Claude Desktop / Cowork:** use that application's native Skill import/install flow. A Claude Code filesystem directory is not proof the Desktop host loaded the Skill. Confirm both Skill discovery and the desktop bridge's ability to execute the native connector on the Mac. Uploading a Skill ZIP to a cloud runtime alone cannot reach the phone.
- **Other local hosts:** use the host's documented Skill location/import mechanism; do not guess a home-directory location or request broad home access when a specific destination suffices.

Inspect the selected package and its manifest before running it. A package checksum detects mismatch; it does not certify the publisher or make a prebuilt binary fully auditable. Do not suppress macOS security controls. Existing local modifications or unknown files must not be overwritten indiscriminately; use the native updater or stage and compare the selected files. Preserve Keychain and private memory outside the skill package.

The bundled connector is ad-hoc signed. Replacing an older executable may cause macOS to deny access to its saved pairing even though the entry and data format are retained. An isolated old-to-new executable check reproduced this; do not promise a seamless update to an already paired Mac. Keep a working installation until the user is ready to update. If the updated tool reports a Keychain error, follow the existing explicit reconnect recovery in SKILL.md; never delete the old pairing automatically.

After installation run, with the actual installed absolute path:

```sh
"/ABSOLUTE/INSTALLED/hyloop/scripts/hyloop" --version
"/ABSOLUTE/INSTALLED/hyloop/scripts/hyloop" doctor
```

Verify connector 0.2.2+, `stagedConnection: true`, `pairingProtocol: 3` and `phoneApprovalPairing: true` for the new flow and refresh the host if Skill discovery requires it. A missing binary means a source checkout or incomplete package: obtain the matching release package or build the connector from the HyLoop source with Swift 6; never claim that merely copying Markdown installed the tool. Core `doctor` does not read Keychain or phone data and does not prove permission to discover the LAN.

Then follow **Install and connect an iPhone** in SKILL.md. Do not register MCP or launch digital-self analysis merely because the user requested installation and pairing. Report the installation path, tool readiness, and actual phone connection result separately.

References: [Claude Code filesystem skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), [Claude custom Skill import](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills).
