# Ansible Role for SQL-Ledger

This role allows you to install the current version of
[SQL-Ledger](https://github.com/Tekki/sql-ledger) on Debian Trixie.

Clone the code to your Ansible roles directory:

```bash
git clone https://github.com/Tekki/ansible-sql-ledger.git $PATH_TO_ANSIBLE/roles/sql-ledger
```

## Prerequisites

The machine from which the playbook is run needs a recent Ansible
version. In most Linux distributions, it is available through the
official package repository. For detailed information how to obtain
current packages for your distribution of choice have a look at the
[Ansible documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

The target machine needs SSH access, Python and sudo. Standard Debian
comes with Python, but without sudo, so you have to install it and
update your host configuration accordingly:

```yaml
ansible_become: true
ansible_become_pass: "{{ vault_sudo_pwd }}"
```

## Role Variables

The following variables can be passed to this role:

| Variable Name                | Default Value                           | Description                                 |
|------------------------------|-----------------------------------------|---------------------------------------------|
| debian\_additional\_packages | []                                      | additional packages to be installed         |
| debian\_locale               | en_US                                   | default UTF-8 locale                        |
| sl\_admin\_pwd               | *undefined*                             | password for admin.pl                       |
| sl\_dvipdf                   | false                                   | use dvipdf instead of pdflatex              |
| sl\_git\_branch              | full                                    | branch that will be checked out             |
| sl\_git\_source              | https://github.com/Tekki/sql-ledger.git | URL of the Git repository                   |
| sl\_helpful\_login           | false                                   | helpful error messages on login screen      |
| sl\_httpd\_config            | /etc/apache2                            | path to the webserver config                |
| sl\_httpd\_path              | /var/www/sql-ledger                     | local path of the installation              |
| sl\_httpd\_url               | sql-ledger                              | browser URL on the server                   |
| sl\_latex                    | true                                    | install and use LaTeX                       |
| sl\_latex\_pdftk             | true                                    | use pdftk to combine PDFs                   |
| sl\_latex\_xelatex           | true                                    | use XeLaTeX instead of pdflatex             |
| sl\_login\_language          |                                         | language of the login screen                |
| sl\_postgres\_user           | sql-ledger                              | user name to connect to PostgreSQL          |
| sl\_protect\_admin           | false                                   | protect admin interface                     |
| sl\_protect\_password        |                                         | password for protected admin interface      |
| sl\_protect\_username        |                                         | username for protected admin interface      |
| sl\_sendmail                 | "\| /usr/sbin/sendmail -f <%from%> -t"  | pipe to sendmail                            |
| sl\_timezone                 | Europe/Zurich                           | time zone for server                        |
| texlive\_lang                | english                                 | language of TeX Live that will be installed |

Please notice that this role doesn't install a mail transport agent or printing
system.

## Example Playbook

To install with the default settings and language German/Switzerland (chd\_utf)
on the login screen, use the following playbook:

```yaml
- hosts: sql-ledger-servers
  roles:
    - name: sql-ledger
      debian_locale: de_CH
      sl_login_language: chd_utf
      texlive_lang: german
```

Or better move the variables to the host configuration file:

```yaml
ansible_become: true
ansible_become_pass: "{{ vault_sudo_pwd }}"

debian_locale: de_CH
sl_login_language: chd_utf
texlive_lang: german
```


If you want to set the password for admin.pl to '12345678' (probably
not a good idea), write:

```yaml
sl_admin_pwd: 12345678
```

Use the
[Vault](http://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html)
to protect your passwords!

To make the original version from DWS available under /sl-dws, write:

```yaml
sl_httpd_path: /var/www/sl-dws
sl_httpd_url: sl-dws
sl_git_branch: master
```

It is possible to install the role multiple times under different URLs
on the same server.

If your hosts are defined in *sql-ledger.hosts* in this way:

```conf
[sql-ledger-servers]
sl-server-[01:05]
```

and the name of the playbook is *sql-ledger.yml*, you can start the
installation with:

```bash
ansible-playbook -i sql-ledger.hosts sql-ledger.yml
```

Be aware that the installation of texlive needs a lot of time.

### Tags

The available tags are:

- `sl-basics` installs the basic packages
- `sl-config` updates sql-ledger and wlprinter config
- `sl-database` configures the access to the database
- `sl-git` downloads SQL-Ledger from GitHub
- `sl-latex` installs LaTeX, if *sl_latex* is true
- `sl-webserver` configures the webserver, including password protection

## Update

To update the installation, simply run the playbook again. If you just
want to download a new version:

```bash
ansible-playbook -t sl-git sql-ledger.yml
```

## Copyright and License

© 2016-2025 by Tekki (Rolf Stöckli)

[GPL3](LICENSE)
