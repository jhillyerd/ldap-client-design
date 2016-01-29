Portable LDAP Client Design
===========================

How do design your LDAP application so that it will be portable across directory
products.  This document was born out of my frustration with many custom and
off-the-shelf LDAP consuming applications that only support a single directory
product.

This work is licensed under a
[Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/)

Tips
----

Do not hard-code anything!  Hostnames, SSL settings, user names, DNs, and
attribute naming conventions will vary between directory products and
environments.

Terminology
-----------

*	CN (Common Name): Most often used as an RDN for a user or group object.
*	DN (Distinguished Name): The fully qualified name of an LDAP object.
*	LDAP (Lightweight Directory Access Protocol): Clearly a relative term, there
  is nothing lightweight about LDAP.
*	RDN (Relative Distinguished Name): The unqualified name of an LDAP object.

Configuration
-------------

Your application configuration should require the following attributes
(at minimum):

| Property Name     | Sample Value                       |
| ----------------- | ---------------------------------- |
| host              | ldapserver.mycorp.com              |
| port              | 636                                |
| use_ssl           | true                               |
| admin_dn          | cn=serviceacct,ou=Service,o=mycorp |
| admin_password    | t0p_Secret                         |
| user_class        | inetOrgPerson                      |
| user_name_attrib  | uid                                |
| user_mail_attrib  | mail                               |
| user_search_base  | ou=Users,o=mycorp                  |
| group_class       | groupOfUniqueNames                 |
| group_name_attrib | cn                                 |
| group_search_base | ou=Groups,o=mycorp                 |

Connecting
----------

### SSL

It is a best practice is to encrypt LDAP connections with SSL to protect user
passwords over the wire.  You may find it easier to develop your application
without SSL, so that can perform packet captures when debugging LDAP code.

### Binding

Authenticating to a server is called “binding” in LDAP terminology.  To handle
a login request, your application will typically bind first as an administrative
user (aka service account), then search for DN the user wishing to login.  If a
matching DN is found, you can then perform a second bind operation with the users
DN and the password they provided.

Searching
---------

### Users

A user search should start at the configured user_search_base as a full subtree
search.  If searching by username, the LDAP filter should be:

    (&(objectclass=${user_class})(${user_name_attrib}=<username>))

After variable expansion, this filter might look like:

    (&(objectclass=inetOrgPerson)(uid=jamesh))

If searching for a user by email address, use the following filter:

    (&(objectclass=${user_class})(${user_mail_attrib}=${email}))

Expanded example:

    (&(objectclass=inetOrgPerson)(mail=jamesh@mycorp.com))

### Groups

A group search should start at the configured group_search_base as a full
subtree search.  Your LDAP filter should be:

    (&(objectclass=${group_class})(${group_name_attrib}=<group name>))

After variable expansion, this filter might look like:

    (&(objectclass=groupOfUniqueNames)(cn=MyGroup))

