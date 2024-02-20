### Overview

Tested with:
- Ansible 2.15.9
- Ansible AWX 23.5.2
- Python 3.10.12

This automation suite is divided into four main parts:

1. **teltonika-firmware-upgrade.yml**
2. **preparation_tasks.yml**
3. **upgrade.yml**
4. **packages_tasks.yml**
5. **restore_tasks.yml**

### Main Upgrade Playbook (teltonika-firmware-upgrade.yml)

This playbook is the orchestration center for the entire upgrade process, integrating the above components based on the router's current and target firmware versions:

- **Safety checks and initial setup**: It begins with ensuring that the automation process is being executed under safe conditions (e.g., limiting the scope of devices being targeted).
- **Determining upgrade path**: Based on the current firmware version, it decides whether a one-step or two-step upgrade is necessary, pulling in tasks from `upgrade.yml` as required.
- **Handling package installations**: For routers already at the target firmware version, it proceeds to manage package installations, ensuring that the device is not only up-to-date in terms of firmware but also has all the necessary software packages installed.

### Preparation Tasks (preparation_tasks.yml)

Preparation tasks are crucial for ensuring that the router is ready for upgrades or configuration changes, focusing on system cleanliness and access:

- **Cleanup from previously failed tasks**: Clears any temporary files or directories left from previous automation attempts, ensuring a clean slate for the current process.
- **Wait for unimpeded access to opkg(solo)**: This task ensures exclusive access to the package management system by temporarily renaming `opkg` to avoid conflicts with other operations, a critical step for maintaining system stability.

### Upgrade Process (upgrade.yml)

Dedicated to the firmware upgrade process, this file handles the critical aspects of ensuring a successful firmware upgrade:

- **Backup current configuration**: Before any upgrade attempt, it backs up the current configuration, safeguarding against data loss.
- **Firmware transfer and upgrade**: This includes transferring the new firmware file to the router and executing the upgrade process, with careful monitoring to ensure success.
- **Verification and cleanup**: After the upgrade, it verifies the new firmware version and cleans up temporary files, ensuring that the system is left in a clean and operational state.

### Packages Tasks (packages_tasks.yml)

This section automates the process of ensuring that necessary packages and their dependencies are installed on the device. It's designed to handle various scenarios where different firmware versions require specific packages. Here's a breakdown of its key components:

- **Check package dependencies**: Before attempting any installation, it checks if the required packages and their dependencies are already installed, optimizing the process by avoiding unnecessary operations.
- **Ensure interface `wwan0` is down**: This step is crucial to prevent the device from attempting to download packages from the internet, ensuring that only the packages provided within the automation process are used, which is important for both security and controlling data usage.
- **Copy package archives to the device**: This task transfers the necessary package files from the control machine to the router, preparing for installation.
- **Extract packages**: After transferring the package archives, this step handles the extraction of these packages in a temporary directory on the device, setting the stage for the installation.
- **Install packages**: Leveraging the `opkg` package manager, this step installs the extracted packages, with retries implemented to handle potential transient issues.

### Restore Tasks (restore_tasks.yml)

Once the main operations are complete, these tasks aim to return the system to its standard operational state:

- **Wait for opkg to be restored**: Similar to the preparation phase but in reverse, this ensures that the package manager is available under its standard name (`opkg`), signifying the end of the automation process and the restoration of normal operations.

### Purpose and Use

The structured approach provided by these YAML files serves multiple purposes:

- **Minimizing Downtime**: By carefully managing each step of the process, the automation ensures that routers are quickly returned to operational status with the new firmware and necessary packages.
- **Ensuring Compatibility**: The tasks account for dependencies and compatibility issues, ensuring that upgrades and package installations do not disrupt the router's functionality.
- **Preserving System Integrity**: Through backups and careful management of system files, the process aims to preserve the integrity of the router's configuration and state, ensuring that upgrades can be rolled back if necessary.
- **Automating Routine Processes**: By automating these tasks, the system reduces the potential for human error and significantly increases the efficiency of managing firmware upgrades across multiple devices.
