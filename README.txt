This is a plugin that makes Moodle an Identity Provider site: other application can use Moodle as a login portal

Requires SimpleSAMLphp, configured as IdP: https://simplesamlphp.org/docs/stable/simplesamlphp-idp

ATTENTION: SimpleSAMLphp's session store (config/config.php, 'store.type') MUST BE "sql" - "phpsession" will not work, "memcache" is not tested
ATTENTION: SimpleSAMLphp's baseurlpath (config/config.php, 'baseurlpath') MUST BE in the full URL format


This instruction describe situation, when we have two Moodle instances and use one of them as Identity Provider (IdP) server and another as Service Provider (SP). In our example we have:
* samlsp.enovation.lan as Service Provider (SP)
* idp.enovation.lan as Identity Provider(IDP)

First of all you need to install Simple SAMLphp according to its instructions and locate it in the folder near IPD site installation directory.

Use standard Moodle instalation mechanism to install the plugin to auth/samlidp on idp.enovation.lan.

In SimpleSAMLphp config/authsources.php, add following to $config var:
	'moodle-userpass' => array(
		'moodle:External',
		'moodle_coderoot' => '/var/www/ticket/moodle314/www',
		'logout_url' => 'https://idp.enovation.lan/auth/samlidp/logout.php',	// plugin's logout page
		'login_url' => 'https://idp.enovation.lan/login/index.php',			// standard Moodle login page
		'cookie_name' => 'MoodleSAMLIDPSessionID',
	)

In SimpleSAMLphp, in metadata/saml20-idp-hosted.php, modify 'auth' with the name from previous step:
	'auth' => 'moodle-userpass'

Now lets config the plugin in Moodle:
Go to Site administration -> Plugins -> Authentication -> SAML Identity Provider and set two parameters:
SimpleSAMLphp installation directory: /opt/vhosts/idp.enovation.lan/www-simplesamlphp
Auth source: moodle-userpass - the string from two previous steps.

Lets configurate SP samlsp.enovation.lan. Download SAML2 plugin and install it (https://moodle.org/plugins/auth_saml2). Go to Site administration -> Plugins -> Authentication ->SAML2 and set parameters. Some of them are described below:
* IdP metadata xml - copy metadata from idp.enovation.lan/idp_simplesaml/saml2/idp/metadata.php?output=xhtml
* Idp label override - any string for display the link on logging page for using SAML IdP logging.
* IdP to Moodle mapping - it is better to map users by email. If you use user id it can cause conflicts between two systems.
* Data mapping - describe how to match data from idp.enovation.lan to samlsp.enovation.lan
* SP metadata - allow you to download SP metadata. Download it and convert to php format - use metada converter: http://idp.enovation.lan/idp_simplesaml/admin/metadata-converter.php. Converted code put in opt/vhosts/idp.enovation.lan/www-simplesamlphp/metadata/saml20-sp-remote.php

Now you can test the logging via SAML_IdP plugin. Create a new user or use existing idp.enovation.lan. Go to samlsp.enovation.lan login page and choose SAML2 option. Enter user name and password of user from idp.enovation.lan and login. After loggining the user will added do samlsp.enovation.lan. That's all.

KNOWN ISSUES
1. If a user logs out from Moodle, it will not log them out from their SP application. The logout process is one-directional, from the SP app to Moodle
