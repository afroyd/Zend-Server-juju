# Overview

This charm deploys Zend Server, The integrated application platform for mobile and web apps.

What Zend Server Does:
- Helps developers write better code and solve issues long before their code reaches production (‘Left Shifting’)
- Automates all steps in the release of PHP applications, from code to production, including provisioning, versioning and rollback
- Optimizes performance through dynamic scaling, code acceleration, code and data caching, job queuing and more
- Helps to detect and fix production issues in applications, faster and with less disruption

# Usage

## General Usage

To deploy a Zend Server service:


By default it will install Zend server . with Z-Ray disabled which is good for production. if you'd like to use it for development set the "zrayEnable:" config to true.

Generate a gui admin api key hash. Should be 64 characters long string. You can generate it in bash with:

    date +%s | sha256sum | base64 | head -c 64


Deploy the charm:

    juju deploy zend-server

Set the admin webapi key and gui password:

    juju set zend-server guiPassword=yourSecretPassword adminKeyHash=yourGenerated64CharactersLongApiKey

Enable Z-Ray (for development only. to use Z-ray in production enable secured mode in Zend Server's gui):
	
    juju set zend-server zrayEnable=true

To expose it to the internet:

    juju expose zend-server

This will open port 80 (http) and 443 (https) for web traffic.
It will also open the Zend server gui ports 10081 (http) and 10082 (https)

Open the Admin gui at http://\<Server_IP\>:10081/ZendServer and manage the PHP configuration, deploy more apps and see statistics and events from your site. 

## adding MySQL and cluster relation
    juju deploy mysql
    juju add-relation zend-server mysql 

This relation will move Zend Server's database from the local machine to MySQL and prepare it to function in a cluster

## Using Zend Server in a cluster with HAProxy:

    juju deploy haproxy
    juju add-relation zend-server:website haproxy:reverseproxy 

After it is added to HAProxy you can add more Zend Server units with:

    juju add-unit  zend-server
The next Zend server units will get all the apps and configuration from the other nodes. Every change in the PHP/Zend Server configuration done from the admin ui of a specific unit will be propagated to all the units.

## Note about config changes:

There is no way to change the webapi key after installation so changing the value will not affect the actual key.
In cluster mode it is recommended not to change the Zend Server configs such as enable zray and password as each node will change the value in the MySQL. The best way is to log into the Zend Server admin gui on port 10081 or 10082 on one of the units and change. This will change it for all the units.
