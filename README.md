What?
=====

There used to be a Makefile based CA script at sial.org but this has died when
the domain expired and the author is not responsive.

This is a port of that CA to a Rakefile with a few enhancements, there are
better CA management tools out there but this one is mine and I like it.

Usage?
======

Clone this repo into your new CA directory and optionally tweak the
_openssl.cnf.erb_ file to your liking, leave the erb bits in there since the
Rakefile will ask you a few questions on initial run.

Alternatively just create a _openssl.cnf_ in the directory and that will be
used without you being asked any questions.

Create a CA
-----------

This will create a new CA, the password it asks will be required to sign new
certificates in the future so DO NOT LOSE IT.

    $ rake init
    CA Common Name (CA): MyCA
    CA Country Name: GB
    CA State or Province: London
    Locality: London
    Email Address: me@my.com
    URL to the CRL: https://ca.my.com/ca-crl.pem
    >>> Creating directory crl
    >>> Creating directory newcerts
    >>> Creating directory private
    openssl req -config openssl.cnf -days 1825 -x509 -newkey rsa:2048 -out
    ca-cert.pem -outform PEM
    Generating a 2048 bit RSA private key
    .........................................................................+++
    .........+++
    writing new private key to './private/ca-key.pem'
    Enter PEM pass phrase:
    Verifying - Enter PEM pass phrase:
    -----

Signing a Certificate Signing Request
-------------------------------------

Now once you have a CA you probably want to sign some certs, if you already
have a CSR then just copy it into your CA directory:

    $ cp /tmp/mycert.csr .
    $ rake sign
    >>> Signing mycert.csr creating mycert.cert
    openssl ca -batch -config openssl.cnf -in mycert.csr -out mycert.cert
    Using configuration from openssl.cnf
    Enter pass phrase for ./private/ca-key.pem:
    Check that the request matches the signature
    Signature ok
    Certificate Details:
            Serial Number: 1 (0x1)
            Validity
                Not Before: May 24 15:32:45 2012 GMT
                Not After : May 24 15:32:45 2014 GMT
            Subject:
                countryName               = GB
                stateOrProvinceName       = London
                organizationName          = MyCo
                organizationalUnitName    = Webops
                commonName                = web1.myco.com
                emailAddress              = sysadmin@myco.com
            X509v3 extensions:
                X509v3 Basic Constraints:
                    CA:FALSE
                X509v3 Subject Key Identifier:
                    9B:34:38:57:B3:70:56:B5:D8:80:F8:5D:4F:24:9F:4B:9C:E3:4B:FD
                X509v3 Authority Key Identifier:
                    keyid:56:88:2C:9A:B3:3C:E8:71:A6:AD:B3:34:C8:9C:3B:C5:F9:81:22:BF
                    DirName:/CN=MyCA/C=GB/ST=London/L=London/emailAddress=me@my.com
                    serial:F9:18:15:E5:E1:8A:22:3C

                Netscape CA Revocation Url:
                    https://ca.my.com/ca-crl.pem
    Certificate is to be certified until May 24 15:32:45 2014 GMT (730 days)

    Write out database with 1 new entries
    Data Base Updated

A copy of the certificate will be stored in _newcerts_

Creating a CSR and Key
----------------------

If you dont have a CSR or key already you can create one:

    % rake gencsr CERT=mycert
    openssl req -out mycert.csr -new -newkey rsa:2048 -keyout mycert.key
    Generating a 2048 bit RSA private key
    ...........................................................+++
    ..................+++
    writing new private key to 'mycert.key'
    Enter PEM pass phrase:
    Verifying - Enter PEM pass phrase:
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [XX]:GB
    State or Province Name (full name) []:London
    Locality Name (eg, city) [Default City]:London
    Organization Name (eg, company) [Default Company Ltd]:MyCo
    Organizational Unit Name (eg, section) []:
    Common Name (eg, your name or your server's hostname) []:web1.my.com
    Email Address []:sysadmin@my.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

This will leave _mycert.key_ and _mycert.csr_ in the current directory
protected with the password you provided, you can now use _rake sign_ to sign
this certificate

Revoking a certificate
----------------------
When retiring systems you need to revoke their old certificates, to revoke a
certificate you need a copy of the certificate.

    % rake revoke CERT=newcerts/01.pem
    >>> Revoking certificate newcerts/01.pem
    openssl ca -config openssl.cnf -revoke 'newcerts/01.pem'
    Using configuration from openssl.cnf
    Enter pass phrase for ./private/ca-key.pem:
    Revoking Certificate 01.
    Data Base Updated
    openssl ca -config openssl.cnf -gencrl -out ca-crl.pem
    Using configuration from openssl.cnf
    Enter pass phrase for ./private/ca-key.pem:

As mentioned all the new certs are stored in *newcerts* so you can find any
cert the CA signed there but you can fetch a copy of the cert from anywhere

The password you need is the CA password, this will also update the
certificate revokation list which you can do manually using _rake gencrl_

Destroying your CA
------------------

If you do not need the CA anymore you can destroy it:

    % rake destroy_ca
    Type 'yes' to destroy the CA: yes
    %

Contact?
========

R.I.Pienaar / rip@devco.net / @ripienaar / http://devco.net/
