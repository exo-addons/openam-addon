OpenAM Addon
=======

### Install and configure eXo Platform
- Go to $PLATFORM_HOME and install OpenAM with command:
```
./addon install exo-openam
```
- Modify `$PLATFOMR_HOME/gatein/conf/exo.properties` or (`$PLATFORM_HOME/standalone/configuration/gatein/exo.properties` if you use Platform Jboss) then update these configuration (add new if they do not exist)
```
gatein.sso.enabled=true
gatein.sso.callback.enabled=${gatein.sso.enabled}
gatein.sso.login.module.enabled=${gatein.sso.enabled}
gatein.sso.login.module.class=org.gatein.sso.agent.login.SSOLoginModule
gatein.sso.server.url=http://localhost:8888/opensso
gatein.sso.openam.realm=gatein
gatein.sso.portal.url=http://localhost:8080
gatein.sso.filter.logout.class=org.gatein.sso.agent.filter.OpenSSOLogoutFilter
gatein.sso.filter.logout.url=${gatein.sso.server.url}/UI/Logout
gatein.sso.filter.login.sso.url=${gatein.sso.server.url}/UI/Login?realm=${gatein.sso.openam.realm}&goto=${gatein.sso.portal.url}/@@portal.container.name@@/initiatessologin
```
- If you are using Platform tomcat, you will need add `<Valve className="org.gatein.sso.agent.tomcat.ServletAccessValve" />` into `$PLATFORM_HOME/conf/server.xml`. It will look like:
```xml
<Engine name="Catalina" defaultHost="localhost">
    <Host name="localhost" appBase="webapps" startStopThreads="-1"
          unpackWARs="${EXO_TOMCAT_UNPACK_WARS}" autoDeploy="true">
        <Valve className="org.gatein.sso.agent.tomcat.ServletAccessValve" />
        ... 
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        ...
        <Listener className="org.exoplatform.platform.server.tomcat.PortalContainersCreator" />
        ...
    </Host>
</Engine>
```
- If you are using Platform jboss, modify `$PLATFORM_HOME/standalone/configuration/standalone-exo.xml`. Uncomment `SSODelegateLoginModule` and replace `${gatein.sso.login.module.enabled}` with `#{gatein.sso.login.module.enabled}`, `${gatein.sso.login.module.class}` with `#{gatein.sso.login.module.class}`
```xml
<login-module code="org.gatein.sso.integration.SSODelegateLoginModule" flag="required">
    <module-option name="enabled" value="#{gatein.sso.login.module.enabled}"/>
    <module-option name="delegateClassName" value="#{gatein.sso.login.module.class}"/>
    <module-option name="portalContainerName" value="portal"/>
    <module-option name="realmName" value="gatein-domain"/>
    <module-option name="password-stacking" value="useFirstPass"/>
</login-module>
```
- Start eXo Platform

### Install and configure OpenAM server
- Download `OpenAM` from: `http://forgerock.org/openam.html`
- You will need download `apache-tomcat` from: `http://tomcat.apache.org/download-70.cgi`
- Copy `openam-x.x.x.war` into $TOMCAT_HOME/webapps then rename it to `opensso.war`
- Change the default port of apache tomcat to avoid a conflict with the default eXo Platform if you're running eXo Platform and OpenAM on the same machine
In this case we assume that you will change HTTP port from `8080` to `8888`. If you change to other port, you will need update `$PLATFORM_HOME/gatein/conf/exo.properties`
- Now, you can start OpenAM by start tomcat with below command but maybe you will need continue to following section to configure authentication for OpenAM
```
cd $TOMCAT_HOME/bin
./catalina.sh run
```

##### Configure callback authentication for OpenAM
You should following the document at: http://docs.exoplatform.com/public/topic/PLF41/sect-Reference_Guide-Single_Sign_On-OpenAM-OpenAM_server_setup.html#sect-Reference_Guide-Single_Sign_On-OpenAM-OpenAM_server_setup-Configuring_OpenAM_Callback
Note: In step 1 - you will need to copy and merge content of `$PLATFORM_HOME/openam-plugin` into `$OPENAM_TOMCAT_HOME/webapps/opensso` instead of from `$GATEIN_SSO_HOME/opensso/plugin` as in document.

#### Configure non-callback authentication for OpenAM
Following the document at: http://docs.exoplatform.com/public/topic/PLF41/sect-Reference_Guide-Single_Sign_On-OpenAM-OpenAM_server_setup.html#sect-Reference_Guide-Single_Sign_On-OpenAM-OpenAM_server_setup-Configuring_OpenAM_NonCallback
