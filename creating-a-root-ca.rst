Certificate Authority Setup
===========================

This setup is a modified version of `this page on creating
intermediate certs
<https://jamielinux.com/docs/openssl-certificate-authority/introduction.html>`_
and `this blog post
<https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/>`_
with updated info from `this page
<https://www.phildev.net/ssl/creating_ca.html>`_.

Create the root CA
------------------

Create root certificate directory structure::

  mkdir /certs/root/ca
  cd /certs/root/ca
  mkdir certs crl csr newcerts private
  chmod 700 private
  touch index.txt

The ``index.txt`` file acts as a flat file database to keep track of
signed certificates.

Create the root config file
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Place the `openssl.root.cnf <https://github.com/luke-powers/misc/blob/master/documentation/openssl.root.cnf>`_ file into ``/certs/root/ca as
/certs/root/ca/openssl.cnf``. Be sure to rename the file to
openssl.cnf when you place it into the directory. See `this page
<https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html#prepare-the-configuration-file>`_
for an explanation for various sections.

Create the root key
~~~~~~~~~~~~~~~~~~~

::

   cd /certs/root/ca
   openssl genrsa -aes256 -out private/companyCA.key 4096
   chmod 400 private/companyCA.key

Create the root certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  cd /certs/root/ca
  openssl req \
    -batch \
    -config openssl.cnf \
    -key private/companyCA.key \
    -new \
    -x509 \
    -days 3700 \
    -sha256 \
    -extensions v3_ca \
    -out certs/companyCA.cert

This will ask for a passphrase. Use the passphrase for *root* that is
stored in lastpass.

Verify the root cert
~~~~~~~~~~~~~~~~~~~~

::

   openssl x509 -noout -text -in certs/companyCA.cert

See `this page <https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html#verify-the-root-certificate>`_ for an explanation of the output.

Create the Intermediate CA
--------------------------

Create the intermediate certificate directory structure::

  mkdir /certs/root/ca/intermediate
  cd /certs/root/ca/intermediate
  mkdir certs crl csr newcerts private ext
  chmod 700 private
  touch index.txt
  echo 1000 > /certs/root/ca/intermediate/crlnumber

``crlnumber`` is used to keep track of certificate revocation lists.

Create the intermediate config file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Place the `openssl.intermediate.cnf <https://github.com/luke-powers/misc/blob/master/documentation/openssl.intermediate.cnf>`_ file into ``/certs/root/ca/intermediate`` as ``/certs/root/ca/intermediate/openssl.cnf``.

Be sure to rename the file to ``openssl.cnf`` when you place it into
the directory.

Create the intermediate key
~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl genrsa -aes256 -out intermediate/private/companyICA.key 4096
  chmod 400 intermediate/private/companyICA.key

This will ask you for a passphrase. Use the passphrase for the
*intermediate* key that is stored in lastpass.

Create the Intermediate Certificate Signing Request (CSR)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl req \
    -batch \
    -config intermediate/openssl.cnf \
    -new \
    -sha256 \
    -key intermediate/private/companyICA.key \
    -out intermediate/csr/companyICA.csr

This will ask for the *intermediate* passphrase which is stored in
lastpass.

Create the Intermediate Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

   cd /certs/root/ca
   openssl x509 \
     -req \
     -days 375 \
     -sha256 \
     -in csr/misoroboticsICA.csr \
     -out intermediate/certs/companyICA.cert \
     -extfile ext/companyICA.ext \
     -CA certs/companyCA.cert \
     -CAkey private/companyCA.key \
     -CAcreateserial

This will ask for the *root* passphrase which is stored in lastpass.

Verify the Intermediate Certificate against the Root Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl verify -CAfile certs/companyCA.cert intermediate/certs/companyICA.cert

This will out put ``companyICA.cert: OK``

Deploy the Intermediate Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the cert that is used in the browser or other client app that
intends to implement tls/https.


Create the certificate chain file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is a file that allows an agent to verify that the certificate is
signed by the root certificate.::

  cd /certs/root/ca
  cat intermediate/certs/companyICA.cert \
      certs/companyCA.cert > intermediate/certs/company-inter-root-chain.cert
  chmod 444 intermediate/certs/company-inter-root-chain.cert

This will need to be installed along with the host cert (created
below) in the web application that intends to use tls/https.

Create the server certificate
-----------------------------

Create the server key
~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl genrsa -out intermediate/private/site-wide.key 2048
  chmod 400 intermediate/private/site-widekey

Create the server Certificate Signing Request (CSR)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl req \
    -batch \
    -config intermediate/openssl.cnf \
    -key intermediate/private/company.key \
    -new \
    -sha256 \
    -out intermediate/csr/company.csr

This wont ask for a passphrase as the server key was not created with
a passphrase (it is possible to create it with a passphrase by using
-aes256, but it greatly complicates the installation step).

Create the extension file for the particular server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This will require some customization based on server location within
the site-wide. Note that for simplicity, a ‘site-wide’ key and cert were
created for the top level name servers using the following extension::

  basicConstraints = CA:FALSE
  nsCertType = server
  nsComment = "OpenSSL Generated Server Certificate, Company Site-Wide"
  subjectKeyIdentifier = hash
  authorityKeyIdentifier = keyid,issuer:always
  keyUsage = critical, digitalSignature, keyEncipherment
  extendedKeyUsage = serverAuth
  subjectAltName = @alt_names

  [alt_names]
  DNS.1 = ns-primary.internal.company.com
  DNS.2 = ns-primary.company
  DNS.3 = ns-primary
  DNS.4 = ns1.subdomain.internal.company.com
  DNS.5 = ns1.subdomain.company
  DNS.6 = ns1.subdomain
  DNS.7 = ns1.dev.internal.company.com
  DNS.8 = ns1.dev.company
  DNS.9 = ns1.dev
  DNS.10 = ns1.service.internal.company.com
  DNS.11 = ns1.service.company
  DNS.12 = ns1.service

This allows the resulting cert to work if used with either
ns1.service.internal.company.com or the shorter name
ns1.service. Place the edited extension file into
``/certs/root/ca/intermediate/ext``. For example, the above extension
file is ``/certs/root/ca/intermediate/ext/site-wide.ext``.

Create the server cert
~~~~~~~~~~~~~~~~~~~~~~
::

  cd /certs/root/ca
  openssl x509 \
  -req \
  -days 375 \
  -sha256 \
  -in intermediate/csr/site-wide.csr \
  -out intermediate/certs/site-wide.cert \
  -extfile intermediate/ext/site-wide.ext \
  -CA intermediate/certs/companyICA.cert \
  -CAkey intermediate/private/companyICA.key \
  -CAcreateserial \
  chmod 444 intermediate/certs/site-wide.cert

This will ask for a set of information, just use the defaults from the
config file.

Verify the cert
~~~~~~~~~~~~~~~
::

 openssl x509 -noout -text -in intermediate/certs/site-wide.cert

The issuer should be the info for the intermediate CA. The subject
should be the info for the certificate itself.

Deploy the server cert
~~~~~~~~~~~~~~~~~~~~~~

Three files need to be installed into whichever webserver is being used

* company-inter-root-chain.cert
* site-wide.key
* site-wide.cert

Done.
