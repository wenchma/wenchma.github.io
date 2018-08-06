---
layout: post
title: "Jira Single Sign-on Configuration"
categories: tech
tags: jira sso
date: 2018-07-20 14:52:23
---

## Startup one Jira instance

```
docker run --name jira -d -v jirahome:/var/atlassian/jira -v jiraconfhome:/opt/atlassian/jira -p 8088:8080 cptactionhank/atlassian-jira-software:latest
```

## Configuring Jira with JASIG CAS Client

Jira software version: 7.10
JASIG CAS Client for java version: 3.5.0

### 1. Modify the web.xml file

edit `/opt/atlassian/jira/atlassian-jira/WEB-INF/web.xml`

**Add the CAS Filters before the last filter list**:

```
    <!-- CAS:START - Java Client Filters -->
    <filter>
      <filter-name>CasSingleSignOutFilter</filter-name>
      <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>
      <init-param>
        <param-name>casServerUrlPrefix</param-name>
        <param-value>http://mycompany.com/cas/</param-value>
      </init-param>
    </filter>
    <filter>
      <filter-name>CasAuthenticationFilter</filter-name>
      <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
    <init-param>
        <param-name>casServerLoginUrl</param-name>
        <param-value>http://mycompany.com/cas/login</param-value>
    </init-param>
    <init-param>
        <param-name>serverName</param-name>
        <param-value>http://127.0.0.1:8080/</param-value>
    </init-param>
    </filter>
    <filter>
        <filter-name>CasValidationFilter</filter-name>
        <filter-class>org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>
        <init-param>
            <param-name>casServerUrlPrefix</param-name>
            <param-value>http://mycompany.com/cas</param-value>
        </init-param>
        <init-param>
            <param-name>serverName</param-name>
            <param-value>http://127.0.0.1:8080/</param-value>
        </init-param>
        <init-param>
            <param-name>redirectAfterValidation</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <!--- CAS:END -->
```

**Before the login filter-mapping add**:

```
    <!-- CAS:START - Java Client Filter Mappings -->
    <filter-mapping>
      <filter-name>CasSingleSignOutFilter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>CasAuthenticationFilter</filter-name>
        <url-pattern>/login.jsp</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>CasValidationFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- CAS:END -->
```

**Add the Single Sign Out listener to the list of listener list**:

```
    <!-- CAS:START - Java Client Single Sign Out Listener -->
    <listener>
        <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>
    </listener>
    <!-- CAS:END -->
```

### 2. Modify the seraph-config.xml

Comment out the normal login and logout URL and replace it with the CAS login and logout URL:
`/opt/atlassian/jira/atlassian-jira/WEB-INF/classes/seraph-config.xml`

```
<init-param>
    <!--
      The login URL to redirect to when the user tries to access a protected resource (rather than clicking on
      an explicit login link). Most of the time, this will be the same value as 'link.login.url'.
    - if the URL is absolute (contains '://'), then redirect that URL (for SSO applications)
    - else the context path will be prepended to this URL

    If '${originalurl}' is present in the URL, it will be replaced with the URL that the user requested.
    This gives SSO login pages the chance to redirect to the original page
    -->
    <param-name>login.url</param-name>
    <!--<param-value>/login.jsp?os_destination=${originalurl}</param-value>-->
    <param-value>http://cas.mycompany.com/cas/login?service=${originalurl}</param-value>
</init-param>
<init-param>
    <!--
      the URL to redirect to when the user explicitly clicks on a login link (rather than being redirected after
      trying to access a protected resource). Most of the time, this will be the same value as 'login.url'.
    - same properties as login.url above
    -->
    <param-name>link.login.url</param-name>
    <!--<param-value>/login.jsp?os_destination=${originalurl}</param-value>-->
    <!--<param-value>/secure/Dashboard.jspa?os_destination=${originalurl}</param-value>-->
    <param-value>http://cas.mycompany.com/cas/login?service=${originalurl}</param-value>
</init-param>
<init-param>
    <!-- URL for logging out.
    - If relative, Seraph just redirects to this URL, which is responsible for calling Authenticator.logout().
    - If absolute (eg. SSO applications), Seraph calls Authenticator.logout() and redirects to the URL
    -->
    <param-name>logout.url</param-name>
    <!--<param-value>/secure/Logout!default.jspa</param-value>-->
    <param-value>https://cas.mycompany.com/cas/logout</param-value>
</init-param>
```

**Comment out the DefaultAuthenticator and add JASIG CAS Jira Authenticator as follows**

For Jira 4.4 and later:

```
<!-- CAS:START - Java Client Jira Authenticator -->
<authenticator class="org.jasig.cas.client.integration.atlassian.Jira44CasAuthenticator"/>
<!-- CAS:END -->
```

For JIRA 4.3 or earlier:

```
<!-- CAS:START - Java Client Jira Authenticator -->
<authenticator class="org.jasig.cas.client.integration.atlassian.JiraCasAuthenticator"/>
<!-- CAS:END -->
```

### 3. Copy CAS Jar libs

Download [cas-client-core-3.5.0.jar](http://central.maven.org/maven2/org/jasig/cas/client/cas-client-core/3.5.0/cas-client-core-3.5.0.jar) and [cas-client-integration-atlassian-3.5.0.jar](http://central.maven.org/maven2/org/jasig/cas/client/cas-client-integration-atlassian/3.5.0/cas-client-integration-atlassian-3.5.0.jar),  
then copy them to $JIRA_HOME/WEB-INF/lib,  
the verison maybe different as your requirement.

For the source code of Jar libs, pls refer to [github](https://github.com/apereo/java-cas-client)