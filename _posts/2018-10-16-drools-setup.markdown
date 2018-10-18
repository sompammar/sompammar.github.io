---
layout: post
title:  "How to set up drools kie server and kie workbench for development using wildfly"
date:   2018-10-16 18:11:34 -0700
categories: drools kie-server-workbench setup
---

## Purpose

This link helps you quickly setup drools kie server and workbench and configure them. This uses [drools](https://www.drools.org/download/download.html) version [7.12.0.Final](http://download.jboss.org/drools/release/7.12.0.Final/). This link focuses on setting up drools on wildfly. There is a separate link regarding set up instructions for tomcat. These instructions are tailored for running it from unix like machines.

## Pre-requisites

- Download and extract [wildfly 11](http://download.jboss.org/wildfly/11.0.0.Final/wildfly-11.0.0.Final.zip) as this is the version of wildfly currently supported by drools 7.12.0.Final.

- Download and install jdk latest version 1.8 or above

## Download and copy the workbench and kie-server files

[Download kie-workbench](https://download.jboss.org/drools/release/7.12.0.Final/kie-drools-wb-7.12.0.Final-wildfly11.war). 

Copy the war file ```kie-drools-wb-7.12.0.Final-wildfly11.war``` to ```{wildfly}/standalone/deployments```

[Download and extract the drools engine distribution file](https://download.jboss.org/drools/release/7.12.0.Final/drools-distribution-7.12.0.Final.zip) 

Copy the file ```kie-server-7.12.0.Final-ee7.war``` from extracted directory to ```{wildfly}/standalone/deployments```


## Set up wildfly server

- Create user kieserver by running the following command from ```{wildfly}/bin```

```
./add-user.sh -a -u kieserver -p kieserver -ro admin,kie-server,rest-all,kiemgmt
```

- Locate and edit ```standalone.conf``` (or ```standalone.conf.bat``` for windows based systems) file under ```{wildfly}/bin``` directory
- Increase the java heap size and MaxMetaspaceSize to suit your needs. 

    ```script
       JAVA_OPTS="-Xms1024m -Xmx1024m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=1024m -Djava.net.preferIPv4Stack=true"
    ```
- Create a script e.g. ```start-drools.sh``` under ```{wildfly}/bin``` and copy the following into this file. 

```script
 SERVER_CONFIG="--server-config=standalone-full.xml"
 KIE_SERVER_USER="-Dorg.kie.server.user=kieserver"
 KIE_SERVER_PWD="-Dorg.kie.server.pwd=kieserver"
 KIE_CONTROLLER_USER="-Dorg.kie.server.controller.user=kieserver"
 KIE_CONTROLLER_PWD="-Dorg.kie.server.controller.pwd=kieserver"
 KIE_CONTROLLER_URL="http://localhost:8080/kie-drools-wb-7.12.0.Final-wildfly11/rest/controller"
 KIE_SERVER_URL="http://localhost:8080/kie-server-7.12.0.Final-ee7/services/rest/server"
 KIE_SERVER="-Dorg.kie.server.location=$KIE_SERVER_URL"
 KIE_CONTROLLER="-Dorg.kie.server.controller=$KIE_CONTROLLER_URL"
 KIE_SERVER_ID="-Dorg.kie.server.id=wildfly-kieserver"

 PARAMS="$SERVER_CONFIG $KIE_SERVER_USER $KIE_SERVER_PWD $KIE_CONTROLLER_USER $KIE_CONTROLLER_PWD $KIE_SERVER $KIE_CONTROLLER $KIE_SERVER_ID"

echo "Starting drools with PARAMS: $PARAMS" 
./standalone.sh $PARAMS
```
- Change the file permissions to allow execute and start the script.
```./start-drools.sh```

- Go to URL using your browser [http://localhost:8080/kie-drools-wb-7.12.0.Final-wildfly11/](http://localhost:8080/kie-drools-wb-7.12.0.Final-wildfly11/) and login using Username ```kieserver``` and password ```kieserver```


## Simple Java Program to connect to Drools Kie Server and execute rules

Refer to sample program at [https://github.com/sompammar/drools-kieserver-restclient](https://github.com/sompammar/drools-kieserver-restclient) for help with creating a sample java application using drools rest api client

## References

The above steps were borrowed and adapted for latest version of drools from this link. [https://www.intertech.com/Blog/simple-setup-of-drools-kie-workbench-and-kie-server-in-one-wildfly-instance/](https://www.intertech.com/Blog/simple-setup-of-drools-kie-workbench-and-kie-server-in-one-wildfly-instance/)

