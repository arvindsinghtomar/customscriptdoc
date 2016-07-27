[TOC]

# Writing a Custom Authentication Script
To develop this article, I decided to write a script to use
[Twilio](http://twilio.com) to send an SMS code to implement a two-step
out-of-band authentication mechanism.

## Custom Script Location
Gluu Server User Interface allows the developer to specify the path to
script rather than use the text box from oxTrust. The script will be
loaded from the specified path. This feature will improve script
development and there is an option to revert back to working script, if
the script is faulty or needs further development. 

The administrator can select `File` from the Script Location Type in oxTrust and the file
input box will be displayed:
![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/script_upload_box.png)

The 'LDAP' option in the Script Location
Type can be used to store the script in the LDAP tree once the
development is complete. Remember that selecting the `LDAP` method
requires that the script has to be copied in the input box that appears
upon LDAP selection:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/script_in_ldap.png)

## Suggested Development Environment

Gluu Server custom scripts are written in
[Jython](http://www.jython.org/). In this article, I'm going to
recommend using Eclipse. You cannot actually run the scripts from
Eclipse, but it provides a nice editor which makes coding more
enjoyable.

Before you begin, install the following:

 - [Python 2.7.x](https://www.python.org/downloads/)
 - [Jython](http://www.jython.org/downloads.html)
 - [Java Standard Edition](http://www.oracle.com/technetwork/java/javase/terms/products/index.html)
 - [Eclipse IDE for Java Developers](https://eclipse.org/downloads/)

### Install Eclipse Plugins

PyDev is a useful Eclipse tool for editing Python files. It also
supports Jython. To add an Eclipse plugin, the first thing to do to
select "Install New Software" from the Help menu:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/01-install-software.jpg)

After this, you'll need to add the repositories for the software.

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/02-Add-Repositories.jpg)

 - PyDev - http://download.jboss.org/jbosstools/mars/development/updates/
 - JBoss Tools - http://download.jboss.org/jbosstools/mars/development/updates/

Then, select the respective repository in the "Work with" dropdown, and
check the box for the software you want to install, as in the images
below:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/03-add-Pydev.jpg)

From JBoss Tools select "Jboss Tools RichFaces" in "JBoss Web and Jave
EE Development":

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/04-add-jboss-richfaces.jpg)

### Create a project

As the next step specify the Jython interpreter under the Window /
Preferences menu:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/05-preferences-jython-interpreter.jpg)

Now you can create a new project to keep all your stuff. From the File
menu, create a new PyDev project:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/06-new-pydev-project.jpg)

Now, you can create some files:

 - A Python file for your script
 - Zero or more XHTML files if you have a custom form for your authentication
 - Zero or more XML files (you'll need one for each XHTML file) that
   provide some information to the Tomcat server about how to display
   the XHTML file.

## Samples and Documentation

There are many good examples of authentication interception scripts
checked into the [integrations
folder](https://github.com/GluuFederation/oxAuth/tree/master/Server/integrations)
of the oxAuth project. Also, the respective `XHTML` and `XML` files are
checked in to the [auth
folder](https://github.com/GluuFederation/oxAuth/tree/master/Server/src/main/webapp/auth). The interfaces for the authentication interception can be found in the [Gluu
Documentation](http://www.gluu.org/docs/reference/interception-scripts/#authentication).

When I started my Twilio script, I decided to use the [Basic
Script](https://raw.githubusercontent.com/GluuFederation/oxAuth/master/Server/integrations/basic/BasicExternalAuthenticator.py)
as a template. I also used the [Wikid
Forms](https://github.com/GluuFederation/oxAuth/tree/master/Server/src/main/webapp/auth/wikid) as my templates for the forms, because I remembered that I'd basically
need to get a code in step 2.
I also looked at the [Wikid
Authentication](https://github.com/GluuFederation/oxAuth/blob/master/Server/integrations/wikid/WikidExternalAuthenticator.py)
script quite a bit for examples of how to process the form.

## Implement methods

The most important method to implement is obviously the `authenticate`
method. This is where the main business logic is located for your
authentication workflow. Notice how you can switch on the step, with the
`if (step == 1):` statement. In oxAuth, there is no assumption that step
1 and step 2 happen on the same server, so the step is sent into the
`authenticate` method.

Another method you usually need to implement is
`getCountAuthenticationSteps`. This method normally just returns 1, 2,
or 3. If you are implementing an adaptive authentication strategy, where
the number of steps depends on the context, you need to get a little
fancier. Check out the Duo script for a good example of this. In our
sample [Duo
script](https://github.com/GluuFederation/oxAuth/blob/master/Server/integrations/duo/DuoExternalAuthenticator.py),
the organization we wrote it for only wanted to use Duo for the IT
group. So we checked group membership, and dynamically adjusted the
number of steps.

If you need to save session variables between steps, use the
`getExtraParametersForStep` method. The Gluu Server persists these
variables in LDAP in able to support stateless clustered two step
authentications.

If you need to display a special Web page for an interactive login, (or
even a custom first page) you'll need to implement the `getPageForStep`
which specifies the page you want to return for a given step.

## Custom Properties

Sometimes it is helpful to enable system administrators to enter
properties that might change a lot. If you don't want to ask them to
modify the script, you can use the Custom Property feature, as seen in
this screenshot:

![image](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/auth_article/07-custom-properties.jpg)

To access this information in your script with
`configurationAttributes.get("<key>").getValue2()` where `<key>`
specifies the value you want to retrieve.

## Returning a message to the user

It is possible to use the Context to return a message to the user, which
could be especially useful if an error happened, or you need some kind
of user action.

## Adding Libraries

If you have some pure Python libraries, you can add them to
`/opt/tomcat/conf/python`, jar files can be added to
`/opt/tomcat/endorsed`.

## Testing

So you think you're done with your script, it is time to test it! Print
statements are sent to `/opt/tomcat/logs/wrapper.log` If you prefix your
logs, you can use `tail -f | grep <prefix>` to just see your script
output while you try to login to test your script. In the Twilio test
script I wrote a special method called `printOut` to make it easier to
add this prefix.

Also, remember that putting all your code in a `try / catch` is a good
practice to avoid unhandled exceptions. But when you're debugging,
sometimes those exceptions may give you a hint as to what's going wrong.

# Reverting Authentication Method
It is not unlikely that you will lock yourself out of Gluu Server while testing the authentication script, if there is any problem in it. In such a case the following method can be used to revert back the older authentication method. Please see the [FAQ](../faq/troubleshooting.md) for details.

