
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

# Tomcat server.xml

I've also modified the tomcat [server.xml](server.xml) file, since the stock version doesn't
come with SSL support. To use this version you will also need to generate a self-signed 
certificate:

    keytool -genkey -keysize 4096 -keyalg RSA -sigalg SHA256withRSA \
      -alias tomcat7 -keystore /etc/tomcat7/keystore \
      -keypass "ABetterPasswordThanThis" -storepass "ABetterPasswordThanThis" \
      -dname "CN=localhost, OU=My Toy Server, O=localhost L=Some City, ST=Some State, C=XX"

# Licence

My modifications are MIT licenced, but mostly I'd just like to know if you find them useful.

The original files from Apache ([server.xml](server.xml) and [catalina.properties](catalina.properties))
are [Apache licenced](http://www.apache.org/licenses/LICENSE-2.0.html).

The systemd unit file that I've based mine on is from a webpage © Copyright 2015 Jonathan de Boyne Pollard.
I've emailed asking for permission to distribute my modifications, and I'm waiting for a response.

