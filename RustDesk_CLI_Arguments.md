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
    *   **Purpose:** (Linux specific with Flutter UI) Handles custom URI schemes to launch RustDesk sessions or perform actions. This is often used for deep linking.
    *   **Value Format:** A URI string (e.g., `rustdesk://connect/123456789?password=abc`).
    *   **Notes:** When a URI is passed, RustDesk attempts to send it to an already running instance via D-Bus. If successful, the current process exits. Otherwise, it might proceed to start a new GUI instance to handle the URI.
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
    *   **Example (internal/URI):** `rustdesk --connect 123456789 --password mypassword`

*   **`--play <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Alias for `--connect` when invoking a new remote desktop session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation).

*   **`--file-transfer <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new file transfer session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for file transfer.

*   **`--view-camera <REMOTE_ID>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new view remote camera session window.
    *   **Value Format:** `REMOTE_ID` (string).
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for viewing the remote camera.

*   **`-p, --port-forward=[SPEC]` (CLI Feature)**
    *   **Purpose:** Sets up TCP port forwarding from a local port to a port on a remote RustDesk peer.
    *   **Value Format:** `SPEC` (string) - `remote-id:local-port:remote-port[:remote-host]`
        *   `remote-id`: ID of the remote peer.
        *   `local-port`: Local port number.
        *   `remote-port`: Remote port number on the peer.
        *   `remote-host` (optional): Target host on the remote peer's network (defaults to `localhost`).
    *   **Notes:** Requires `feature = "cli"` to be enabled. Uses the key provided by `--key` if present.
    *   **Example:** `rustdesk --port-forward 123456789:8080:80` (forwards local port 8080 to remote peer 123456789's port 80)
    *   **Example:** `rustdesk --port-forward 123456789:9090:3000:192.168.1.10` (forwards local 9090 to 192.168.1.10:3000 on remote peer's network)

*   **`--port-forward <REMOTE_ID:ARGS...>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new port forwarding session window/setup.
    *   **Value Format:** `REMOTE_ID` followed by port forwarding specifics, usually parsed from a URI.
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for port forwarding.

*   **`--rdp <REMOTE_ID:ARGS...>` (Flutter UI Invocation)**
    *   **Purpose:** Instructs an existing RustDesk instance to open a new RDP session window/setup (likely via port forwarding).
    *   **Value Format:** `REMOTE_ID` followed by RDP parameters, usually parsed from a URI.
    *   **Notes:** Same as `--connect` (Flutter UI Invocation) but for RDP.

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
    *   **Purpose:** Starts the RustDesk server logic. This is often invoked by the OS service or when the GUI client needs to ensure its server part is running in the correct user session (especially on Linux).
    *   **Value Format:** None (flag).
    *   **Notes:**
        *   On Linux, this may involve starting a `--tray` process.
        *   On Windows, it can restore registry connectivity if privacy mode was active.
        *   On macOS, it starts the server in a thread and also launches the tray icon.
    *   **Example (internal):** `rustdesk --server`

*   **`--service`**
    *   **Purpose:** Runs the RustDesk application as an OS background service. This is the primary entry point for unattended access capabilities.
    *   **Value Format:** None (flag).
    *   **Notes:** This will initialize and run the main server logic, managed by the system's service controller (e.g., systemd on Linux, Services on Windows). The process typically does not exit until the service is stopped.
    *   **Example (internal, by service manager):** `rustdesk --service`

*   **`--install-service`**
    *   **Purpose:** Installs the RustDesk background OS service.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative/root privileges. The application will exit after attempting installation.
    *   **Example:** `sudo rustdesk --install-service`

*   **`--uninstall-service`**
    *   **Purpose:** Uninstalls the RustDesk background OS service.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires administrative/root privileges. The application will exit after attempting uninstallation.
    *   **Example:** `sudo rustdesk --uninstall-service`

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
        *   (Windows) May start a portable service if not elevated.
        *   Behavior can be influenced by executable naming (e.g., containing "-qs-").
    *   **Example:** `rustdesk-qs.exe` or `rustdesk --quick_support`

*   **`--portable-service` (Windows specific)**
    *   **Purpose:** Used internally for managing the portable service, often ensuring it runs with necessary privileges.
    *   **Value Format:** None (flag).
    *   **Notes:** May trigger elevation. The process typically exits.

## IV. Installation and Update Arguments

These arguments are related to installing, uninstalling, or updating the RustDesk application.

*   **`--install`**
    *   **Purpose:** Initiates the installation process. This is often implicitly added when running a setup executable on Windows.
    *   **Value Format:** None (flag).
    *   **Notes:** (Windows) Triggers the `InstallPage` in the Flutter UI. Requires administrative privileges for a full system install.
    *   **Example (often implicit):** `rustdesk_setup.exe` might run `rustdesk --install`

*   **`--silent-install [OPTIONS]` (Windows specific)**
    *   **Purpose:** Performs a silent (unattended) installation of RustDesk.
    *   **Value Format:** `[OPTIONS]` (string, optional) - A space-separated list of installation components (e.g., "desktopicon startmenu printer").
    *   **Notes:** Requires administrative privileges. Disallowed if installation is globally disabled.
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
    *   **Example:** `rustdesk --uninstall` (from an installed location)

*   **`--update`**
    *   **Purpose:** Initiates the software update process for RustDesk.
    *   **Value Format:** None (flag).
    *   **Notes:** (Windows, macOS) Disallowed if installation (and thus updates) is globally disabled. The application exits after initiating the update.
    *   **Example:** `rustdesk --update`

*   **`--after-install` (Windows specific)**
    *   **Purpose:** Executes post-installation tasks.
    *   **Value Format:** None (flag).
    *   **Notes:** Typically run automatically by the installer. The application exits.

*   **`--before-uninstall` (Windows specific)**
    *   **Purpose:** Executes pre-uninstallation tasks.
    *   **Value Format:** None (flag).
    *   **Notes:** Typically run automatically by the uninstaller. The application exits.

*   **`--remove <FILEPATH>`**
    *   **Purpose:** Removes a specified file, usually the old executable during an update.
    *   **Value Format:** `FILEPATH` (string) - Path to the file to remove.
    *   **Notes:** Waits for 1 second before attempting removal. The application exits.
    *   **Example (internal):** `rustdesk --remove /path/to/old_rustdesk.exe`

*   **`--install-idd` (Windows specific)**
    *   **Purpose:** Installs the RustDesk Indirect Display Driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Only proceeds if virtual display is supported. The application exits.
    *   **Example:** `rustdesk --install-idd`

*   **`--uninstall-idd` (Windows specific, referring to RustDesk IDD)**
    *   **Purpose:** Uninstalls the RustDesk Indirect Display Driver. (Note: `core_main.rs` uses `rustdesk_idd::install_update_driver` for `--install-idd` and `amyuni_idd::uninstall_driver` for `--uninstall-amyuni-idd`. Assuming this refers to the RustDesk IDD based on the install counterpart).
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.

*   **`--uninstall-amyuni-idd` (Windows specific)**
    *   **Purpose:** Uninstalls the Amyuni Indirect Display Driver.
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.

*   **`--install-remote-printer` (Windows specific)**
    *   **Purpose:** Installs the RustDesk remote printer driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires Windows 10 or greater. The application exits.
    *   **Example:** `rustdesk --install-remote-printer`

*   **`--uninstall-remote-printer` (Windows specific)**
    *   **Purpose:** Uninstalls the RustDesk remote printer driver.
    *   **Value Format:** None (flag).
    *   **Notes:** Requires Windows 10 or greater. The application exits.

*   **`--uninstall-cert` (Windows specific)**
    *   **Purpose:** Uninstalls the RustDesk certificate (likely for secure local communication or driver signing).
    *   **Value Format:** None (flag).
    *   **Notes:** The application exits.

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

*   **`--run-as-system` (Windows specific)**
    *   **Purpose:** Attempts to run the RustDesk process as the SYSTEM user.
    *   **Value Format:** None (flag).
    *   **Notes:** Used for operations requiring the highest level of system privileges. The current process will exit.

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
