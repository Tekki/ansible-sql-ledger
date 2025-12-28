# Ansible Role for SQL-Ledger

This role allows you to install the current version of
[SQL-Ledger](https://github.com/Tekki/sql-ledger) on Debian Trixie.

## Older Versions

The code from this branch installs SQL-Ledger version 4. If your
playbook was made for version 3, refer to the list below to update the
variables.

To install an older version, switch to the [branch
version3](https://github.com/Tekki/ansible-sql-ledger/tree/version3).
Changing variable `sl_git_branch` to `full` or `dws` will fail because
of the different configuration file formats.

## Prerequisites

The machine from which the playbook is run needs a recent Ansible
version. In Debian and most other Linux distributions, Ansible is
available through the official package repository.

The target machine needs SSH access, Python and sudo. Standard Debian
comes with Python, sudo and SSH server are optional. For sudo, install
Debian without root password. The SSH server has to be activated
during «Software selection». Deselect all the rest.

After the basic installation, log in to the server with SSH to test
the connection.

## Role Variables

The following variables can be passed to this role:

### Variables for Operating System

| Variable Name           | Default Value | Description                         |
|-------------------------|---------------|-------------------------------------|
| sl\_lx\_locales         | ['en_US']     | list of the available UTF-8 locales |
| sl\_lx\_default\_locale | en_US         | default UTF-8 locale                |
| sl\_lx\_timezone        | UTC           | time zone for server                |
| sl\_texlive\_languages  | ['english']   | list of the TeX Live languages      |

### Variables for SQL-Ledger

| Variable Name         | Default Value                           | Description                                      |
|-----------------------|-----------------------------------------|--------------------------------------------------|
| sl\_admin\_pwd        | *undefined*                             | password for admin.pl                            |
| sl\_gpg\_path         | /var/www/gnupg                          | home directory for gpg                           |
| sl\_git\_branch       | main                                    | branch that will be checked out                  |
| sl\_git\_source       | https://github.com/Tekki/sql-ledger.git | URL of the Git repository                        |
| sl\_helpful\_login    | false                                   | helpful error messages on login screen           |
| sl\_httpd\_config     | /etc/apache2                            | path to the webserver config                     |
| sl\_httpd\_path       | /var/www/sql-ledger                     | local path of the installation                   |
| sl\_httpd\_url        | sql-ledger                              | browser URL on the server                        |
| sl\_httpd\_whitelist  | *undefined*                             | list of allowed IP addresses                     |
| sl\_latex             | true                                    | install and use LaTeX                            |
| sl\_latex\_dvipdf     | false                                   | use dvipdf instead of pdflatex                   |
| sl\_latex\_pdftk      | true                                    | use pdftk to combine PDFs                        |
| sl\_latex\_xelatex    | true                                    | use XeLaTeX instead of pdflatex                  |
| sl\_login\_language   |                                         | language of the login screen                     |
| sl\_postgres\_user    | sql-ledger                              | user name to connect to PostgreSQL               |
| sl\_protect\_admin    | false                                   | protect admin interface                          |
| sl\_protect\_password | *undefined*                             | .htaccess password for protected admin interface |
| sl\_protect\_username | *undefined*                             | .htaccess username for protected admin interface |
| sl\_sendmail          | '\| /usr/sbin/sendmail -f <%from%> -t'  | pipe to sendmail                                 |
| sl\_totp\_admin       | false                                   | TOTP for admin activated                         |

Always activate all locales you anticipate needing on the first run.

If other packages or Perl modules from CPAN have to be installed, add
them to the variables `debian_additional_packages` or
`debian_perl_additional_cpan`.

Please notice that this role doesn't install a mail transport agent or
a printing system.

## Example Playbook

Assuming that your server has the name 'sl_server', create a directory
and file structure that looks like this:

```
./
├── group_vars/
│   └── sl_server/
│       ├── sl_server_vars.yml
│       └── vault_vars.yml
├── roles/
│   └── sql-ledger/
├── hosts
└── sl_server.yml
```

Start with adding the IP address or URL of the SQL-Ledger server to
`hosts`:

```conf
[sl_server]
$IP_OR_URL
```

Next test the Ansible connection.

```bash
ansible sl_server -m ping
```

Create the playbook `sl_server.yml`:

```yaml
- hosts: sl_server
  roles:
    - name: sql-ledger
      tags: [sl]
```

Clone the code of this repository to the `roles` directory:

```bash
git clone https://github.com/Tekki/ansible-sql-ledger roles/sql-ledger
```

To install a server with

- Central European Time (Zurich)
- languages German/Switzerland, French/Switzerland, Italian/Switzerland
- German (chd\_utf) as system default and language of the login screen

set the parameters in `sl_server_vars.yml` to 

```yaml
lx_locales:
  - de_CH
  - fr_CH
  - it_CH
lx_default_locale: de_CH
lx_timezone: Europe/Zurich
texlive_languages:
  - german
  - french
  - italian
sl_login_language: chd_utf
```

Never type cleartext passwords into the config file. Always use the
[vault](https://docs.ansible.com/projects/ansible/latest/vault_guide/index.html)
to protect them.

```yaml
ansible_become: true
ansible_become_pass: "{{ vault_sudo_pwd }}"

sl_admin_pwd: "{{ vault_sl_admin_pwd }}"
```

If the `sudo` password is 'Abc9876' and the password for `admin.pl`
should be set to '12345678' (both not a good idea), open
`vault_vars.yml`

```bash
ansible-vault edit group_vars/sl_server/vault_vars.yml
```

and add the passwords:

```yaml
vauls_sudo_pwd: Abc9876
vault_sl_admin_pwd: 12345678
```

If variable `sl_admin_pwd` is set, password changes made in the user
interface will be overwritten with the next execution of Ansible.

Now start the installation with

```bash
ansible-playbook sl_server.yml
```

### Tags

The available tags are, listed in the order they are executed:

- `sl-basics` installs the basic packages
- `sl-database` configures the access to the database
- `sl-latex` installs LaTeX, if *sl_latex* is true
- `sl-program` downloads SQL-Ledger from GitHub
- `sl-config` updates sql-ledger config
- `sl-webserver` configures the webserver, including password protection

## Update

Run the playbook again to update the installation. If you only want to
upgrade the code to a newer version, restrict execution to the
`sl-program` task:

```bash
ansible-playbook sql-ledger.yml -t sl-program
```

## Copyright and License

© 2016-2025 Tekki (Rolf Stöckli)

[GPL3](LICENSE)
