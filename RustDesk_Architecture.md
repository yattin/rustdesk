# RustDesk Global Architecture

## 1. Overview

RustDesk is an open-source remote desktop software written primarily in Rust. It provides a secure and configurable way to access and control devices remotely.RustDesk supports multiple platforms and offers features like self-hosting, P2P connections, and end-to-end encryption.

## 2. Key Components

The main architectural components of RustDesk are:

*   **Core Logic (Rust Backend):** The heart of the application, handling all core functionalities like connection management, data streaming, security, and platform interactions.
*   **User Interface (UI):** Built with Flutter, providing a cross-platform graphical interface for both desktop and mobile applications.
*   **Rendezvous Server (`hbbs`):** A publicly accessible or self-hosted server that facilitates peer discovery and connection initiation between clients. It helps clients find each other and punch through NATs.
*   **Relay Server (`hbbr`):** Used when a direct P2P connection cannot be established. It relays encrypted traffic between the controlling and controlled devices.
*   **Platform Abstraction Layer:** A set of modules within the Rust core that abstract OS-specific functionalities, allowing the core logic to remain platform-agnostic to a large extent.
*   **HTTP API Server (Part of `hbbs`):** Provides an HTTP API for user account management, device synchronization, and remote configuration when using the commercial or self-hosted Pro versions.

## 3. Core Logic (Rust Backend)

The Rust backend is responsible for the heavy lifting of remote desktop operations.

*   **Modularity:** The backend is organized into several libraries (crates) to promote code reuse and separation of concerns:
    *   `hbb_common`: Contains common utilities, data structures (protobuf definitions), configuration management, TCP/UDP wrappers, file system functions for file transfer, and security primitives.
    *   `scrap`: Handles screen capture across different platforms.
    *   `enigo`: Provides platform-specific keyboard and mouse control.
    *   `clipboard`: Implements file copy and paste functionality for Windows, Linux, and macOS.
    *   Other smaller crates like `virtual_display` and `remote_printer` handle specific functionalities.

*   **Main Application Entry Points and Process Roles:** The `rustdesk` executable can operate in several modes, determined by command-line arguments:
    *   **Default (Client/Main UI):** Launches the Flutter UI, allowing the user to initiate connections or accept incoming connections. It also starts a background server component.
    *   **Server (`--server`):** Runs as a dedicated server process, typically managed by the OS service. This process handles incoming connection requests, manages active sessions, and provides services like video, audio, input, and clipboard sharing.
    *   **Service (`--service`):** Manages the main server process, ensuring it runs reliably in the background. This is crucial for unattended access. On Linux, this also handles X11/Wayland session management.
    *   **Command-Line Interface (CLI) (various flags like `--connect`, `--port-forward`):** Allows for scripting and headless operations like initiating connections or setting up port forwarding.
    *   The `src/main.rs` and `src/core_main.rs` files handle parsing these arguments and dispatching to the appropriate logic. `core_main.rs` is a shared entry point for both Sciter (deprecated) and Flutter UIs, handling common initialization and argument parsing before launching the UI.

*   **Server-side (`src/server.rs`):**
    *   Manages active connections from clients.
    *   Provides various services to connected clients:
        *   **Video Service:** Captures the screen (using `scrap`) and streams it to the client.
        *   **Audio Service:** Captures and streams audio.
        *   **Input Service:** Receives input events (mouse, keyboard) from the client and injects them into the local system (using `enigo`).
        *   **Clipboard Service:** Synchronizes clipboard content between the client and server.
        *   **File Transfer Service:** Facilitates file transfer operations.
    *   Handles session lifecycle and security.

*   **Client-side (`src/client.rs`):**
    *   Initiates connections to remote peers via the rendezvous server or directly.
    *   Manages the connection lifecycle (P2P, relay).
    *   Receives and renders video/audio streams from the server.
    *   Captures local input events and sends them to the server.
    *   Handles clipboard synchronization.

## 4. User Interface (Flutter)

The Flutter framework is used for the client-facing UI on both desktop and mobile platforms.

*   **Structure:**
    *   **Main App Window:** The primary interface for users to manage their connections, settings, and view their ID for incoming connections. (`flutter/lib/main.dart` initializes this).
    *   **Multi-Window Support:** For active remote sessions (remote desktop, file transfer, port forwarding, view camera), RustDesk utilizes Flutter's `desktop_multi_window` plugin to create separate windows for each session. This allows for managing multiple remote sessions concurrently.
    *   Different window types (`WindowType.RemoteDesktop`, `WindowType.FileTransfer`, etc.) are defined to handle specific session UIs.

*   **Communication with Rust Backend:**
    *   The Flutter UI communicates with the Rust core logic via **Foreign Function Interface (FFI)**.
    *   `flutter_rust_bridge` is used to generate the boilerplate code for FFI, simplifying the interaction between Dart (Flutter) and Rust.
    *   The `flutter/lib/models/platform_model.dart` and the generated `bridge_generated.dart` (implied) are key for this communication, allowing Flutter to call Rust functions and receive events/data from the Rust backend.

*   **State Management:** While not explicitly detailed in the provided files, Flutter applications typically use state management solutions like Provider, BLoC, GetX, or Riverpod. `flutter/lib/common.dart` and individual page/widget states likely manage UI state, peer lists, connection status, etc. `GetX` (`get/get.dart`) is visible in `flutter/lib/main.dart`, indicating its use for routing and potentially state management.

## 5. Platform Abstraction

RustDesk is designed to be cross-platform, and this is achieved through a platform abstraction layer.

*   **`src/platform/` Directory:** This directory contains OS-specific modules (e.g., `linux.rs`, `windows.rs`, `macos.rs`). Each module implements platform-dependent functionalities required by the core logic.
*   **Functionalities Handled:**
    *   **Display Management:** Enumerating displays, capturing screen content (interfacing with `scrap`'s platform specifics), handling display resolutions.
    *   **Input Handling:** Capturing and injecting keyboard/mouse events (interfacing with `enigo`'s platform specifics).
    *   **Service Management:** Installing, starting, stopping, and uninstalling the RustDesk background service on different operating systems.
    *   **Clipboard Integration:** Accessing and manipulating the system clipboard.
    *   **Audio Input/Output:** Interfacing with system audio APIs.
    *   **Permissions:** Handling OS-specific permission requests (e.g., screen recording, accessibility).
    *   **Network Configuration:** Retrieving network interface information, NAT type detection.
    *   **WakeLock:** `src/platform/mod.rs` defines a `WakeLock` structure to prevent the system from sleeping during active sessions, with platform-specific implementations.
    *   `src/platform/linux.rs` shows detailed handling for X11 and Wayland display servers, PulseAudio, and systemd service management.

## 6. Networking and Communication

RustDesk employs a sophisticated networking model to establish connections between peers.

*   **Rendezvous Mechanism (`src/rendezvous_mediator.rs`):**
    *   Clients register their ID and network information (public IP, NAT type) with a rendezvous server (`hbbs`).
    *   When a client wants to connect to a peer, it queries the rendezvous server for the peer's information.
    *   The rendezvous server facilitates the initial handshake and helps in NAT traversal.
    *   RustDesk can use public rendezvous servers or a self-hosted `hbbs` instance.
    *   The `RendezvousMediator` in Rust handles communication with the rendezvous server, registration, and peer discovery.

*   **Connection Establishment:** RustDesk attempts connections in the following order:
    1.  **Direct LAN Connections:** If both peers are on the same local network, a direct TCP connection is attempted.
    2.  **P2P (UDP/TCP Hole Punching):** If peers are behind NATs, the rendezvous server assists in performing UDP hole punching to establish a direct P2P connection. TCP hole punching might also be attempted.
    3.  **Relay Connections:** If direct P2P connection fails (e.g., due to symmetric NATs), the connection is relayed through an `hbbr` server. The traffic is still end-to-end encrypted.

*   **Security:**
    *   **Public Key Registration:** Clients register their public keys with the rendezvous server. This helps in authenticating peers.
    *   **Session Encryption:** End-to-end encryption is used for all session data. NaCl (Networking and Cryptography library) `box` (asymmetric encryption) and `sign` (digital signatures) primitives are likely used, as suggested by `sodiumoxide` in `Cargo.toml`. The initial handshake involves exchanging keys to establish a secure channel.
    *   Passwords and tokens are used for authentication.

*   **Protocols:**
    *   **Protobuf:** Used for defining the structure of messages exchanged with the rendezvous server and during session setup (e.g., `RendezvousMessage`, `VideoFrame`).
    *   **UDP:** Primarily used for P2P hole punching and potentially for streaming media data in P2P sessions.
    *   **TCP:** Used for direct LAN connections, relay connections, and communication with the rendezvous server if UDP fails or if WebSockets are used.
    *   **WebSockets (WS):** Can be used as a transport for rendezvous and relay communication, especially useful for bypassing firewalls or when running `hbbs`/`hbbr` behind a reverse proxy like Nginx. The `use_ws()` function in `src/rendezvous_mediator.rs` indicates this capability.

*   **HTTP API Communication (`src/hbbs_http.rs`):**
    *   This component handles communication with the `hbbs` HTTP API server.
    *   It's used for features like user authentication, managing device lists, synchronizing configurations, and potentially for license management in the Pro version.
    *   It uses the `reqwest` crate for making HTTP requests.

## 7. Build and Packaging

*   **Cargo Workspaces:** The `Cargo.toml` defines a workspace with members like `libs/scrap`, `libs/hbb_common`, etc. This helps in managing multiple related Rust crates within the same project.
*   **Platform-Specific Build Tooling:** While not explicitly detailed in the provided files, the presence of platform-specific code and dependencies (e.g., `winapi` for Windows, X11/Wayland libs for Linux) implies that the build process handles these differences. Flutter also has its own build system for packaging the UI for various platforms.
*   **Features Flags:** `Cargo.toml` uses feature flags (e.g., `flutter`, `hwcodec`, `wayland`) to enable or disable specific functionalities during compilation, allowing for tailored builds.

## 8. Key Differentiators

*   **Self-Hosting:** The architecture clearly supports self-hosting of rendezvous (`hbbs`) and relay (`hbbr`) servers, giving users full control over their data and infrastructure.
*   **Cross-Platform:** The combination of a Rust core with a platform abstraction layer and a Flutter UI makes RustDesk inherently cross-platform.
*   **Security Focus:** End-to-end encryption and public key cryptography are integral to the design.
*   **Modularity:** The Rust backend's library-based structure allows for better maintainability and potential reuse of components.
