---
layout: post
title:  "Weblogic xml files for webarchive (war)"
date:   2018-12-21 18:41:34 -0700
categories: weblogic war archive xml
---
If you want to build war file for weblogic following files are needed in WEB-INF directory

WEB-INF/web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
 	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
</web-app>

```

WEB-INF/weblogic.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.oracle.com/weblogic/weblogic-web-app http://xmlns.oracle.com/weblogic/weblogic-web-app/1.7/weblogic-web-app.xsd">
	<wls:container-descriptor>
		<wls:prefer-web-inf-classes>true</wls:prefer-web-inf-classes>
	</wls:container-descriptor>
    <wls:context-root>mywebapp</wls:context-root>
</wls:weblogic-web-app>
```

WEB-INF/wsm-assembly.xml
```
<orawsp:wsm-assembly xmlns:orawsp="http://schemas.oracle.com/ws/2006/01/policy">
  <sca11:policySet xmlns:sca11="http://docs.oasis-open.org/ns/opencsa/sca/200912" name="policySet"
                   appliesTo="REST-RESOURCE()" attachTo="SERVICE('podadmin-rest')" orawsp:highId="1"
                   xml:id="REST-RESOURCE__SERVICE__httptokenservice.HttpBasicAuthApp__">
    <wsp:PolicyReference xmlns:wsp="http://www.w3.org/ns/ws-policy"
                         DigestAlgorithm="http://www.w3.org/ns/ws-policy/Sha1Exc"
                         URI="oracle/wss_http_token_service_policy" orawsp:status="enabled" orawsp:id="1"/>
  </sca11:policySet>
</orawsp:wsm-assembly>
```