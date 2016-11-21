* instalar JDK y JBoss
```shell
yum install java-1.8.0-oracle-devel 
yum groupinstall "JBoss EAP 6"
```

* Agregar un Management User en Jboss mediante add-user.sh. Agregar usuario jenkins para realizar las tareas.

* Al instalar jboss hay que habilitar el acceso a la consola y la aplicación por fuera del servidor. Modificar ${JBOSS_HOME}/domain/configuration/host.xml (su equivalente en modo standalone es ${JBOSS_HOME}/standalone/configuration/standalone.xml):

Cambiar los jboss.bind.address (aplicación) y de management (consola) a 0.0.0.0. Tiene que quedar: 
```xml
<interfaces>
	<interface name="management">
		<inet-address value="${jboss.bind.address.management:0.0.0.0}"/>
	</interface>
	<interface name="public">
		<inet-address value="${jboss.bind.address:0.0.0.0}"/>
	</interface>
	<interface name="unsecure">
		<inet-address value="${jboss.bind.address.unsecure:127.0.0.1}"/>
	</interface>
</interfaces>
```

* Agregar JDBCDriver al modo DOMAIN mediante jboss-cli.sh
```
cd /tmp
wget http://ansesalm.anses.gov.ar/nexus/service/local/repositories/maven-anses-homologacion/content/com/microsoft/sqljdbc4/3.0/sqljdbc4-3.0.jar

cd /usr/share/jbossas/bin
./jboss-cli.sh

module add --name=com.microsoft.sqlserver --resources=/tmp/sqljdbc4-3.0.jar  --dependencies=javax.api,javax.transaction.api,org.jboss.as.naming
```
El path en resources es la ubación del jar

* Agregar el modulo desde el jboss cli a tu perfil (full en este caso)
```
/profile=full/subsystem=datasources/jdbc-driver=sqlserver:add(driver-module-name=com.microsoft.sqlserver,driver-name=sqlserver,driver-xa-datasource-class-name=com.microsoft.sqlserver.jdbc.SQLServerXADataSource)

```

* Agregar a domain.xml para usar los modulos de login custom en el perfil correspondiente:
```xml
<security-domain name="ANSeSTokenAuthSecurity">
    <authentication>
        <login-module code="ar.gov.anses.loginModule.ansesToken.jaas.loginModule.AnsesTokenAuthLoginModule" flag="required"/>
    </authentication>
</security-domain>
<security-domain name="ANSeSTokenRenewSecurity ">
    <authentication>
        <login-module code="ar.gov.anses.loginModule.ansesToken.jaas.loginModule.AnsesTokenRenewLoginModule" flag="required"/>
    </authentication>
</security-domain>
```

* Restartear el server.
* Correr las tareas de jenkins: Datasource -> Grupos -> Variables de entorno -> Deploy
Ojo con las tareas de jenkins porque tienen los remove antes de agregar configuraciones, si es la primera vez hay que modificarlas.
Luego del deploy recordar hacerle assign al deployment en el grupo correspondiente.
