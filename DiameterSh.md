# Service Description #

In this example we will show some integration between Mobicents and OpenIMS, using the Diameter Sh interface to receive profile updates and SIP.

The idea is to provide a service that will let a user, who has been offline, know who called him while he was offline. To do that our service will be a proxy for SIP messages in the AS so he can capture the 404 sent by the CSCF when a call is made to him, while the user is offline.
It will also register on the HSS for receiving changes to the User-IMS-State, using the proper Diameter Sh message, Profile-Notification-Request.

When a call to a non-registered user happens, data is stored, regarding the person who called and the time of the call.  When the user becomes online (registers it's SIP phone), the HSS will fire a notification with the new user state, and by that time we will look into our map to find if the user has missed any call while offline.  If so, the user will receive a message, for each missed call, stating "UserXXX sip:xxx@open-ims.test tried to call you on dd/mm/yyyy at hh:mm:ss".

This flow is described in the [diagram](http://mobicents-public.googlegroups.com/web/Mobicents-Diameter-Sh-OpenIMS-SD.jpg) (SIP details and CSCF - HSS communication are not described for simplicity purposes)

# How to activate it #

## From the binary ##

Download the latest version of the war file corresponding to this example from [here](https://oss.sonatype.org/content/groups/public/org/mobicents/servlet/sip/examples/diameter-openims/)

Drop the downloaded war file into your tomcat\_home/webapps directory or jboss\_home/server/default/deploy directory
Get the [corresponding dar configuration file](http://sipservlets.googlecode.com/git/sip-servlets-examples/diameter-openims/diameteropenims-dar.properties).

To understand what the dar configuration file is used for, check the Application Router Documentation .

Drop it in your tomcat\_home/conf/dars directory or jboss\_home/server/default/conf/dars directory.

To use this dar file for this service, specify in the Service xml tag, darConfigurationFileLocation attribute of the tomcat\_home/conf/server.xml file or jboss\_home/server/default/deploy/jboss-web.deployer/server.xml , the following :
conf/dars/diameteropenims-dar.properties

You can now run your container (Tomcat or Jboss).

## From the source ##

Please check out the speed dial example located under [this location](http://code.google.com/p/sipservlets/source/browse/#git%2Fsip-servlets-examples%2Fdiameter-openims)
from the svn repository. Just call mvn clean install war:inplace to create the war file in the target directory

# Configuration #

There is a lot of configuration needed for this example and it's hard to setup so if you need help, drop us a request in [Mobicents Discussion](http://groups.google.com/group/mobicents-public/topics) and we should help as possible. Also read all the instructions here before setting up any of the components to ge a full grasp of the thing

## OpenIMS Core ##

First of all, you should install [OpenIMS Core](http://www.openimscore.org/) and set it up to use Mobicents SIP Servlets as the SIP Application Server.

It is a not-so-easy task, we advise to not tweak too much at the beginning and just change the default IP address 127.0.0.1 to one of your private network such as 192.168.0.10 ...

You can either install it from the source and follow the [OpenIMS Core Installation guide](http://www.openimscore.org/installation_guide) or use a VM ready to run [VMWare image](http://www.openimscore.org/vm).

To help you out a bit, we provide a bunch of example configuration files on setting up the whole thing on IP Address 192.168.0.10 in [the conf directory of the example](http://sipservlets.googlecode.com/git/sip-servlets-examples/diameter-openims/conf/) (from the source)

This conf directory is break down in the following sub directories (Please notice that each file in those directories as a comment saying where the file should be located in your file system ) :

  * dns : the list of DNS files set up to run open-ims.test domain and subdomains resolving to IP address 192.168.0.10. If you don't know how to setup a DNS, changes the IP address to map your own in those files and place them in the corresponding location on your file system and Don't forget to run sudo /etc/init.d/bind9 restart to apply the configuration
  * mobicents-jdiameter : this is not needed for the OpenIMS setup but needed later on to configure the Mobicents Diameter Solution
  * open-ims : this is the configuration files setup to run OpenIMS on IP address 192.168.0.10. If you use those files, use them when you have completed the Step 5 of the [OpenIMS Core Installation guide](http://www.openimscore.org/installation_guide)
  * open-ims/database : this is a dump of the OpenIMS Core database in a working state with Mobicents Sip Servlets setup to act as a SIP application Server to OpenIMS Core. It is recommended to use this dump instead of the ones provided in Step 4 of the [OpenIMS Core Installation guide](http://www.openimscore.org/installation_guide)
  * open-ic : there is only one file in there providing you with the values needed to setup the [OpenIC IMS client](http://www.open-ims.org/openic/)

Now you'll need to setup OpenIMS Core to use Mobicents Sip Servlets as a SIP Application Server, follow those simple steps (this is a more thorough explanation of what can be found in [OpenIMS FAQ](http://www.openimscore.org/node/83) ) :

  * start the OpenIMS Core HSS server : cd /opt/OpenIMSCore/FHoSS/deploy/ then sh startup.sh
  * log in into http://192.168.0.10:8080/ with username hssAdmin and password hss
  * Click on Services Tab
  * On the left hand side panel, under Application Servers Click on Create.
    * In name enter MSS
    * In Server Name , enter sip:192.168.0.10:5080
    * In Diameter FQDN , enter mobicents.open-ims.test
    * In Default Handling , select Session Terminated
      * Let Service Info empty and Rep-Data Limit to 1024
      * Check all the Sh Permissions
      * Don't attach any iFC for now, it will be done later automatically when you will create the given iFC
      * Click on Save
      * [Corresponding Screenshot](http://www.mobicents.org/mss-ims-as.png)

  * On the left hand side panel, under Trigger Points Click on Create.
    * In Name , enter MSS TP
    * In Conjunction Type CNF , select Disjunctive Normal Format , click on Save Button
    * On the next screen, Don't attach any iFC for now, it will be done later automatically when you will create the given iFC
    * Add SPTs to the Trigger point, SIP Method : INVITE AND Session Case : Origin - Session
    * Click on Save again
    * [Corresponding Screenshot](http://www.mobicents.org/mss-ims-tp.png)

  * On the left hand side panel, under Initial Filter Criteria Click on Create.
    * In Name , enter MSS iFC
    * In Trigger Point , select MSS TP
    * In Application Server , select MSS
    * In Profile Part Indicator , select Any
    * Click on Save Button
    * [Corresponding Screenshot](http://www.mobicents.org/mss-ims-ifc.png)

  * On the left hand side panel, under Service Profiles Click on Search link then left the input text field blanks and click on Search Button, then select default\_sp .
    * Detach the current iFC default\_ifc
    * Attach the MSS iFC MSS iFC
    * Click on Save Button
    * [Corresponding Screenshot](http://www.mobicents.org/mss-ims-sp.png)

Alternatively, if you use the sql database dumps in open-ims/database instead of the one provided by OpenIMS Core, you'll still need to modify the IP Address of the SIP Application Server in the FHoSS Web interface as explained above in Application Server section except that instead of creating it you need to search for an existing one with the name MSS

Start all open ims core elements :
  * CSCFs : Start pcscf.sh, icscf.sh and scscf.sh
  * HSS: Start FHoSS/deploy/startup.sh

## OpenIC Lite - IMS Client ##

Download and Install [Free OpenIC Lite IMS client](http://www.open-ims.org/openic/), and modify the OpenIC\_Lite.sh to set the JAVA\_HOME variable to match your JDK installation.

Then start it with sh OpenIC\_Lite.sh . To register Alice with OpenIMS Core (Alice is one of the already configured user in OpenIMS Core, so it's better to use it or Bob for starters), here is the information you need to enter :

  * Public identity: sip:alice@open-ims.test
  * Private identity: alice@open-ims.test
  * Secret Key: alice
  * Realm: open-ims.test
  * Proxy Server: 192.168.0.10:4060 (modify the IP address to match your own where Open IMS Core PCSCF component is running)

Then register and cross your fingers so that it goes through all the way. Registration uses all components and as such, it is a good test if all is up & running

## Mobicents Diameter Solution ##

Install Mobicents Diameter or make sure it is already installed (it comes prepackaged with the Mobicents Sip Servlets JBoss Version)
You have one file to modify so that it can connect to OpenIMS Core. In JBOSS\_HOME/server/default/deploy/mobicents-diameter-mux-1.0.0.BETA3.sar/lib , you need to modify the org/mobicents/diameter/stack/jdiameter-config.xml file in the mobicents-diameter-mux-jar-1.0.0.BETA3.jar archive

Modify it with the file located at http://sipservlets.googlecode.com/git/sip-servlets-examples/diameter-openims/conf/mobicents-jdiameter/jdiameter-config.xml (Make sure you modify all the 192.168.0.10 IP addresses in that file to map to your own IP address)

## Mobicents Sip Servlets ##

Since OpenIMS Core HSS component is deploying a management web application on port 8080, you'll need to modify the http connector port in Mobicents Sip Servlets to avoid any conflicts.

So go edit JBOSS\_HOME/server/default/deploy/jboss-web.deployer/server.xml file.

Look for the connector tag whose the port is 8080 and change it to 8081 or any other value that makes sense for you.

## Example ##

A configuration file can be found for this example at src/main/sipapp/META-INF/diameter-openims.properties for the source, or at the root inside diameter-openims-version .war. Parameters are:

  * origin.ip - The IP address of the MSS server. Used for creating ORIGIN\_HOST AVP;
  * origin.port - The Diameter listening port of the MSS server Diameter Mux. Used for creating ORIGIN\_HOST AVP;
  * origin.realm - The Diameter Realm of the MSS server;
  * origin.host - If defined, will override generation of ORIGIN\_HOST AVP for Diameter Messages (recommended for this example!);
  * destination.ip - The IP address of the OpenIMS HSS. Used for creating DESTINATION\_HOST AVP;
  * destination.port - The listening port of the OpenIMS HSS. Used for creating DESTINATION\_HOST AVP;
  * destination.realm - The Diameter Realm of the OpenIMS HSS;
  * destination.host - If defined, will override generation of DESTINATION\_HOST AVP for Diameter Messages;
  * users - The users to subscribe Profile notifications (separate by comma)

It is usually only needed to modify origin.ip to match to your own IP address. and destination.ip to match to the IP address where the HSS is installed (if you installed it on the same machine as Mobicents Sip Servlets the IP address for those properties will be the same)

Then do a mvn clean install>> and deploy the war to <<<JBOSS\_HOME/server/default/deploy .

Get the corresponding [dar configuration file](http://sipservlets.googlecode.com/git/sip-servlets-examples/diameter-openims/diameteropenims-dar.properties).

Drop it in your jboss\_home/server/default/conf/dars directory.

To use this dar file for this service, specify in the Service xml tag, darConfigurationFileLocation attribute of the jboss\_home/server/default/deploy/jboss-web.deployer/server.xml , the following :
conf/dars/diameteropenims-dar.properties

# How to play with it #

Start JBoss on the binding address of the same IP address you configured elsewhere, by example sh run.sh -b 192.168.0.10 , and make sure no exception pop up during startup because that could mean that Diameter cannot connect to the OpenIMS Core HSS server

Start one OpenIC Lite with some account (doesn't have to be registered, but it's OK if it is).

Make a call to one of the other user configured for this service (alice@open-ims.test or bob@open-ims.test by default), that is NOT registered.
CSCF will answer with a 404 NOT FOUND and the example will use that info for collecting a missed call.

Now, register the called user (you can do it with the same SIP phone or a different one), and by that time the application should receive a Diameter Push-Notification-Request and will be aware of the user new state (REGISTERED). After that it will retrieve the missed calls data and send him SIP Message(s) with that information. So when the called user is fully registered, it will receive a message stating that "User XXX tried to call at dd/mm/yyyy at hh:mm:ss".