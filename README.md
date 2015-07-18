
# Tomcat, Debian and systemd

I use [Apache Tomcat](http://tomcat.apache.org/tomcat-7.0-doc/) as a webserver,
running on a [Debian Jessie](https://www.debian.org/) system. Debian has (with
the upgrade from v7 to v8) changed from System V init to systemd, and I wanted
to make best use of systemd.

I stumbled across [a page](http://homepage.ntlworld.com./jonathan.deboynepollard/FGA/systemd-house-of-horror/tomcat.html)
which both explains the problems with the current System V init scripts and 
provides a skeleton systemd unit file for tomcat.

I have adapted the skeleton systemd unit file from that page (to include,
among other things, authbind support) and made a couple of other tweaks.

I'm publishing the info here a) because I'll forget next time I re-install
and b) in case anyone else finds it useful.

# systemd unit file

The [systemd unit file](tomcat7.service) should be installed in `/etc/systemd/system`.
This will over-ride the System V init script.

The unit file pulls some environment variables from the [Debian default](tomcat7) file
(`/etc/default/tomcat7`). (My version is heavily modified compared to the stock version).

Once you've update the unit file remember to reload systemd's configuration:

    $ sudo systemctl demon-reload

# Tomcat server.xml

I've also modified the tomcat [server.xml](server.xml) file, since the stock version doesn't
come with SSL support. To use this version you will also need to generate a self-signed 
certificate:

    keytool -genkey -keysize 4096 -keyalg RSA -sigalg SHA256withRSA \
      -alias tomcat7 -keystore /etc/tomcat7/keystore \
      -keypass "ABetterPasswordThanThis" -storepass "ABetterPasswordThanThis" \
      -dname "CN=localhost, OU=My Toy Server, O=localhost L=Some City, ST=Some State, C=XX"

# authbind

According to its man page, authbind allows you to "bind sockets to privileged ports without root", which is great if you're trying to run an unpriveleged webserver.

To permit tomcat access to ports 80 (http) and 443 (https), run the following command:

    sudo sh -c 'for port in 80 443 ; do $( cd /etc/authbind/byport && touch $port && chown tomcat7.tomcat7 $port && chmod 0100 $port ); done '

which creates two files in /etc/authbind/byport (one named 80 and one named 443), gives them to the tomcat7 user and sets the execute bit.

# Working with maven

[Apache Maven](https://maven.apache.org/) is a "software project management and
comprehension tool" according to its website, or a build and dependency 
management tool (according to me).

Maven can semi-automaticaly deploy to tomcat, but needs a little setting up
on the tomcat side first.

Specificaly, you need to make sure the tomcat-admin package is installed:

    sudo apt-get install tomcat7-admin

add a management user to `/etc/tomcat7/tomcat-users.xml`:

    <tomcat-users>
        <user username="manager" password="ThisPasswordIsImportant" roles="manager-script" />
    </tomcat-users>

(note that anyone who knows, or can brute-force this password can upload web-apsps to your server. I've written a [password generator](https://moosemorals.com/tools/password.html) that might help.)

add a server definition to your maven settings file (`~/.m2/settings.xml`)

    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
      ...
      <servers>
        ...
        <server>
          <id>localhost-tomcat</id>
          <username>manager</username>
          <password>ThisPasswordIsImportant</password>
        </server>
        ...
      </servers>
      ...
    </settings>

and finaly, update your project [pom.xml](https://maven.apache.org/pom.html):

    <project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
      ...
      <profiles>
        ...
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <tomcat-server>localhost-tomcat</tomcat-server>
                <!-- Change this to the hostname of your server
                     Think about https, but self-signed certs aren't
                     trusted. I'm looking into that. -->
                <tomcat-url>http://localhost:80/manager/text</tomcat-url>
            </properties>
        </profile>
        ...
      </profiles>
      ...
      <build>
        ...
        <plugins>
          ...
          <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
              <url>${tomcat-url}</url>
              <server>${tomcat-server}</server>
              <!-- Change this path to the context path for your project -->
              <path>/</path>
            </configuration>
          </plugin>
          ...
        </plugins>
        ...
      </build>
      ...
    </project>

When you're ready to deploy to tomcat, you can just

   mvn tomcat7:redeploy

to update the server.

# Licence

My modifications are MIT licenced, but mostly I'd just like to know if you find them useful.

The original files from Apache ([server.xml](server.xml) and [catalina.properties](catalina.properties))
are [Apache licenced](http://www.apache.org/licenses/LICENSE-2.0.html).

The systemd unit file that I've based mine on is from a webpage © Copyright 2015 Jonathan de Boyne Pollard.
I've emailed asking for permission to distribute my modifications, and I'm waiting for a response.

