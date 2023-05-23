## Moving things away from pkghandler
- [ ] `get_system_packages_for_replacement`
- [ ] `get_installed_pkg_information` Could be moved to be in the RPM class
- [ ] `get_installed_pkg_objects` 
- [ ] `call_yum_cmd` Is a special one... Maybe we could place it in a `utils.py` for pkgmanager
- [ ] `get_rpm_header` could probably live inside a RPM class as it is very specific
