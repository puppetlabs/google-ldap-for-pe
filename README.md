# Google Secure LDAP for PE
This README documents the process and configuration needed to connect Puppet Enterprise to Google's Secure LDAP service as an external directory for managing RBAC. While this configuration has been validated there are two caveats, Puppet Enterprise only officially supports Active Directory or OpenLDAP and you'll be required to configure stunnel to handle encryption and authentication of your PE console services to Google's LDAP frontend due to Google's requirement for certificate based client authentication not currently supported in Puppet Enterprise for LDAP external directories.

### Manual method
* [Setup and create a client certificate for the Google Secure LDAP service](https://support.google.com/cloudidentity/answer/9048516)
	* This is specifically items indicated as 1 through 5 in the linked support article
* Setup [Puppet Enterprise](https://puppet.com/docs/pe/2018.1/installing.html) to your liking if you haven't already done so
* [Setup stunnel](https://support.google.com/cloudidentity/answer/9098476#stunnel)
	* Installing stunnel on Ubuntu

`apt install  stunnel4`

	* Create the configuration file */etc/stunnel/google-ldap.conf* with the following contents (change the ldap-client.key and ldap-client.cert to reflect the name of the certificate downloaded during Google Secure LDAP setup and client certificate creation)

```
[ldap]
client = yes
accept = 127.0.0.1:1636
connect = ldap.google.com:636
cert = /etc/stunnel/ldap-client.crt
key = /etc/stunnel/ldap-client.key
```
 
	* Upload LDAP client certificate and key obtained previously to the machine running your Puppet Enterprise console services and place them into the */etc/stunnel* directory

	* Enable stunnel, edit */etc/default/stunnel4* so that *ENABLED=1*

	* Start/restart stunnel

`systemctl restart stunnel4`

* [Configure Puppet Enterprise external directory](https://puppet.com/docs/pe/2018.1/rbac_ldap_intro.html#connecting-puppet-enterprise-with-external-directory-services)

|*Name*|*Example Google Secure LDAP settings*|
|Directory name|Google Secure LDAP (example.com)|	
|Login help (optional)|https://example.com/docs/google-puppet-login|
|Hostname|127.0.0.1|
|Port{1636|
|Lookup user (optional)|ExampleSecureLDAPUser|
|Lookup password (optional)|the_secure_ldap_provisioned_password|
|Connection timeout (seconds)|30|
|Connect using:|Plain text (insecure connection)|
|Validate the hostname?|No|
|Allow wildcards in SSL certificate?|No|
|Base distinguished name|dc=example,dc=com|
|User login attribute|uid|
|User email address|mail|
|User full name|displayName|
|User relative distinguished name (optional)|ou=Users|
|Group object class|groupOfNames|
|Group membership field|memberUid|
|Group name attribute|displayName|
|Group lookup attribute|cn|
|Group relative distinguished name (optional)|ou=Groups|
|Turn off LDAP_MATCHING_RULE_IN_CHAIN?|No|
|Search nested groups?|Yes|
