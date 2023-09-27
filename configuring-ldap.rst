LDAP Server
===========

Install needed libraries::

  sudo apt install slapd ldap-utils

Enter a password for the ``admin`` account when prompted, be sure to store it somewhere safe (a password manager).

Configure ``slapd`` to adjust the Directory Information Tree::

  sudo dpkg-reconfigure slapd

Omit OpenLDAP server configuration? (select No)

Enter the DNS domain name that is associated with the Directory Tree, in this case, ``your.example.com``

Enter in the desired Organization name associated with the Directory Tree (company name or other identifier)

Create the admin password, be sure to store it somewhere safe (a password manager).

Database backend to use: (select MDB)

Do you want the database to be removed when slapd is purged? (select No)

Do you want to move the old database? (select Yes)

Create the initial set of user info by creating a file ``ldap.ldif`` and enter the following::

  dn: ou=People,dc=your,dc=example,dc=com
  objectClass: organizationalUnit
  ou: People

  dn: ou=Groups,dc=your,dc=example,dc=com
  objectClass: organizationalUnit
  ou: Groups

  dn: uid=<USER>,ou=People,dc=your,dc=example,dc=com
  objectClass: inetOrgPerson
  objectClass: posixAccount
  objectClass: shadowAccount
  uid: <USER>
  sn: <LASTNAME>
  givenName: <FIRSTNAME>
  cn: <FULLNAME>
  displayName: <DISPLAYNAME>
  uidNumber: 10000
  gidNumber: 5000
  userPassword: <USER>
  gecos: <FULLNAME>
  loginShell: /bin/bash
  homeDirectory: <USERDIRECTORY>

Where

<USER>
  the user account on the linux system

<LASTNAME>
  the last name of the user

<FIRSTNAME>
  the first name of the user

<FULLNAME>
 the full name of the user

<DISPLAYNAME>
  the name to be displayed for the user

<USERDIRECTORY>
  the user's home directory on the Linux server

Once the ``ldap.ldif`` file is created properly, load it into the ldap database with::

  ldapadd -x -D cn=admin,dc=your,dc=example,dc=com -W -f ldap.ldif

It can be tested with::

  ldapsearch -x -LLL -b dc=your,dc=example,dc=com 'uid=<USER>' cn gidNumber

Finally (optionally) install `LAM <https://www.techrepublic.com/article/how-to-install-ldap-account-manager-on-ubuntu-18-04/>`_. It makes creating and editing users very easy.

LDAP Client Interactive
=======================

Install needed libraries::

  sudo apt-get install libnss-ldap libpam-ldap ldap-utils nscd -y

LDAP server URI for server created above (be careful not to use ldapi for this example)::

  ldap://your.example.com/

Distinguished name of the search base::

  dc=your,dc=example,dc=com

Specify LDAP version 3

Make local root Database admin (select Yes)

Does the LDAP database require login? (select No)

LDAP admin account::

  cn=admin,dc=your,dc=example,dc=com

Specify password for LDAP admin account (configured in LDAP server)

Edit ``/etc/nsswitch.conf`` and add ldap to match the following::

  passwd: compat systemd ldap
  group: compat systemd ldap
  shadow: files ldap

Edit ``/etc/pam.d/common-passwordand`` remove ``use_authtok`` from the following line::

  password [success=1 user_unknown=ignore default=die] pam_ldap.so use_authtok try_first_pass

Edit ``/etc/pam.d/common-session`` and add the following line::

  session optional pam_mkhomedir.so skel=/etc/skel umask=077

Create the file ``/etc/ldap.secret`` and put in it the ldap admin password from lastpass.
  
Restart the server::

  sudo /etc/init.d/nscd restart

LDAP Client Scripted
====================
::

  sudo DEBIAN_FRONTEND=noninteractive apt-get install libnss-ldap libpam-ldap ldap-utils nscd -y
  sudo sed -i -e "/base dc=example/c\base dc=your,dc=example,dc=com" -e "s+uri ldapi:///+uri ldap://10.191.4.2/+g" -e "/rootbinddn /c\rootbinddn cn=admin,dc=your,dc=example,dc=com" /etc/ldap.conf
  sudo sed -i -e '/^passwd: / s/$/ ldap/' -e '/^group: / s/$/ ldap/' -e '/^shadow: / s/$/ ldap/' /etc/nsswitch.conf
  sudo sed -i '/pam_ldap.so/s/ use_authtok//' /etc/pam.d/common-password
  sudo sed -i '/# end of pam/i session optional\tpam_mkhomedir.so\tskel=/etc/skel\tumask=077' /etc/pam.d/common-session
  sudo /etc/init.d/nscd restart
