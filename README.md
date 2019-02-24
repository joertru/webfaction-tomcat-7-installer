# Tomcat install script for WebFaction #

## What does it do? ##

This creates an Apache Tomcat server, version 8.0.32, and the front-end web server proxies incoming requests to the Tomcat server listening on an unprivileged port. It also makes some configuration adjustments.

The AJP protocol is disabled, since WebFaction is using [nginx][1] as the front-end to proxy incoming requests to the Tomcat server and nginx doesn't support it.

The shutdown port Tomcat uses to terminate itself is also disabled. The process is shuting down with a kill signal from the operating system instead.

The main log, `~/webapps/<app_name>/logs/catalina.out`, is also moved to your `~/logs/user/<app_name>.log` file, with a symbolic link pointing back to it.

You should be able to start the server by executing the `~/webapps/<app_name>/bin/startup.sh` command. You can stop the server by executing the `~/webapps/<app_name>/bin/shutdown.sh` one.

A cron job is created to restart the Tomcat server every 20 minutes if it is not already running.

The official Tomcat 7 documentation is available [here][2].

## Disclaimer ##

This is not an official WebFaction one-click installer, so please address any installer issues on the project's [issue tracker][3].

The installer was inspired after the [Install Tomcat, howto or step by step tutorial][4] question on WebFaction's Q&A forum.

## Installation ##

To install this you need to:

1. Go to the [Add new application][5] form.
2. In the *Name* field, enter a name for the application.
3. In the *App category* menu, click to select *Custom*.
4. In the *App type* menu, click to select *Custom install script*.
5. In the *Script url* field, enter [the URL of the installer file][6] and click the *Fetch scipt* link.
6. Click the blue *Add application* button.

Now, you can [create or modify a website entry][7] which points to your new Tomcat application. Requests to that application’s URL will be proxied by your server to the port number assigned. Your application can then listen and respond to requests on that port.

## Tips ##

### Regarding the shutdown port ###

Tomcat listens to a second port by default, which is used by the shutdown script to terminate the process.

Tomcat's `~/webapps/<app_name>/bin/shutdown.sh` script, when executed, makes a connection to that port in order to close all web applications and shut down cleanly.

But this behavior is a potential security issue in a shared hosting environment, as any user on the server could make a connection to the default 8005 port, using `telnet` and send a SHUTDOWN string to stop your server as easily as:

    $ telnet localhost 8005
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    SHUTDOWN

You can change both the default port, as well as the shutdown string, but that wouldn't stop someone performing a dictionary based attack against your service.

So the installer disables the shutdown port and relies on operating system signals to terminate the process. Because of that, you will see the following message, when you try to shut the service down:

    SEVERE: No shutdown port configured. Shut down server through OS signal. Server not shut down.
    Killing Tomcat with the PID: NNNNN

That's perfectly ok, despite the SEVERE status. It just says that the command couldn't connect to the shutdown port and needed to use the `kill` command to stop the server.

If you do want this feature re-enabled, then assign yourself a port by [creating][8] a "Custom app (listening on port)" application from the control panel. Then open your `~/webapps/<app_name>/conf/server.xml` file on an editor and search for the first Server tag, the one with the shutdown option, and change the port from -1 to the one you've got assigned. Change for example:

    <Server port="-1" shutdown="SHUTDOWN">

to:

    <Server port="NNNNN" shutdown="SHUTDOWN">

where NNNNN is the port you got assigned from the control panel. And do not forget to change the default SHUTDOWN string to a random one.

Now go to your `~/webapps/<app_name>/bin/shutdown.sh` file and remove the `-force` option from the last line, where the command is getting executed.


### Install Oracle's JRE ###

All WebFaction servers come with OpenJDK pre-installed:

    $ which java
    /usr/bin/java
    $ java -version
    java version "1.6.0_24"
    OpenJDK Runtime Environment (IcedTea6 1.11.5) (rhel-1.50.1.11.5.el6_3-x86_64)
    OpenJDK 64-Bit Server VM (build 20.0-b12, mixed mode)

If you want to use Oracle's JRE with your Tomcat instance, here's what you do:

1. Visit [Oracle's Java download page][9] and click to download the latest Java Runtime Environment (JRE) version.
2. On the next page, accept the License Agreement and download the appropriate package. Select the 'Linux x86 tar.gz' file if you are on a CentOS 5 (32-bit) machine, or the 'Linux x64' file if you are on a CentOS 6 (64-bit) one.  
3. When the download is complete, upload it on your server and start an SSH session. See WebFaction's [Accesing Your Data][10] if you don't know how.
4. Create a directory to place Java and traverse to that directory. Execute for example `mkdir ~/opt && cd ~/opt` and press `Enter`.
5. Now extract the java package to the new directory. If you've uploaded the java package in your home directory, then a sample command is `tar xvzf ~/jre-8u45-linux-x64.tar.gz`.
6. Create a symbolic link for easier easier reference. Enter `ln -s jre1.8.0_45/ java` and press `Enter`.
8. Add `~/opt/java/bin` to your PATH environmental variable. Enter `echo -e '\nexport PATH=$HOME/opt/java/bin/:$PATH >> $HOME/.bash_profile` and press `Enter`.
9. Make your .bashrc changes take effect. Enter `source ~/.bash_profile` and press `Enter`

You should now be using Oracle's latest JRE:

    $ which java
    ~/opt/java/bin/java
    $ ./java -version
    java version "1.8.0_45"
    Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
    Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)

To use Oracle's JRE with your Tomcat's instance change the JRE_HOME environmental variable in your `~/webapps/<app_name>/bin/setenv.sh` from:

    JRE_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64

or, on 32-bit machines, from:

    JRE_HOME=/usr/lib/jvm/jre-1.6.0-openjdk

to:

    JRE_HOME=$HOME/opt/java

And restart your Tomcat instance.

[1]: <http://wiki.nginx.org/Main> "nginx's website"
[2]: <http://tomcat.apache.org/tomcat-8.0-doc/index.html> "Tomcat documentation"
[3]: <https://github.com/joertru/webfaction-tomcat-8-installer/issues> "Project's issue tracker"
[4]: <http://community.webfaction.com/questions/4116/install-tomcat-howto-or-step-by-step-tutorial> "Install Tomcat, howto or step by step tutorial"
[5]: <https://my.webfaction.com/new-application/> "Add new application form"
[6]: <https://raw.githubusercontent.com/joertru/webfaction-tomcat-8-installer/master/TomcatInstall> "TomcatInstaller URL"
[7]: <http://docs.webfaction.com/user-guide/websites.html#create-a-website-with-the-control-panel> "Create or modify a website entry"
[8]: <https://my.webfaction.com/new-application> "Create new application"
[9]: <http://www.oracle.com/technetwork/java/javase/downloads/index.html> "Oracle's Java"
[10]: <http://docs.webfaction.com/user-guide/access.html> "Accessing Your Data"
