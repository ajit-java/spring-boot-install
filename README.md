# spring-boot-install

55. Installing Spring Boot applications
Prev 	Part VI. Deploying Spring Boot applications	 Next
55. Installing Spring Boot applications
In additional to running Spring Boot applications using java -jar it is also possible to make fully executable applications for Unix systems. This makes it very easy to install and manage Spring Boot applications in common production environments.

To create a ‘fully executable’ jar with Maven use the following plugin configuration:

<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
With Gradle, the equivalent configuration would be:

apply plugin: 'spring-boot'

springBoot {
    executable = true
}
[Note]
Fully executable jars work by embedding an extra script at the front of the file. Not all tools currently accept this format so you may not always be able to use this technique.
[Note]
The default script supports most Linux distributions and is tested on CentOS and Ubuntu. Other platforms, such as OS X and FreeBSD, will require the use of a custom embeddedLaunchScript.
[Note]
When a fully executable jar is run, it uses the jar’s directory as the working directory.
55.1 Unix/Linux services
Spring Boot application can be easily started as Unix/Linux services using either init.d or systemd.

55.1.1 Installation as an init.d service (System V)

The default executable script that can be embedded into Spring Boot jars will act as an init.d script when it is symlinked to /etc/init.d. The standard start, stop, restart and status commands can be used. The script supports the following features:

Starts the services as the user that owns the jar file
Tracks application’s PID using /var/run/<appname>/<appname>.pid
Writes console logs to /var/log/<appname>.log
Assuming that you have a Spring Boot application installed in /var/myapp, to install a Spring Boot application as an init.d service simply create a symlink:

$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
Once installed, you can start and stop the service in the usual way. You can also flag the application to start automatically using your standard operating system tools. For example, if you use Debian:

$ update-rc.d myapp defaults <priority>
Securing an init.d service

[Note]
The following is a set of guidelines on how to secure a Spring Boot application that’s being run as an init.d service. It is not intended to be an exhaustive list of everything that should be done to harden an application and the environment in which it runs.
When executed as root, as is the case when root is being used to start an init.d service, the default executable script will run the application as the user which owns the jar file. You should never run a Spring Boot application as root so your application’s jar file should never be owned by root. Instead, create a specific user to run your application and use chown to make it the owner of the jar file. For example:

$ chown bootapp:bootapp your-app.jar
In this case, the default executable script will run the application as the bootapp user.

[Tip]
To reduce the chances of the application’s user account being compromised, you should consider preventing it from using a login shell. Set the account’s shell to /usr/sbin/nologin, for example.
You should also take steps to prevent the modification of your application’s jar file. Firstly, configure its permissions so that it cannot be written and can only be read or executed by its owner:

$ chmod 500 your-app.jar
Secondly, you should also take steps to limit the damage if your application or the account that’s running it is compromised. If an attacker does gain access, they could make the jar file writable and change its contents. One way to protect against this is to make it immutable using chattr:

$ sudo chattr +i your-app.jar
This will prevent any user, including root, from modifying the jar.

If root is used to control the application’s service and you use a .conf file to customize its startup, the .conf file will be read and evaluated by the root user. It should be secured accordingly. Use chmod so that the file can only be read by the owner and use chown to make root the owner:

$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
55.1.2 Installation as a systemd service

Systemd is the successor of the System V init system, and is now being used by many modern Linux distributions. Although you can continue to use init.d scripts with systemd, it is also possible to launch Spring Boot applications using systemd ‘service’ scripts.

Assuming that you have a Spring Boot application installed in /var/myapp, to install a Spring Boot application as a systemd service create a script named myapp.service using the following example and place it in /etc/systemd/system directory:

[Unit]
Description=myapp
After=syslog.target

[Service]
User=myapp
ExecStart=/var/myapp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
[Tip]
Remember to change the Description, User and ExecStart fields for your application.
Note that unlike when running as an init.d service, user that runs the application, PID file and console log file behave differently under systemd and must be configured using appropriate fields in ‘service’ script. Consult the service unit configuration man page for more details.

To flag the application to start automatically on system boot use the following command:

$ systemctl enable myapp.service
Refer to man systemctl for more details.

55.1.3 Customizing the startup script

The script accepts the following parameters as environment variables, so you can change the default behavior in a script or on the command line:

Variable	Description
MODE

The “mode” of operation. The default depends on the way the jar was built, but will usually be auto (meaning it tries to guess if it is an init script by checking if it is a symlink in a directory called init.d). You can explicitly set it to service so that the stop|start|status|restart commands work, or to run if you just want to run the script in the foreground.

USE_START_STOP_DAEMON

If the start-stop-daemon command, when it’s available, should be used to control the process. Defaults to true.

PID_FOLDER

The root name of the pid folder (/var/run by default).

LOG_FOLDER

The name of the folder to put log files in (/var/log by default).

LOG_FILENAME

The name of the log file in the LOG_FOLDER (<appname>.log by default).

APP_NAME

The name of the app. If the jar is run from a symlink the script guesses the app name, but if it is not a symlink, or you want to explicitly set the app name this can be useful.

RUN_ARGS

The arguments to pass to the program (the Spring Boot app).

JAVA_HOME

The location of the java executable is discovered by using the PATH by default, but you can set it explicitly if there is an executable file at $JAVA_HOME/bin/java.

JAVA_OPTS

Options that are passed to the JVM when it is launched.

JARFILE

The explicit location of the jar file, in case the script is being used to launch a jar that it is not actually embedded in.

DEBUG

if not empty will set the -x flag on the shell process, making it easy to see the logic in the script.

[Note]
The PID_FOLDER, LOG_FOLDER and LOG_FILENAME variables are only valid for an init.d service. With systemd the equivalent customizations are made using ‘service’ script. Check the service unit configuration man page for more details.
In addition, the following properties can be changed when the script is written by using the embeddedLaunchScriptProperties option of the Spring Boot Maven or Gradle plugins.

Name	Description
mode

The script mode. Defaults to auto.

initInfoProvides

The Provides section of “INIT INFO”. Defaults to spring-boot-application for Gradle and to ${project.artifactId} for Maven.

initInfoShortDescription

The Short-Description section of “INIT INFO”. Defaults to Spring Boot Application for Gradle and to ${project.name} for Maven.

initInfoDescription

The Description section of “INIT INFO”. Defaults to Spring Boot Application for Gradle and to ${project.description} (falling back to ${project.name}) for Maven.

initInfoChkconfig

The chkconfig section of “INIT INFO”. Defaults to 2345 99 01.

useStartStopDaemon

If the start-stop-daemon command, when it’s available, should be used to control the process. Defaults to true.

55.1.4 Customizing the startup script with a conf file

With the exception of JARFILE and APP_NAME, the above settings can be configured using a .conf file,

JAVA_OPTS=-Xmx1024M
LOG_FOLDER=/custom/log/folder
The file should be situated next to the jar file and have the same name but suffixed with .conf rather than .jar. For example, a jar named /var/myapp/myapp.jar will use the configuration file named /var/myapp/myapp.conf if it exists.

To learn about securing this file appropriately, please refer to the guidelines for securing an init.d service.

Prev 	Up	 Next
54. Deploying to the cloud 	Home	 56. Microsoft Windows services
 



Search Documentation
 Search Documentation
