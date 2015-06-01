# Introduction #

The installation process is pretty straight forward:
  1. Install [Drupal](http://drupal.org) 7.x (or 6.x)
  1. Install [simpleSAMLphp](http://simplesamlphp.org)
  1. Configure SimpleSAMLphp to use something other than "phpsession" for session storage, e.g., SQL or memcache (See: store.type in simplesamlphp/config/config.php).
  1. Download [drupalauth](http://code.google.com/p/drupalauth/)
  1. Unpack drupalauth
  1. Move the drupalauth module directory into simplesamlphp/modules directory
  1. Configure the authentication source in simplesamlphp/config/authsources.php ('drupalauth:External' for approach #1 and 'drupalauth:UserPass' for approach #2)
  1. (Approach #1) Install the drupal4ssp module in Drupal


# Two Approaches #

## 1) Authenticate against Drupal but use the Drupal login page ##

The advantage of this approach is that the SimpleSAMLphp IdP session is tied to a Drupal session. This allows the user who is already logged into the Drupal site to then navigate to a SAML SP that uses the IdP without the need to authenticate again.

### Details ###

Configure the authentication source by putting something like this into simplesamlphp/config/authsources.php

```
'drupal-userpass' => array(
     'drupalauth:External',
 
     // The filesystem path of the Drupal directory.
     'drupalroot' => '/var/www/drupal',
 
     // Whether to turn on debug
     'debug' => true,
 
     // the URL of the Drupal logout page
     'drupal_logout_url' => 'https://www.example.com/drupal/user/logout',
 
     // the URL of the Drupal login page
     'drupal_login_url' => 'https://www.example.com/drupal/user',
 
     // Which attributes should be retrieved from the Drupal site.
 
               'attributes' => array(
                                     array('drupaluservar'   => 'uid',  'callit' => 'uid'),
                                      array('drupaluservar' => 'name', 'callit' => 'cn'),
                                      array('drupaluservar' => 'mail', 'callit' => 'mail'),
                                      array('drupaluservar' => 'field_first_name',  'callit' => 'givenName'),
                                      array('drupaluservar' => 'field_last_name',   'callit' => 'sn'),
                                      array('drupaluservar' => 'field_organization','callit' => 'ou'),
                                      array('drupaluservar' => 'roles','callit' => 'roles'),
                                    ),
),
```

## 2) Authenticate against Drupal but use the SimpleSAMLphp login page ##

The advantage of this approach is that their is no obvious connection between SimpleSAMLphp IdP and the Drupal site.

### Details ###

Put something like this into simplesamlphp/config/authsources.php

```
'drupal-userpass' => array(
        'drupalauth:UserPass',

        // The filesystem path of the Drupal directory.
        'drupalroot' => '/home/drupal',            

        // Whether to turn on debug
        'debug' => true,

        // Which attributes should be retrieved from the Drupal site.
        // This can be an associate array of attribute names, or NULL, in which case
        // all attributes are fetched.
        //
        // If you want everything (except) the password hash do this:
        //      'attributes' => NULL,
        //
        // If you want to pick and choose do it like this:
        //'attributes' => array(
        //                    array('drupaluservar'   => 'uid',  'callit' => 'uid'),
        //                      array('drupaluservar' => 'name', 'callit' => 'cn'),
        //                      array('drupaluservar' => 'mail', 'callit' => 'mail'),
        //                      array('drupaluservar' => 'field_first_name',  'callit' => 'givenName'),
        //                      array('drupaluservar' => 'field_last_name',   'callit' => 'sn'),
        //                      array('drupaluservar' => 'field_organization','callit' => 'ou'),
        //                      array('drupaluservar' => 'roles','callit' => 'roles'),
        //                     ),
        //
        // The value for 'drupaluservar' is the variable name for the attribute in the
        // Drupal user object.
        //
        // The value for 'callit' is the name you want the attribute to have when it's
        // returned after authentication. You can use the same value in both or you can
        // customize by putting something different in for 'callit'. For an example,
        // look at uid and name above.
        'attributes' => array(
                              array('drupaluservar' => 'uid',  'callit' => 'uid'),
                              array('drupaluservar' => 'name', 'callit' => 'cn'),
                              array('drupaluservar' => 'mail', 'callit' => 'mail'),
                              array('drupaluservar' => 'field_first_name','callit' => 'givenName'),
                              array('drupaluservar' => 'field_last_name','callit' => 'sn'),
                              array('drupaluservar' => 'field_organization','callit' => 'ou'),
                              array('drupaluservar' => 'roles','callit' => 'roles'),
                             ),
),
```

# Notes #

You must configure "store.type" in simplesamlphp/config/config.php to be something other than "phpsession", or this module may not work. "SQL" and "memcache" work just fine. The tell tail sign of the problem is infinite browser redirection when the SimpleSAMLphp login page should be presented.