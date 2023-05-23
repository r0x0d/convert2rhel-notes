
## BasePackageManager

```mermaid
classDiagram
BasePackageManager --|> Yum
BasePackageManager --|> Dnf

BasePackageManager : get_installed_packages()
BasePackageManager : install()
BasePackageManager : upgrade()
BasePackageManager : downgrade()
BasePackageManager : reinstall()

```