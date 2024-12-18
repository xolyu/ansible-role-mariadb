# mariadb

<!-- [![CI](https://github.com/xolyu/ansible-role-ROLE_NAME/actions/workflows/ci.yml/badge.svg)](https://github.com/xolyu/ansible-role-ROLE_NAME/actions/workflows/ci.yml) -->

Installs and configures the MariaDB database server. Creates databases and users.


## Requirements

* Systempackage `python3-pymysql` &ndash; for Ansible's MySQL modules [`mysql_user`](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html) and [`mysql_db`](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_db_module.html).

*For automatic ensuring of the packages, see variable `mariadb_ensure_requirements`.*


## Dependencies

* [`Community.Mysql`](https://docs.ansible.com/ansible/latest/collections/community/mysql/index.html) collection


## Role Variables

* **`mariadb_ensure_requirements`**  
  Provides for the installation of the packages listed in section requirements.  
  Type: bool  
  Default: `no`

* **`mariadb_enabled_on_startup`**  
  Defines if the MariaDB service should be enabled.  
  Type: bool  
  Default: `yes`

* **`mariadb_secure_installation`**  
  Defines whether the "secure installation" section should be executed, similar to MariaDB's "secure_installation" script.  
  Choices: `on_install`, `always`, `never`  
  Default: `on_install`

* **`mariadb_config_style`**  
  Defines the configuration style.  
  `single_cnf` means that only the global `mariadb.cnf` file is filled, include files will not be touched, while `included_cnf` ensures the existence of the `mariadb.conf.d` directory and manages the include files in addition to a minimal configuration of `mariadb.cnf`. With `none` is no configuration carried out.  
  Choices: `single_cnf`, `included_cnf`, `none`  
  Default: `single_cnf`


* **`mariadb_root_username`**  
  Username of the MariaDB's root user. For normal reason it should be `root`.  
  Type: str  
  Default: `root`

* **`mariadb_root_password`**  
  Password for the MariaDB's root user.  
  Type: str  
  Default: _undefined_

* **`mariadb_root_auth_by_unix_socket`**  
  Defines whether the `unix_socket` authentication plugin is activated for root user or not.  
  Type: bool  
  Default: `yes`

* **`mariadb_root_home`**  
  Home directory of the root user. This value is only used if root password is used without `unix_socket` auth, so the `.my.cnf` file with auth username and password is saved to root's home directory.  
  Type: str  
  Default: `/root`

* **`mariadb_root_auth_update`**  
  The authentication methods for root are set during installation or when this variable is set to `true`.  
  In the case of authentication with a password, this means that this only needs to be present once during installation, but no longer afterwards, or only if it is changed.  
  Type: bool  
  Default: `no`

* **`mariadb_admin_username`**  
  Username of an admin user, next to the root user.  
  This could be an admin user for Ansible so that Ansible can make administrative changes to the database without having to act as root.  
  Type: str  
  Default: _undefined_

* **`mariadb_admin_password`**  
  Password of the admin user.  
  Type: str  
  Default: _undefined_

* **`mariadb_admin_sysuser`**  
  System user name for the user from which the database is to be used as admin.  
  Type: str  
  Default: _undefined_

* **`mariadb_admin_home`**  
  Home directory of the system user. The `.my.cnf` file with the authentication data of the database admin user is saved in this directory.  
  Type: str  
  Default: _undefined_


* **`mariadb_config`**  
  Describes the configuration for MariaDB, organized into individual files in the case of `included_cnf`.  
  Type: Dict of Dict of Dict  
  Default: _see defaults/main.yml_

  ```yml
  mariadb_config:
    {cnf_file}:
      $state: present  # present, disabled, absent
      {section}:
        {option}: {value}
  ```

  * `cnf_file`: Name of the file (without extension) in `mariadb.conf.d` directory (_see variable `mariadb_configs_dir`_).  
    Special names:
    * `global_cnf` is used for config of global mariadb.cnf file, in case of `included_cnf` config style.
    * `single_cnf` is used as basis for the single (global) mariadb.cnf file, in case of `single_cnf` config style.
  * `section`: The INI section in MariaDB's config file.  
    Special names:
    * `$state` is the state of the config file. Default is `present`, more choices `disabled` and `absent`.
    * `$header` is printed on top of all sections in the cnf file, but not automatically converted as comment.
  * `option`: Option within the section.
  * `value`: Value for the option.  
    Special values:
    * If the value is a List, the option is repeated for every list item.
    * `$set` or an empty string (length == 0) means the option is set without a value.
    * `$unset` means that the option will not be present in the cnf file.
    * `$var(VAR_NAME)` means a lookup is performed for the `VAR_NAME` variable at the time of evaluation.

* **`mariadb_config_extra_1`, `mariadb_config_extra_2`, `mariadb_config_extra_3`**  
  Exactly the same as `mariadb_config`. These variables enable additional enrichment of the configuration without having to completely redefine everything. The extra variables overwrite previously defined values, with a higher number outweighing the lower number.  
  If a value that is defined in advance is to be removed, this can be done using the special value `$unset`.  
  Default: _undefined_

* **`mariadb_packages`**  
  List of packages to be installed for the MariaDB server.  
  Type: List of str  
  Default: _depends on OS, default see vars/\[OS-family\].yml_

* **`mariadb_config_file`**  
  Path of the global mariadb.cnf file.  
  Type: str  
  Default: _depends on OS, default see vars/\[OS-family\].yml_

* **`mariadb_configs_dir`**  
  Path of the include directory for the cnf files.  
  Type: str  
  Default: _depends on OS, default see vars/\[OS-family\].yml_

* **`mariadb_socket`**  
  Path of the socket file of MariaDB server instance.  
  Type: str  
  Default: _depends on OS, default see vars/\[OS-family\].yml_

* **`mariadb_pid_file`**  
  Path of the pid file of MariaDB server instance.  
  Type: str  
  Default: _depends on OS, default see vars/\[OS-family\].yml_

* **`mariadb_bind_address`**  
  Bind address for MariaDB server.
  Is used in default config with `$var(...)` value.  
  Type: str  
  Default: `127.0.0.1`

* **`mariadb_datadir`**  
  Is used to ensure the data directory with the permissions.  
  In case of changing from default, it needed to be added to the config, e.g. `datadir: $var(mariadb_datadir)`.
  Type: str  
  Default: `/var/lib/mysql`

* **`mariadb_encoding`**  
  Encoding settings for MariaDB config.
  Is used in default config with `$var(...)` value.  
  Type: str  
  Default: `utf8mb4`

* **`mariadb_collation`**  
  Collation settings for MariaDB config.
  Is used in default config with `$var(...)` value.  
  Type: str  
  Default: `utf8mb4_general_ci`

* **`mariadb_databases`**  
  DESC  
  Type: List of Dicts  
  Default: `[]`

  ```yml
  mariadb_databases:
    - name: example
      encoding: utf8
      collation: utf8_general_ci
      state: present  # present, absent
  ```

* **`mariadb_users`**  
  DESC  
  Type: List of Dicts  
  Default: `[]`

  ```yml
  mariadb_users:
    - name: example
      host: 'localhost'
      password: TOP_secret
      priv: '*.*:USAGE'
      state: present  # present, absent
      append_privs: no  # no, yes
      encrypted: no  # no, yes
  ```


<!--
* **`VAR`**  
  DESC  
  Choices: `VAL`, `ANOTHER`  
  Default: `VAL`

* **`VAR`**  
  DESC  
  Type: List of Dicts  
  Default: `VAL`
-->


## Example Playbook

Examples of playbooks using and configuring the role.


## License

GNU General Public License v3.0


## Author Information

Xolyu.
