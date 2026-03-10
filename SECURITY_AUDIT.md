# Security Audit Report

This document outlines the findings of the security audit performed on the `macro-keyboards` repository, focusing on the pre-compiled CLI binaries and automated security measures.

## 1. Automated Security Scanning

A GitHub Actions workflow has been implemented to perform continuous security audits using **Trivy**.

- **Workflow File**: `.github/workflows/trivy-scan.yml`
- **Scans Performed**:
  - **Filesystem Scan (`fs`)**: Detects vulnerabilities (CVEs) in project dependencies and OS packages.
  - **Configuration Scan (`config`)**: Identifies security misconfigurations in the repository's setup files.
- **Triggers**: Every push to `master`, pull requests, and a weekly scheduled run.

## 2. Binary Auditing - Sayo_CLI

The repository includes pre-compiled binaries for multiple platforms to configure the macro keyboards.

### 2.1 Binary Hashes (Verified)

| Platform | Binary Name | SHA-256 Hash |
| :--- | :--- | :--- |
| **Linux** | `Sayo_CLI_Linux` | `89452654e31239a47e57e20a0f3480cfcebfd872fe2e23064a8fb8b14a8c71e8` |
| **Windows** | `Sayo_CLI_Windows.exe` | `d3235e10100ea9bfd7f0ed1a19782020289afaa9089e89a8a802a6ef4e0725b7` |
| **macOS (M1)** | `Sayo_CLI_Mac_M1` | `027bfa24c6e1df3f767db9aac6b4bb6fdf160668f6aac4c5c1a583650b8d427d` |
| **macOS (X86)** | `Sayo_CLI_Mac_X86` | `f4bc56f2327514d9f68e3b39420816075d2a627e31e60f720b98f3b20383e3ee` |

### 2.2 Functional Analysis

Through static analysis (strings, shared libraries, symbols), the following behavior was observed across all platforms:

- **Local HTTP Server**: The binaries embed a lightweight web server (**Sayobot_HTTP/0.0.5**) that listens on `127.0.0.1`. This server facilitates communication between the local web-based UI (`html/index.html`) and the hardware device.
- **Hardware Interaction**:
  - On **Linux**, it uses `libhidapi-hidraw`.
  - On **macOS**, it utilizes the `IOKit` and `IOHIDManager` frameworks.
- **Protocol**: It implements an internal `O2Protocol` for device interaction.
- **Dependencies**: Uses `jsoncpp` for parsing configuration and `pthread` for multi-threading.

### 2.3 Auditing Techniques Used

#### Static Analysis
- **Strings**: Used to identify embedded URLs, error messages, and internal namespaces (`Sayo_control_CLI`, `Sayobot_HTTP`).
- **Library Dependencies**: Identified via `ldd` (Linux) and Mach-O header analysis (macOS).
- **Symbol Analysis**: Examined exported symbols and internal class names via `nm` and `strings`.

#### Reconnaissance Findings
- **Network behavior**: The binaries listen on a local port (default is often 8000 or similar, configurable via `-p`). No evidence of external telemetry or phone-home behavior was found.
- **Permissions**: Binaries require elevated privileges (`sudo` on Linux/macOS) to access raw HID devices.

## 3. Recommendations for Users

1. **Verify Hashes**: Always verify the checksum of the binaries against the ones documented here.
2. **Local Firewall**: Ensure that your firewall does not allow external connections to the ports opened by the CLI tool.
3. **Run as Needed**: Only run the CLI tool when configuring the device and terminate it afterwards.
