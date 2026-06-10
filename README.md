# AntiSPectator
A hardened, User-Mode Host Intrusion Prevention System (HIPS) for Windows.
AntiSPectator is designed to proactively block Remote Access Trojans (RATs), malicious remote takeovers, and unauthorized system modifications (such as those seen in Scammer Payback scenarios or silent LOLBIN executions).
Traditional antivirus solutions often fail to detect these threats because attackers abuse legitimate administrative tools (like AnyDesk, TeamViewer, PowerShell, or WMI). AntiSPectator solves this by establishing a zero-trust environment around critical OS functions using native Windows security mitigations.
## Architecture Overview
The project follows a strict separation of privileges and relies on a "Fail-Secure" design. It consists of two main components communicating via a hardened IPC channel.
### 1. The Core Engine (C++ Windows Service)
 * **Privilege Level:** Runs as NT AUTHORITY\SYSTEM.
 * **Design Choice:** 100% User-Mode. We avoid Kernel-mode drivers to maintain compatibility with Strict Secure Boot and avoid BSOD risks, relying instead on heavy user-mode hardening.
 * **Process Monitoring (ETW):** Uses Windows Event Tracing (Microsoft-Windows-Kernel-Process) via KrabsETW to monitor process creation, verify digital signatures (WinVerifyTrust), and block malicious arguments for LOLBINs (e.g., cmd.exe, powershell.exe).
 * **Anti-Tamper & Self-Defense:** The service protects itself by applying explicit deny DACLs (SDDL) to its process handle, neutralizing SeDebugPrivilege abuse by local administrators. Config states are stored in an SQLite database encrypted with SQLCipher. The encryption key is derived from a hardware ID (SMBIOS, CPU ID) and a DPAPI-protected salt.
### 2. The Dashboard (Python & PyQt6)
 * **Privilege Level:** Runs as an elevated Administrator.
 * **Role:** A clean, decoupled UI that sends state changes (e.g., "Mute Microphone", "Kill Network", "Lockdown Files") to the Core Service.
 * **Security:** The Python codebase contains zero hardcoded cryptographic keys or critical enforcement logic. It acts purely as a client.
### 3. Hardened IPC (Named Pipes)
To prevent TOCTOU (Time-of-Check to Time-of-Use) attacks and pipe squatting, the communication between the UI and the Service uses a strict Challenge-Response protocol:
 * The Named Pipe is created with FILE_FLAG_FIRST_PIPE_INSTANCE.
 * The Service authenticates the UI via ImpersonateNamedPipeClient, validates the token SID, and holds a PROCESS_QUERY_LIMITED_INFORMATION handle to prevent PID reuse during signature verification.
 * A CNG-based HMAC-SHA256 handshake ensures the client is authorized before accepting any payload.
## Current Status: Bootstrapping
We are currently in the **MVP bootstrapping phase**. The architecture has been fully designed and audited, and we are now looking for contributors to help write the initial C++ and Python implementations.
### Roadmap for v0.1 (MVP)
 * [ ] Implement C++ Windows Service skeleton and basic SetupAPI device control (Network kill-switch).
 * [ ] Implement the Named Pipe server with Challenge-Response handshake.
 * [ ] Build the basic Python/PyQt6 dashboard and IPC client.
 * [ ] Integrate KrabsETW for process creation blocking (Targeting cmd.exe / powershell.exe execution).
## Contributing
We are actively looking for developers to join the project. Whether you want to contribute code, audit the architecture, or test the builds, you are welcome.
**We specifically need help with:**
 * **C++ / Windows Internals:** Setting up the ETW listeners, writing the DPAPI/SQLCipher wrappers, and handling the low-level WinAPI calls for process DACL modification.
 * **Python:** Building the PyQt6 UI, handling asynchronous IPC communication with the named pipe, and packaging the app.
### How to get involved
 1. Check the Issues tab for tasks labeled help wanted or good first issue.
 2. Join our Discord server: **[Insert Discord Link Here]** to discuss architecture and coordinate development.
 3. Fork the repo, create a feature branch, and submit a PR.
## License
This project is licensed under the GNU General Public License v3.0 (GPLv3) - see the LICENSE file for details.
