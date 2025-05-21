# RustDesk Command-Line Interface

This document provides a comprehensive overview of the command-line arguments available in RustDesk, based on analysis of `src/main.rs` and `src/core_main.rs`. These arguments allow for a wide range of operations, from initiating connections and managing services to configuring the application.

## I. General Application Arguments

These arguments are typically handled by `src/core_main.rs` and affect the overall application behavior or provide information.

*   **`--version`**
    *   **Purpose:** Displays the current installed version of RustDesk.
    *   **Value Format:** None (flag).
    *   **Notes:** The application will print the version and then exit.
    *   **Example:** `rustdesk --version`

*   **`--build-date`**
    *   **Purpose:** Displays the build date of the RustDesk executable.
    *   **Value Format:** None (flag).
    *   **Notes:** The application will print the build date and then exit.
    *   **Example:** `rustdesk --build-date`

*   **`<URI_SCHEME>` (e.g., `rustdesk://connect/...`)**
    *   **Purpose:** Handles custom URI schemes (e.g., `rustdesk://connect/...`) to launch RustDesk sessions or perform actions. This is often used for deep linking.
    *   **Value Format:** A URI string (e.g., `rustdesk://connect/123456789?password=abc`).
    *   **Notes:** When a URI is passed, RustDesk attempts to send it to an already running main RustDesk instance. If successful, the current process usually exits. Otherwise, it might proceed to start a new GUI instance to handle the URI.
    *   **Windows Implementation Details:**
        *   The `core_main_invoke_new_connection` function (in `src/core_main.rs`) parses the command line for session arguments.
        *   It constructs a URI string.
        *   `platform::send_message_to_hnwd` (in `src/platform/windows.rs`) is called.
        *   This function uses `FindWindowW` to locate the main Flutter window (class `FLUTTER_RUNNER_WIN32_WINDOW_CLASS`) and `SendMessageW` with `WM_COPYDATA` to pass the URI to the running instance. The `COPYDATASTRUCT` structure is used for this inter-process communication.
    *   **Example:** `rustdesk rustdesk://connect/123456789`

## II. Connection and Session Management Arguments

These arguments are used for initiating or managing remote sessions.

*   **`-c, --connect=[REMOTE_ID]` (CLI Feature)**
    *   **Purpose:** Initiates a "test" connection to a remote RustDesk peer. Intended for testing purposes.
    *   **Value Format:** `REMOTE_ID` (string) - The ID of the remote RustDesk peer.
    *   **Notes:** Requires `feature = "cli"` to be enabled. Uses the key provided by `--key` if present.
    *   **Example:** `rustdesk --connect 123456789`

*   **`--connect <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new remote desktop session window.
    *   **Value Format:** `REMOTE_ID` (string). Can be accompanied by `--password <PASSWORD>`, `--relay`, `--switch_uuid <UUID>`.
    *   **Notes:** This is typically used via URI schemes or internal process calls to the main GUI instance. The new process usually exits after sending the command.
    *   **Windows Implementation Details:** Similar to URI Scheme handling, uses `platform::send_message_to_hnwd` with `WM_COPYDATA` to communicate the request to the main RustDesk process.
    *   **Example (internal/URI):** `rustdesk --connect 123456789 --password mypassword`

*   **`--play <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Alias for `--connect` when invoking a new remote desktop session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation).
    *   **Windows Implementation Details:** Uses `platform::send_message_to_hnwd`.

*   **`--file-transfer <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new file transfer session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for file transfer.
    *   **Windows Implementation Details:** Uses `platform::send_message_to_hnwd`.

*   **`--view-camera <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new view remote camera session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for viewing the remote camera.
    *   **Windows Implementation Details:** Uses `platform::send_message_to_hnwd`.

*   **`-p, --port-forward=[SPEC]` (CLI Feature)**
    *   **Purpose:** Sets up TCP port forwarding from a local port to a port on a remote RustDesk peer.
    *   **Value Format:** `SPEC` (string) - `remote-id:local-port:remote-port[:remote-host]`
        *   `remote-id`: ID of the remote peer.
        *   `local-port`: Local port number.
        *   `remote-port`: Remote port number on the peer.
        *   `remote-host` (optional): Target host on the remote peer's network (defaults to `localhost`).
    *   **Notes:** Requires `feature = "cli"` to be enabled. Uses the key provided by `--key` if present.
    *   **Example:** `rustdesk --port-forward 123456789:8080:80`
    *   **Example:** `rustdesk --port-forward 123456789:9090:3000:192.168.1.10`

*   **`--port-forward <REMOTE_ID:ARGS...>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new port forwarding session window/setup.
    *   **Value Format:** `REMOTE_ID` followed by port forwarding specifics, usually parsed from a URI.
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for port forwarding.
    *   **Windows Implementation Details:** Uses `platform::send_message_to_hnwd`.

*   **`--rdp <REMOTE_ID:ARGS...>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new RDP session window/setup (likely via port forwarding).
    *   **Value Format:** `REMOTE_ID` followed by RDP parameters, usually parsed from a URI.
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for RDP.
    *   **Windows Implementation Details:** Uses `platform::send_message_to_hnwd`.

*   **`-k, --key=[KEY]` (CLI Feature)**
    *   **Purpose:** Specifies an authentication key for rendezvous server communication or direct peer connection.
    *   **Value Format:** `KEY` (string) - The authentication key.
    *   **Notes:** Used with `--port-forward` and `--connect` (CLI).

## III. Server and Service Management Arguments

These arguments control the RustDesk server component and its lifecycle as an OS service.

*   **`-s, --server` or `--server=[]` (CLI Feature)**
    *   **Purpose:** Starts the RustDesk server component directly in the current process.
    *   **Value Format:** Can optionally take a value, but it's not used by the argument's logic; presence of the flag is key.
    *   **Notes:** Requires `feature = "cli"` to be enabled. This is for running the server manually, not as a managed OS service.
    *   **Example:** `rustdesk --server`

*   **`--server` (GUI/Service context)**
    *   **Purpose:** Starts the RustDesk server logic. This is often invoked by the OS service or when the GUI client needs to ensure its server part is running in the correct user session.
    *   **Value Format:** None (flag).
    *   **Notes:**
        *   On Linux, this may involve starting a `--tray` process.
        *   On Windows, it can call `privacy_mode::restore_reg_connectivity(true)`. This function reads display connectivity settings from the registry (`SYSTEM\CurrentControlSet\Control\GraphicsDrivers\Connectivity`) and can restore previous settings if changes made by a virtual display-based privacy mode are detected.
        *   On macOS, it starts the server in a thread and also launches the tray icon.
    *   **Example (internal):** `rustdesk --server`

*   **`--service`**
    *   **Purpose:** Runs the RustDesk application as an OS background service. This is the primary entry point for unattended access capabilities.
    *   **Value Format:** None (flag).
    *   **Notes:** This will initialize and run the main server logic, managed by the system's service controller. The process typically does not exit until the service is stopped.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `start_os_service()` calls `windows_service::service_dispatcher::start()`, which registers `ffi_service_main` (a wrapper for `service_main`).
        *   `service_main` then calls `run_service`.
        *   `run_service()`:
            *   Registers a service control handler using `RegisterServiceCtrlHandlerW` (via the `windows_service` crate) to respond to Service Control Manager (SCM) commands (Stop, Shutdown, etc.).
            *   Sets service status to `SERVICE_RUNNING` using `SetServiceStatus`.
            *   Enters a loop to monitor the active user session using `get_current_session` (from `windows.cc`, which calls `WTSGetActiveConsoleSessionId`).
            *   Launches the `--server` process in that session using `LaunchProcessWin` (from `windows.cc`, which uses `CreateProcessAsUserW`).
            *   Listens for IPC messages (e.g., `Data::Close` to stop, `Data::SAS` to trigger Secure Attention Sequence via `SendSAS` C function from `sas.dll`).
    *   **Example (internal, by service manager):** `rustdesk --service`

*   **`--install-service`**
    *   **Purpose:** Installs the RustDesk background OS service.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative/root privileges. The application will exit after attempting installation.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: The `install_service()` function.
        *   Generates and executes a batch script using `run_cmds` (which uses `runas::Command` for elevation).
        *   The script uses `sc.exe create <app_name> binpath= "<exe_path> --service" start= auto DisplayName= "<app_name> Service"` to create the Windows service.
        *   It also uses `sc.exe start <app_name>` to start the service.
        *   Copies a tray icon shortcut to the system startup folder.
    *   **Example:** `rustdesk --install-service` (requires elevation)

*   **`--uninstall-service`**
    *   **Purpose:** Uninstalls the RustDesk background OS service.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative/root privileges. The application will exit after attempting uninstallation.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: The `uninstall_service()` function.
        *   Sets a "stop-service" configuration option.
        *   Generates and executes a batch script using `run_cmds`.
        *   The script uses `sc.exe stop <app_name>` and `sc.exe delete <app_name>` to stop and remove the service.
        *   It also removes the tray icon startup shortcut and uses `taskkill` to terminate running RustDesk processes.
    *   **Example:** `rustdesk --uninstall-service` (requires elevation)

*   **`--no-server`**
    *   **Purpose:** When launching the main GUI, this flag prevents the automatic startup of its embedded server component.
    *   **Value Format:** None (flag).
    *   **Notes:** Useful if you only want to use RustDesk for outgoing connections from that specific GUI instance.
    *   **Example:** `rustdesk --no-server`

*   **`--tray`**
    *   **Purpose:** Starts the RustDesk system tray icon utility.
    *   **Value Format:** None (flag).
    *   **Notes:** Checks if another tray process is already running. The application will exit after launching the tray utility (which runs as a separate process).
    *   **Example:** `rustdesk --tray`

*   **`--quick_support`**
    *   **Purpose:** Runs RustDesk in a portable "quick support" mode, often without full installation.
    *   **Value Format:** None (flag).
    *   **Notes:**
        *   (Windows) May start a portable service if not elevated. Behavior can be influenced by executable naming (e.g., containing "-qs-") or the "pre-elevate-service" option.
        *   **Windows Implementation Details:**
            *   `src/core_main.rs`: Sets `_is_quick_support = true`.
            *   If not installed and not elevated, calls `portable_service::client::start_portable_service(client::StartPara::Direct)`.
            *   `portable_service::client::start_portable_service` (in `src/server/portable_service.rs`):
                *   Creates shared memory (`ShmemConf` from `shared_memory` crate, flink in user-accessible folder like `ProgramData` or `Windows\Temp`, permissions set via `icacls` command).
                *   Launches the current executable with `--portable-service` in the background using `platform::run_background` (which uses `ShellExecuteW` with `SW_HIDE`).
                *   Starts an IPC server (`client::start_ipc_server`) for communication with the portable service process.
    *   **Example:** `rustdesk-qs.exe` or `rustdesk --quick_support`

*   **`--portable-service` (Windows specific)**
    *   **Purpose:** Internal argument for the process that runs as the portable service, usually in an elevated or SYSTEM context.
    *   **Value Format:** None (flag).
    *   **Notes:**
        *   If this process isn't already SYSTEM, `core_main.rs` calls `platform::elevate_or_run_as_system` to attempt to relaunch as SYSTEM.
        *   Once running as SYSTEM, `portable_service::server::run_portable_service()` is executed.
        *   **Windows Implementation Details (`server::run_portable_service` in `src/server/portable_service.rs`):**
            *   Opens the existing shared memory created by the client part (`SharedMemory::open_existing(SHMEM_NAME)`).
            *   Spawns threads for:
                *   Screen capture (`scrap::Capturer`): Writes frame data to shared memory. Handles DXGI errors by potentially falling back to GDI. Uses Windows API `GetCursorInfo` via FFI for cursor handling.
                *   Cursor info (`winuser::GetCursorInfo` called directly in C and data written to shared memory): Writes cursor shape and position to shared memory.
                *   IPC client: Connects to the main GUI's IPC server to receive input events (mouse/keyboard) and relay them to `input_service` functions, and to monitor connection counts for auto-exit.
            *   Uses counters in shared memory for synchronization between reader/writer for screen frames and cursor data.

## IV. Installation and Update Arguments

These arguments are related to installing, uninstalling, or updating the RustDesk application.

*   **`--install`**
    *   **Purpose:** Initiates the installation process. This is often implicitly added when running a setup executable on Windows.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative privileges for a full system install.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs`: If `crate::common::is_setup(&arg_exe)` is true, `--install` is added to arguments.
        *   Flutter UI (`main.dart`) detects this and shows `InstallPage`.
        *   Actual installation logic is in `platform::install_me()` in `src/platform/windows.rs`.
        *   `install_me`:
            *   Generates and executes a complex batch script via `run_cmds` (elevated using `runas::Command`).
            *   Script performs: uninstallation of old versions (`get_uninstall`), directory creation (`md`), file copying (`XCOPY`, `copy` for main executable and `WIN_TOPMOST_INJECTED_PROCESS_EXE`), registry writes (`reg add` for uninstall info under `HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall`, file associations in `HKEY_CLASSES_ROOT`, URL protocol in `HKEY_CLASSES_ROOT`), shortcut creation (via `cscript` and temporary `.vbs` files for desktop and Start Menu), firewall rules (`netsh advfirewall`), service creation (`sc create` via `get_create_service`).
    *   **Example (often implicit):** `rustdesk_setup.exe` might run `rustdesk --install`

*   **`--silent-install [OPTIONS]` (Windows specific)**
    *   **Purpose:** Performs a silent (unattended) installation of RustDesk.
    *   **Value Format:** `[OPTIONS]` (string, optional) - A space-separated list of installation components (e.g., "desktopicon startmenu printer"). Default on Windows is "desktopicon startmenu printer".
    *   **Notes:** Requires administrative privileges. Disallowed if installation is globally disabled. The application exits.
    *   **Windows Implementation Details:**
        *   Calls `platform::install_me(options, "", true, ...)` in `src/platform/windows.rs`. The `true` flag indicates silent.
        *   Logic is similar to `--install` but non-interactive.
        *   Displays a Toast notification for success/failure using `tauri-winrt-notification` (calls `Toast::new().show()`).
    *   **Example:** `rustdesk_setup.exe --silent-install`
    *   **Example:** `rustdesk_setup.exe --silent-install "desktopicon startmenu"`

*   **`--noinstall`**
    *   **Purpose:** Prevents any implicit installation actions, such as when running a setup executable.
    *   **Value Format:** None (flag).
    *   **Notes:** Clears other arguments that might trigger installation.

*   **`--uninstall` (Windows specific)**
    *   **Purpose:** Uninstalls RustDesk from the system.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative privileges. The application exits after initiating uninstallation.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `platform::uninstall_me(true)`.
        *   `platform::uninstall_me` in `src/platform/windows.rs` calls `get_uninstall(true, true)` to generate a batch script.
        *   The script includes: stopping/deleting the service (`sc.exe stop/delete`), killing processes (`taskkill` for main exe and broker exe), removing registry keys (associations, uninstall info from `HKLM` and `HKCR`), uninstalling printer (`rustdesk.exe --uninstall-remote-printer`), uninstalling certificates (`rustdesk.exe --uninstall-cert`), deleting directories (`rd /s /q`), and removing shortcuts.
        *   Executed via `run_cmds` (elevated).
    *   **Example:** `rustdesk --uninstall` (from an installed location)

*   **`--update`**
    *   **Purpose:** Initiates the software update process for RustDesk.
    *   **Value Format:** None (flag).
    *   **Notes:** (Windows, macOS) Disallowed if installation (and thus updates) is globally disabled. The application exits after initiating the update.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `update_me()` function.
        *   Kills existing RustDesk processes (`taskkill`).
        *   Generates and executes a batch script via `run_cmds` (elevated).
        *   Script performs: registry updates for version info (`reg add` to uninstall key), copies new executable files (`XCOPY`, `copy`), restores service (`sc start`), reinstalls printer if it was present (via `--uninstall-remote-printer` then `--install-remote-printer`).
        *   Attempts to restore GUI and tray processes based on their previous session IDs using `run_exe_in_session` which calls `LaunchProcessWin`.
        *   Displays a Toast notification.
    *   **Example:** `rustdesk --update`

*   **`--after-install` (Windows specific)**
    *   **Purpose:** Executes post-installation tasks.
    *   **Value Format:** None (flag).
    *   **Notes:** Typically run automatically by the installer. The application exits.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `run_after_install()` calls `get_after_install()`.
        *   `get_after_install()` generates a batch script for setting up registry keys for file/URL associations in `HKEY_CLASSES_ROOT`, firewall rules (`netsh advfirewall add rule`), service creation (`sc.exe create` via `get_create_service`), and enabling Software SAS Generation (`reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v SoftwareSASGeneration /t REG_DWORD /d 1`). Executed via `run_cmds`.

*   **`--before-uninstall` (Windows specific)**
    *   **Purpose:** Executes pre-uninstallation tasks.
    *   **Value Format:** None (flag).
    *   **Notes:** Typically run automatically by the uninstaller. The application exits.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `run_before_uninstall()` calls `get_before_uninstall(true)`.
        *   `get_before_uninstall()` generates a batch script to stop/delete service (`sc.exe`), kill processes (`taskkill`), and remove registry associations from `HKEY_CLASSES_ROOT`. Executed via `run_cmds`.

*   **`--remove <FILEPATH>`**
    *   **Purpose:** Removes a specified file, usually the old executable during an update.
    *   **Value Format:** `FILEPATH` (string) - Path to the file to remove.
    *   **Notes:** Waits for 1 second before attempting removal. The application exits.
    *   **Example (internal):** `rustdesk --remove /path/to/old_rustdesk.exe`

*   **`--install-idd` (Windows specific)**
    *   **Purpose:** Installs the RustDesk Indirect Display Driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Only proceeds if virtual display is supported (`crate::virtual_display_manager::is_virtual_display_supported()`). The application exits.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `crate::virtual_display_manager::rustdesk_idd::install_update_driver()`.
        *   This function resides in `libs/virtual_display/src/lib.rs` (as `install_update_driver` from the `LIB_WRAPPER`) which dynamically loads `dylib_virtual_display.dll`.
        *   The DLL is expected to use Windows SetupAPI (e.g., `UpdateDriverForPlugAndPlayDevicesW`) for driver installation.
    *   **Example:** `rustdesk --install-idd`

*   **`--uninstall-idd` (Windows specific, referring to RustDesk IDD)**
    *   **Purpose:** Uninstalls the RustDesk Indirect Display Driver.
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.
    *   **Windows Implementation Details:**
        *   This specific flag for the RustDesk IDD seems to be missing a direct handler in `core_main.rs`. The documentation should clarify if this is handled by the generic `--uninstall-service` or another mechanism. `libs/virtual_display/src/lib.rs` does expose an `uninstall_driver` function pointer from the DLL, which would likely use SetupAPI (e.g., `DiUninstallDevice`).

*   **`--uninstall-amyuni-idd` (Windows specific)**
    *   **Purpose:** Uninstalls the Amyuni Indirect Display Driver.
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `crate::virtual_display_manager::amyuni_idd::uninstall_driver()`.
        *   This, similar to RustDesk IDD, calls the `uninstall_driver` function pointer from the loaded `dylib_virtual_display.dll`.

*   **`--install-remote-printer` (Windows specific)**
    *   **Purpose:** Installs the RustDesk remote printer driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires Windows 10 or greater (`crate::platform::is_win_10_or_greater()`). The application exits.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `remote_printer::install_update_printer(&crate::get_app_name())`.
        *   This function (in `libs/remote_printer/src/windows.rs` via `libs/remote_printer/src/setup.rs`) uses Windows Print Spooler APIs. Key steps involve:
            *   `AddPrinterDriverExW` using the INF file at `drivers/RustDeskPrinterDriver/RustDeskPrinterDriver.inf`.
            *   Creating a printer port (e.g., using `AddPortExW` or XcvData API).
            *   `AddPrinterW` to add the printer instance.
    *   **Example:** `rustdesk --install-remote-printer`

*   **`--uninstall-remote-printer` (Windows specific)**
    *   **Purpose:** Uninstalls the RustDesk remote printer driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires Windows 10 or greater. The application exits.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `remote_printer::uninstall_printer(&crate::get_app_name())`.
        *   This function uses Windows Print Spooler APIs: `DeletePrinter`, `DeletePortW`, and `DeletePrinterDriverExW`.

*   **`--uninstall-cert` (Windows specific)**
    *   **Purpose:** Uninstalls the RustDesk certificate (likely for secure local communication or driver signing).
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.
    *   **Windows Implementation Details:**
        *   `src/core_main.rs` calls `crate::platform::windows::uninstall_cert()`.
        *   This calls `cert::uninstall_cert()` which in turn calls the C function `DeleteRustDeskTestCertsW` (defined in `src/platform/windows.cc`). This C function would use Windows Certificate Store APIs (e.g., `CertOpenStore`, `CertFindCertificateInStore`, `CertDeleteCertificateFromStore`).

## V. Configuration Arguments

These arguments are used to set, get, or import RustDesk configurations.

*   **`--import-config <FILEPATH>`**
    *   **Purpose:** Imports configuration settings from a specified TOML file.
    *   **Value Format:** `FILEPATH` (string) - Path to the `.toml` configuration file.
    *   **Notes:** Imports `Config` and `Config2` if the source file is newer than existing configs and older than the executable. The application exits.
    *   **Example:** `rustdesk --import-config /path/to/myconfig.toml`

*   **`--password <PASSWORD>`**
    *   **Purpose:** Sets the permanent password for unattended access.
    *   **Value Format:** `PASSWORD` (string).
    *   **Notes:** Requires RustDesk to be installed and run with administrative/root privileges. The application exits.
    *   **Example:** `rustdesk --password "mypassword123"`

*   **`--set-unlock-pin <PIN>` (Flutter Feature)**
    *   **Purpose:** Sets a PIN to unlock the RustDesk settings UI.
    *   **Value Format:** `PIN` (string).
    *   **Notes:** Requires `feature = "flutter"`, RustDesk installed, and administrative/root privileges. The application exits.
    *   **Example:** `rustdesk --set-unlock-pin 1234`

*   **`--get-id`**
    *   **Purpose:** Prints the current RustDesk ID to the console.
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.
    *   **Example:** `rustdesk --get-id`

*   **`--set-id <NEW_ID>`**
    *   **Purpose:** Changes the RustDesk ID.
    *   **Value Format:** `NEW_ID` (string).
    *   **Notes:** Requires RustDesk to be installed and run with administrative/root privileges. The application exits.
    *   **Example:** `rustdesk --set-id 987654321`

*   **`--config <CONFIG_STRING>`**
    *   **Purpose:** Configures custom server settings (ID/Rendezvous, Relay, API) from an encrypted string, often embedded in a custom client executable's name.
    *   **Value Format:** `CONFIG_STRING` (string) - e.g., `rustdesk-host=my.server.com,key=mykey.exe`
    *   **Notes:** Requires RustDesk to be installed and run with administrative/root privileges. The application exits.
    *   **Example:** `rustdesk --config "rustdesk-host=hbbs.example.com,key=mylicensekey.exe"`

*   **`--option <OPTION_NAME> [VALUE]`**
    *   **Purpose:** Gets or sets a specific advanced configuration option.
    *   **Value Format:**
        *   To get: `<OPTION_NAME>`
        *   To set: `<OPTION_NAME> <VALUE>`
    *   **Notes:** Requires RustDesk to be installed and run with administrative/root privileges. The application exits.
    *   **Example (get):** `rustdesk --option custom-rendezvous-server`
    *   **Example (set):** `rustdesk --option custom-rendezvous-server hbbs.example.com`

*   **`--assign --token <TOKEN> [options...]`**
    *   **Purpose:** Assigns the device to a user, group, or applies a strategy on a RustDesk Pro server.
    *   **Value Format:** Requires `--token <TOKEN>`. Optional arguments include:
        *   `--user_name <USER_NAME>`
        *   `--strategy_name <STRATEGY_NAME>`
        *   `--address_book_name <AB_NAME>`
        *   `--address_book_tag <AB_TAG>` (used with `--address_book_name`)
        *   `--address_book_alias <AB_ALIAS>` (used with `--address_book_name`)
        *   `--device_group_name <DG_NAME>`
    *   **Notes:** Requires RustDesk to be installed and run with administrative/root privileges. At least one of user_name, strategy_name, address_book_name, or device_group_name is required. The application exits.
    *   **Example:** `rustdesk --assign --token "myapitoken" --user_name "john.doe"`

## VI. Privilege and Process Management Arguments

These arguments are primarily for managing process privileges or internal process control.

*   **`--elevate` (Windows specific)**
    *   **Purpose:** Restarts the RustDesk process with elevated (administrator) privileges.
    *   **Value Format:** None (flag).
    *   **Notes:** Used when an action requires admin rights and the current process is not elevated. The current process will exit.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `elevate()` function calls `run_uac()`.
        *   `run_uac()` uses `ShellExecuteW` with the `runas` verb to trigger a UAC prompt for the current executable.
        *   The `is_elevated()` function uses `OpenProcessToken` and `GetTokenInformation` (with `TokenElevation`) to check the elevation state of a process.

*   **`--run-as-system` (Windows specific)**
    *   **Purpose:** Attempts to run the RustDesk process as the SYSTEM user.
    *   **Value Format:** None (flag).
    *   **Notes:** Used for operations requiring the highest level of system privileges. The current process will exit.
    *   **Windows Implementation Details:**
        *   `src/platform/windows.rs`: `run_as_system()` function utilizes the `impersonate_system` crate. This crate likely uses low-level Windows APIs such as `OpenProcessToken`, `LookupPrivilegeValueW`, `AdjustTokenPrivileges`, `LogonUserW`, `CreateProcessWithTokenW` or similar to acquire necessary privileges and create a process as SYSTEM.
        *   The C function `is_local_system()` (in `src/platform/windows.cc`) is called by the Rust function `is_root()` to check if the current process is already running as SYSTEM. It uses `OpenProcessToken`, `GetTokenInformation` and `EqualSid` with the `SECURITY_LOCAL_SYSTEM_RID`.

*   **`-gtk-sudo` (Linux specific)**
    *   **Purpose:** Internal argument for executing commands with root privileges using a graphical sudo prompt.
    *   **Value Format:** Followed by the command and its arguments to be executed.
    *   **Notes:** Handled by `crate::platform::gtk_sudo::exec()`. The application exits.

## VII. Connection Manager Arguments

These arguments are related to the Connection Manager feature.

*   **`--cm`**
    *   **Purpose:** Starts the RustDesk Connection Manager UI.
    *   **Value Format:** None (flag).
    *   **Notes:** This will launch the Flutter UI in Connection Manager mode.

*   **`--cm-no-ui`**
    *   **Purpose:** Starts the Connection Manager background logic without launching its Flutter UI.
    *   **Value Format:** None (flag).
    *   **Notes:** Likely for scenarios where CM functionality is needed without visual interaction. The application exits.

## VIII. Plugin Framework Arguments (Feature: `plugin_framework`)

These arguments manage plugins if the `plugin_framework` feature is enabled.

*   **`--plugin-install <PLUGIN_NAME_OR_URL> [CHECKSUM_OR_URL]`**
    *   **Purpose:** Installs or enables a plugin.
    *   **Value Format:**
        *   `<PLUGIN_NAME>`: Enables a previously downloaded/installed plugin.
        *   `<URL_TO_PLUGIN_ZIP> <EXPECTED_CHECKSUM>`: Downloads and installs a new plugin.
        *   `<PLUGIN_NAME> <URL_TO_UPDATE_FROM>`: Updates an existing plugin.
    *   **Notes:** Requires `feature = "plugin_framework"`. The application exits.

*   **`--plugin-uninstall <PLUGIN_NAME>`**
    *   **Purpose:** Uninstalls or disables a plugin.
    *   **Value Format:** `<PLUGIN_NAME>` (string).
    *   **Notes:** Requires `feature = "plugin_framework"`. The application exits.

## IX. Hardware and System Feature Arguments

*   **`--check-hwcodec-config`**
    *   **Purpose:** Performs a check or process related to hardware codec (encoder/decoder) configuration.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires `feature = "hwcodec"` to be enabled. The application exits.
    *   **Example:** `rustdesk --check-hwcodec-config`

This document should serve as a good reference for understanding and using the RustDesk command-line interface. Note that some arguments are platform-specific or depend on compiled features.
