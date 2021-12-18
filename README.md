# Ansible Role for SQL-Ledger

This role allows you to install [SQL-Ledger](https://github.com/Tekki/sql-ledger) on Debian Stretch.

Clone it to your Ansible roles directory:

    git clone https://github.com/Tekki/ansible-sql-ledger.git path_to_ansible/roles/sql-ledger

## Prerequisites

The machine from which the playbook is being run needs to have Ansible 2.0
or higher installed. For detailed information how to obtain current packages for your
distribution of choice have a look at the
[Ansible documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

The target machine needs SSH access and Python and sudo installed. As standard Debian
comes without sudo, you should install it and update your host configuration:

    ansible_become: true
    ansible_become_pass: "{{ vault_sudo_pwd }}"

## Role Variables

The following variables can be passed to this role:

| Variable Name | Default Value | Description |
| ------------- | ------------- | ----------- |
| sl\_admin\_pwd | *undefined* | password for admin.pl |
| sl\_dvipdf | false | use dvipdf instead of pdflatex |
| sl\_git\_branch | full | branch that will be checked out |
| sl\_git\_source | https://github.com/Tekki/sql-ledger.git | URL of the Git repository |
| sl\_helpful\_login | false | helpful error messages on login screen |
| sl\_httpd\_config | /etc/apache2 | path to the webserver config |
| sl\_httpd\_path | /var/www/sql-ledger | local path of the installation |
| sl\_httpd\_url | sql-ledger | browser URL on the server |
| sl\_latex | true | install and use LaTeX |
| sl\_login\_language | | language of the login screen |
| sl\_pdftk | true | use pdftk to combine PDFs |
| sl\_postgres\_user | sql-ledger | user name to connect to PostgreSQL |
| sl\_protect\_admin | false | protect admin interface |
| sl\_protect\_password | | password for protected admin interface |
| sl\_protect\_username | | username for protected admin interface |
| sl\_sendmail | "\| /usr/sbin/sendmail -f <%from%> -t" | pipe to sendmail |
| sl\_xelatex | false | use XeLaTex instead of pdflatex |
| texlive\_lang | german | language of TeX Live that will be installed |

Please notice that this role doesn't install any mail transport agent.

## Example Playbook

To install with the default settings and language German/Switzerland (chd\_utf) on the login screen, use the following playbook:

    - hosts: sql-ledger-servers
      roles:
         - { role: sql-ledger, sl_login_language: chd_utf }

If you want to set the password for admin.pl to '12345678' (probably not a good idea), write:

    - hosts: sql-ledger-servers
      roles:
         - { role: sql-ledger, sl_admin_pwd: 12345678 }

Use the [Vault](http://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html) to protect your
passwords!

To make the original version from DWS available under /sl-dws, write:

    - hosts: sql-ledger-servers
      roles:
        - { role: sql-ledger, sl_httpd_path: /var/www/sl-dws, sl_httpd_url: sl-dws, sl_git_branch: master }

It is possible to install the role multiple times under different URLs on the same server.

If your hosts are defined in *sql-ledger.hosts* in this way:

    [sql-ledger-servers]
    sl-server-[01:05]

and the name of the playbook is *sql-ledger.yml*, you can start the installation with:

    ansible-playbook -i sql-ledger.hosts sql-ledger.yml

Be aware that the installation of texlive needs a lot of time.

### Tags

The available tags are:

- `sl-basics` installs the basic packages
- `sl-config` updates sql-ledger and wlprinter config
- `sl-database` configures the access to the database
- `sl-git` downloads SQL-Ledger from GitHub
- `sl-latex` installs LaTeX, if *sl_latex* is true
- `sl-webserver` configures the webserver, including password protection

Update
------

To update the installation, simply run the playbook again. If you just want
to download a new version:

    ansible-playbook -t sl-git sql-ledger.yml

## License

[GPL3](LICENSE)
