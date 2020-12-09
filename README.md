# tomcat-systemd

Need to see your version 

```
sudo find / -name "tomcat7"

/usr/share/tomcat7/bin$ ./version.sh 
Using CATALINA_BASE:   /usr/share/tomcat7
Using CATALINA_HOME:   /usr/share/tomcat7
Using CATALINA_TMPDIR: /usr/share/tomcat7/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/share/tomcat7/bin/bootstrap.jar:/usr/share/tomcat7/bin/tomcat-juli.jar
NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
Server version: Apache Tomcat/7.0.94 (Debian)
Server built:   May 29 2019 06:04:01 UTC
Server number:  7.0.56-3+really7.0
OS Name:        Linux
OS Version:     4.19.0-5-amd64
Architecture:   amd64
JVM Version:    11.0.3+7-post-Debian-5
JVM Vendor:     Debian
```
Stop and remove old service

```
/etc/init.d/tomcat stop
rm /etc/init.d/tomcat
```
Need to edit new service file

```
nano /etc/systemd/system/tomcat.service

[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
Environment=CATALINA_PID=/usr/share/tomcat7/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/share/tomcat7
Environment=CATALINA_BASE=/usr/share/tomcat7
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/usr/share/tomcat7/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```

Exemple 2

```
[Unit]
Description=Apache Tomcat Web Application Container
Requires=network.target

[Service]
Type=simple
User=tomcat7
Group=tomcat7
WorkingDirectory=/usr/share/tomcat7
ExecStart=/usr/lib/jvm//java-1.7.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/var/lib/tomcat7/conf/logging.properties \
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
-server -Djava.awt.headless=true -Duser.language=en \
-Djavax.net.ssl.trustStore=/etc/ssl/certs/java/cacerts \
-Djavax.net.ssl.trustStorePassword=changeit \
-javaagent:/home/admin/spring-instrument-3.1.2.RELEASE.jar \
-Dcom.e3i.gda.util.chkpasswd=true \
-Xms1g -Xmx6g -XX:MaxPermSize=1g \
-Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n \
-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9015 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=10.x.x.x \
-Djdk.tls.ephemeralDHKeySize=2048 -Dignore.endorsed.dirs= \
-classpath /usr/share/tomcat7/bin/bootstrap.jar:/usr/share/tomcat7/bin/tomcat-juli.jar \
-Dcatalina.base=/var/lib/tomcat7 \
-Dcatalina.home=/usr/share/tomcat7 \
-Djava.io.tmpdir=/tmp/tomcat7-tomcat7-tmp org.apache.catalina.startup.Bootstrap start
ExecStop=/bin/kill -15 $MAINPID
RestartSec=30
Restart=always

[Install]
WantedBy=multi-user.target
```

Exemple 3

```
[Unit]
Description=Tomcat
After=syslog.target network.target

[Service]
Type=forking
User=root
WorkingDirectory=/opt/tomcat8/bin

ExecStart=/opt/tomcat8/bin/startup.sh
ExecStop=/opt/tomcat8/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

After that run 

```
mkdir -p /usr/share/tomcat7/temp
chmod 664 /etc/systemd/system/tomcat.service
```

And now start 

```
systemctl daemon-reload
systemctl start tomcat
```
