# LDAP for ProcessWire

This is a module for ProcessWire that enables user authentication over LDAP, so you are able to use your Active Directory users.

**Features**

- Native ProcessWire Sign In 
- Ability to specify a default domain name
- Ability to specity a default role for new LDAP users
- Passwords for LDAP users are automatically updated
- Settings and messages are translatable

**Login Page and Process**

[<img alt="Login Page" src="http://abload.de/img/ldap-login-page2bi6y.png" height="300px">](http://abload.de/img/ldap-login-page2bi6y.png) [<img alt="Login Process" src="http://abload.de/img/ldap-login-successfult8dg1.png" height="300px">](http://abload.de/img/ldap-login-successfult8dg1.png) 

**Module Settings**

[<img alt="Module Settings" src="http://abload.de/img/ldap-settings-english5nc2i.png" height="400px">](http://abload.de/img/ldap-settings-english5nc2i.png)

**Translated Module Settings**

[<img alt="Module Settings" src="http://abload.de/img/ldap-settings-germannzdop.png" height="400px">](http://abload.de/img/ldap-settings-germannzdop.png)


### Translation

This module ships with a translation into German. To install it, just upload the json file - which you can find under /translations - to your defined language.

### Changelog

**Version 0.5.1**

- Adds the option to connect to the LDAP server via SSL
- Adds Debug Mode, so the messages of PHP's LDAP extension are displayed in apache's log file
- The configuration is validated when saved

**Version 0.5.0**

Initial release