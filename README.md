# Keycloak Radius Integration
[![CircleCI](https://circleci.com/gh/vzakharchenko/keycloak-radius-plugin/tree/master.svg?style=svg)](https://circleci.com/gh/vzakharchenko/keycloak-radius-plugin/tree/master)  
[![Coverage Status](https://coveralls.io/repos/github/vzakharchenko/keycloak-radius-plugin/badge.svg?branch=master)](https://coveralls.io/github/vzakharchenko/keycloak-radius-plugin?branch=master)  
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.vzakharchenko/keycloak-plugins/badge.svg)]  
<a href="https://codeclimate.com/github/vzakharchenko/keycloak-radius-plugin/maintainability"><img src="https://api.codeclimate.com/v1/badges/499d56ae9242cfaf2cbb/maintainability" /></a>

Run radius server inside keycloak.  
features:
- radius server inside keycloak
- [radius oidc password](Examples/OneTimePasswordJSExample)
- [radius OTP password (TOTP/HOTP via Google Authenticator or FreeOTP)](Examples/OTPPasswordJSExample)
- use Keycloak user password, if radius access-request protocol is PAP. Otherwise is using radius-password credential or OTP
- use Kerberos credential(only if Radius client use PAP authorization)
- can work as [radius proxy](#radius-proxy)
- support [Radsec Protocol](keycloak-plugins/rad-sec-plugin/README.md#radsec-example) (Radius over TLS)
- Map Keycloak [Role](#assign-radius-attributes-to-role), [Group](#assign-radius-attributes-to-group) and [User](#assign-radius-attributes-to-user) Attributes to Radius Attributes
- start/stop Keycloak Session ![sessionManagment](docs/sessionManagment.png)
- BackChannel logout(Disconnect-message request)
- [Mikrotik plugin](keycloak-plugins/mikrotik-radius-plugin)
- [Social Hotspot Login](hotspot)

## Release Setup
1. Download  keycloak-radius.zip asset from [github releases](https://github.com/vzakharchenko/keycloak-radius-plugin/releases)
2. unzip release <pre><code>unzip keycloak-radius.zip -d keycloak-radius</pre></code>
3. run keycloak  <pre><code>sh keycloak-radius/bin/standalone.sh  -c standalone.xml -b 0.0.0.0 -Djboss.bind.address.management=0.0.0.0 --debug 8190 -Djboss.http.port=8090</pre></code>
4. open http://localhost:8090
5. initialize keycloak master realm
## Manual Setup
### build project
***requirements***: java jdk 11 and above, maven 3.5 and above
 - <pre><code>cd keycloak-plugins</pre></code>
 - <pre><code>mvn clean install</pre></code>
### Configure keycloak
***requirements***: [keycloak 8.0.1](https://downloads.jboss.org/keycloak/8.0.1/keycloak-8.0.1.zip) (I think it should work on earlier versions)
- setup radius-plugin <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --command=module add --name=keycloak.plugins.radius --resources=${SOURCE}/keycloak-plugins/radius-plugin/target/radius-plugin-1.0-SNAPSHOT.jar --dependencies=org.jboss.logging,org.keycloak.keycloak-core,org.keycloak.keycloak-services,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.apache.commons.io,javax.activation.api,javax.servlet.api,org.jboss.resteasy.resteasy-jaxrs,javax.ws.rs.api,com.fasterxml.jackson.core.jackson-databind,org.keycloak.keycloak-common,com.fasterxml.jackson.core.jackson-core,javax.transaction.api,org.hibernate,io.netty,org.slf4j,javax.xml.bind.api,org.apache.commons.codec</pre></code>
- setup rad-sec plugin <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --command=module add --name=keycloak.plugins.rad.sec --resources=${SOURCE}/keycloak-plugins/rad-sec-plugin/target/rad-sec-plugin-1.0-SNAPSHOT.jar --dependencies=org.jboss.logging,org.keycloak.keycloak-core,org.keycloak.keycloak-services,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.apache.commons.io,javax.activation.api,com.fasterxml.jackson.core.jackson-databind,org.keycloak.keycloak-common,com.fasterxml.jackson.core.jackson-core,javax.transaction.api,org.hibernate,io.netty,org.slf4j,javax.xml.bind.api,org.apache.commons.codec,keycloak.plugins.radius</pre></code>
- setup mikrotik plugin <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --command=module add --name=keycloak.plugins.radius.mikrotik --resources=${SOURCE}/keycloak-plugins/mikrotik-radius-plugin/target/mikrotik-radius-plugin-1.0-SNAPSHOT.jar --dependencies=org.jboss.logging,org.keycloak.keycloak-core,org.keycloak.keycloak-services,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.apache.commons.io,javax.activation.api,com.fasterxml.jackson.core.jackson-databind,org.keycloak.keycloak-common,com.fasterxml.jackson.core.jackson-core,javax.transaction.api,org.hibernate,io.netty,org.slf4j,javax.xml.bind.api,org.apache.commons.codec,keycloak.plugins.radius</pre></code>
- setup radius-disconnect plugin <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --command=module add --name=keycloak.plugins.radius.dm --resources=${SOURCE}/keycloak-plugins/radius-disconnect-plugin/target/radius-disconnect-plugin-1.0.1-SNAPSHOT.jar --dependencies=org.jboss.logging,org.keycloak.keycloak-core,org.keycloak.keycloak-services,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.apache.commons.io,javax.activation.api,com.fasterxml.jackson.core.jackson-databind,org.keycloak.keycloak-common,com.fasterxml.jackson.core.jackson-core,javax.transaction.api,org.hibernate,io.netty,org.slf4j,javax.xml.bind.api,org.apache.commons.codec,keycloak.plugins.radius,keycloak-model-jpa,javax.persistence.api,org.hibernate</pre></code>
- setup proxy-radius plugin <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --command=module add --name=keycloak.plugins.radius.proxy --resources=${SOURCE}/keycloak-plugins/proxy-radius-plugin/target/proxy-radius-plugin-1.0.1-SNAPSHOT.jar --dependencies=org.jboss.logging,org.keycloak.keycloak-core,org.keycloak.keycloak-services,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-server-spi-private,org.apache.commons.io,javax.activation.api,com.fasterxml.jackson.core.jackson-databind,org.keycloak.keycloak-common,com.fasterxml.jackson.core.jackson-core,javax.transaction.api,org.hibernate,io.netty,org.slf4j,javax.xml.bind.api,org.apache.commons.codec,keycloak.plugins.radius</pre></code>
- run script for standalone <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --file=${SOURCE}/cli/radius.cli</pre></code>
- run script for standalone-ha <pre><code>${KEYCLOAK_PATH}/bin/jboss-cli.sh --file=${SOURCE}/cli/radius-ha.cli</pre></code>
where
- **KEYCLOAK_PATH** - Path where you are unpacked keycloak-8.0.1.zip
- **SOURCE** - Path where you checked out the code and built the project
## Configuration
### Radius server config file
-  create file ${KEYCLOAK_PATH}config/radius.config
-  example <pre><code>{
  "sharedSecret": "radsec",
  "authPort": 1812,
  "accountPort": 1813,
  "numberThreads": 8,
  "useUdpRadius": true,
  "radsec": {
    "privateKey": "config/private.key",
    "certificate": "config/public.crt",
    "numberThreads": 8,
    "useRadSec": true
  },
   "coa":{
      "port":3799,
      "useCoA":true
   }
}
</code></pre>
where

 -  **sharedSecret** - Used to secure communication between a RADIUS server and a RADIUS client.
   -  **authPort** - Authentication and authorization port
   -  **accountPort** - Accounting port
   -  **useUdpRadius** - if true, then listen to authPort and accountPort
   -  **radsec** - radsec configuration
   -  **privateKey** - private SSL key (https://netty.io/wiki/sslcontextbuilder-and-private-key.html)
   -  **certificate** - certificates chain
   -  **useRadSec** - if true, then listen  radsec port
   -  **numberThreads** - number of connection threads
   -  **coa** - CoA request configuration
   -  **port** - CoA port (Mikrotik:3799, Cisco:1700)
   -  **useCoA** - use CoA request
##
 Run Keycloak Locally
<pre><code>
#!/usr/bin/env bash
set -e
cd keycloak-8.0.1
public_ip=`ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1' -m 1`
sh bin/standalone.sh  -c standalone-ha.xml -b 0.0.0.0 -Djboss.bind.address.management=0.0.0.0 -Djboss.bind.address.private=${public_ip} -Djboss.node.name=${public_ip} -Djgroups.bind.address=${public_ip} --debug 8190 -Djboss.http.port=8090
</pre></code>

### Keycloak Client with Radius Protocol
![radiusProtocol](docs/radiusProtocol.png)

### Assign Radius Attributes to Role
> **_NOTE:_**  Composite roles supported

![RoleAttributes](docs/RoleAttributes.png)
### Assign Radius Attributes to Group
> **_NOTE:_**  SubGroups supported
![groupAttributes](docs/groupAttributes.png)
### Assign Radius Attributes to User
![userAttributes](docs/userAttributes.png)

###  Hotspot Example (with Facebook login)

[Hotspot Example (with Facebook login)](hotspot)

### Example CoA Configuration
[Radius Disconnect Message](keycloak-plugins/radius-disconnect-plugin)

### Radius Proxy

[Radius Proxy Module](keycloak-plugins/proxy-radius-plugin)
