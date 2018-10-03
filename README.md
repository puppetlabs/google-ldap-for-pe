# Google Secure LDAP for PE
This README documents the process and configuration needed to connect Puppet Enterprise to Google's Secure LDAP service as an external directory for managing RBAC. While this configuration has been validated there are two caveats, Puppet Enterprise only officially supports Active Directory or OpenLDAP and you'll be required to configure stunnel to handle encryption and authentication of your PE console services to Google's LDAP frontend due to Google's requirement for certificate based client authentication not currently supported in Puppet Enterprise for LDAP external directories.

## Manual method
1. [Setup and create a client certificate for the Google Secure LDAP service](https://support.google.com/cloudidentity/answer/9048516)
	* Begin specifically with the items indicated as 1, 2, 3, and 5 in the linked support article
	* After finishing the previous process Puppet Enterprise requires you to provision Access credentials
	
	##### Additional Secure LDAP setup
	1. Return to the LDAP app that lists the clients that you've provisioned and select the client you previously provisioned for the use with PE, in my example I named mine *Secure LDAP Docs*
	
	![Image of LDAP Admin Console](https://raw.githubusercontent.com/puppetlabs/google-ldap-for-pe/master/img/admin.png)
	
	2. This'll open the client's settings pane which should near the bottom have panel *Authentication* that'll ist *1 certificate* and *0 access credentials*; if this is true click on *Access Credentials*
	
	![Image of LDAP Clients Settings](https://raw.githubusercontent.com/puppetlabs/google-ldap-for-pe/master/img/client.png)
	
	3. Scroll down the new pane and click *GENERATE NEW CREDNTIALS* and a new random user name and password will be created
	
	![Image of LDAP Client Auth](https://raw.githubusercontent.com/puppetlabs/google-ldap-for-pe/master/img/auth.png)
	
	4. Save the credentials provided in the resulting popup pane, you won't be able to retrieve after dismissing the pane
	
	![Image of LDAP Access Cred](https://raw.githubusercontent.com/puppetlabs/google-ldap-for-pe/master/img/cred.png)
	
2. Setup [Puppet Enterprise](https://puppet.com/docs/pe/2018.1/installing.html) to your liking if you haven't already done so
3. [Setup stunnel](https://support.google.com/cloudidentity/answer/9098476#stunnel)

#### Installing stunnel on Ubuntu
 
* Package installation

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

4. [Configure Puppet Enterprise external directory](https://puppet.com/docs/pe/2018.1/rbac_ldap_intro.html#connecting-puppet-enterprise-with-external-directory-services)

* In the following example configuration you'll see a need for *Lookup user* and *Lookup password*, these were provisioned and provided to you in 

|*Name*|*Example Google Secure LDAP settings*|
|------|-------------------------------------|
|Directory name|Google Secure LDAP (example.com)|	
|Login help (optional)|https://example.com/docs/google-puppet-login|
|Hostname|127.0.0.1|
|Port|1636|
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
