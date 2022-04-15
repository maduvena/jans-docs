# Contents
1. [Overview]()
1. Types of Interception scripts in Jans server
1. An example 
1. [Implementation language - Jython](https://github.com/maduvena/jans-docs/wiki/Interception-scripts---Custom-scripts#implementation-language---jython)
1. [Mandatory methods in any Custom script]()
1. Configurable properties of a custom script
1. Operations on custom scripts using jans-cli
1. Client specific implementations
1. FAQs and troubleshooting

# Overview

Interception scripts (or custom scripts)  allow you to define custom business logic for various features offered by the OpenID Provider (Jans-auth server) like implementing a 2FA authentication method, consent gathering, client registration, adding business specific claims to ID token or Access token to name a few.
Scripts can easily be upgraded and doesn't require forking the Jans Server code or re-building it. 

# Types of Interception scripts in Jans server
Listed below, are custom scripts classified into various types, each of which represents a feature of the Jans server that can be extended as per the business need. Each script type is described by a java interface whose methods should be overridden to implement your business case. 
1. [Person Authentication](https://github.com/maduvena/jans-docs/wiki/Person-Authentication-scripts) : Allows the definition of multi-step authentication workflows, including adaptive authentication, where the number of steps varies depending on the context
1. Consent Gathering : Allows exact customization of the authorization (or consent) process. By default, the OP will request authorization for each scope, and display the respective scope description.
1. User Registration
1. Update User
1. Client Registration
1. Dynamic scopes : Enables admin to generate scopes on the fly, for example by calling external APIs
1. ID Generator
1. Cache Refresh
1. Session Management
1. SCIM
1. Inrospection
1. Resource Owner Password Credentials
1. UMA 2 RPT Authorization Policies
1. UMA 2 Claims-Gathering

# An example
The example below is only meant to convey the concept, we will cover the details in later parts of the documentation.
Suppose, we are implementing an Openbanking Identity platform and we have to add business specific claims say `openbanking_intent_id` to the ID token. The custom script which will help us accomplish our goal is of the type `UpdateTokenType` where the `modifyIdToken` method has to be implemented. A sample custom script with this business logic will be as stated below :
```
class UpdateToken(UpdateTokenType):
    def __init__(self, currentTimeMillis):
        self.currentTimeMillis = currentTimeMillis

    def init(self, customScript, configurationAttributes):
        < initialization code comes here > 
        return True

    def destroy(self, configurationAttributes):
        < clean up code comes here>
        return True

    def getApiVersion(self):
        return <version number>

   def modifyIdToken(self, jsonWebResponse, context):

       Step1: <get openbanking_intent_id from session >
              sessionId = context.getSession()
              openbanking_intent_id = sessionId.getSessionAttributes().get("openbanking_intent_id ")   
    
       Step2: <add custom claims to ID token here>
              jsonWebResponse.getClaims().setClaim("openbanking_intent_id ", openbanking_intent_id )
                
```
# Implementation language - Jython
Interception scripts are written in [Jython](http://www.jython.org/), enabling Java or Python libraries to be imported. While the syntax of the script requires Python, most of the functionality can be written in Java.

## Using Python libraries in a script:
### Caution:
1. You can only use libraries (packages and modules) that are written in **Pure Python**. Importing a Python class which is a wrapper around a library written in C is not supported by the Jans server. As an example, the psycopg2 library used to connect to PostgreSQL from Python. Since it is a C wrapper around libpq, it won't work with Jython.  

1. Python 3 packages / modules are not supported.

### Steps:
1. Pure Python libraries should be added to `/opt/jans/python/libs`

2. Using pip to install additional Python packages: 

* Find out about your Jython version first. cd into the /opt directory in your Jans Server container and run ls. A directory named jython-<version> should be listed too where <version> will correspond to the Jython version. Note the version.
* Open the file `/etc/jans/conf/jans.properties` and look for the line starting with `pythonModulesDir=`. Append the value `/opt/jython-<version>/Lib/site-packages` to any existing value. Each value is separater by a colon (:). It should look something like this ` pythonModulesDir=/opt/jans/python/libs:/opt/jython-2.7.2a/Lib/site-packages` 
Run the following command ` /opt/jython-<version>/bin/jython -m ensurepip `
Install your library with `/opt/jython-<version>/bin/pip install <library_name> ` where <library_name> is the name of the library to install.
* Restart the jans-auth service : `systemctl restart jans-auth`

## Using Java libraries in a script:
### Steps:
1. Add library jars to `/opt/jans/jetty/jans-auth/custom/libs/`
2. Edit /opt/jans/jetty/jans-auth/webapps/jans-auth.xml and add the following line replacing the word `library-name` with the actual name of the library:
```
<Set name="extraClasspath">/opt/jans/jetty/jans-auth/custom/libs/library-name.jar</Set>
```
3. Restart jans-auth service
`systemctl restart jans-auth` 




# Mandatory methods in any Custom script
* `init(self, customScript, configurationAttributes)` :	This method is only called once during the script initialization (or jans-auth service restarts). It can be used for global script initialization, initiate objects etc

* `destroy(self, configurationAttributes)`:	This method is called when a custom script fails to initialize or upon jans-auth service restarts. It can be used to free resource and objects created in the init() method

* `getApiVersion(self, configurationAttributes, customScript)` : The getApiVersion method allows API changes in order to do transparent migration from an old script to a new API. Only include the customScript variable if the value for getApiVersion is greater than 10

# Configurable properties of a custom script
<table>
<tr><td> Name </td><td>unique identifier for the custom script e.g. update_user</td></tr>
<tr><td> Description </td><td>Description text</td></tr>
<tr><td> Programming Languages </td><td>Python </td></tr>
<tr><td> Level </td><td>Used in Person Authentication script type, the strength of the credential is a numerical value assigned to the custom script that is tied to the authentication method. The higher the value, the stronger it is considered. Thus, if a user has several credentials enrolled, he will be asked to present the one of them having the highest strength associated.</td></tr>

<tr><td> Location type </td><td>
<ul><li>Database - Stored in persistence (LDAP, MYSQL or PLSQL whichever applicable ) </li><li>File - stored as a file</li></ul>
 </td></tr>
<tr><td> Interactive </td><td>
<ul><li>Web - web application</li><li>native - mobile application</li><li>both</li></ul>
 </td></tr>
<tr><td> Custom properties</td><td>Key - value pairs for configurable parameters like Third Party API keys, location of configuration files etc </td></tr>
 </table>

#  Operations on custom scripts using jans-cli

Jans-cli supports the following six operations on custom scripts: 

1. `get-config-scripts`, gets a list of custom scripts.
2. `post-config-scripts`, adds a new custom script.
3. `put-config-scripts`, updates a custom script.
4. `get-config-scripts-by-type`, requires an argument `--url-suffix TYPE: ______`.  
    You can specify the following types: PERSON_AUTHENTICATION, INTROSPECTION, RESOURCE_OWNER_PASSWORD_CREDENTIALS, APPLICATION_SESSION, CACHE_REFRESH, UPDATE_USER, USER_REGISTRATION, CLIENT_REGISTRATION, ID_GENERATOR, UMA_RPT_POLICY, UMA_RPT_CLAIMS, UMA_CLAIMS_GATHERING, CONSENT_GATHERING, DYNAMIC_SCOPE, SPONTANEOUS_SCOPE, END_SESSION, POST_AUTHN, SCIM, CIBA_END_USER_NOTIFICATION, PERSISTENCE_EXTENSION, IDP, or UPDATE_TOKEN. 
5. `get-config-scripts-by-inum`, requires an argument `--url-suffix inum: _____`
6. `delete-config-scripts-by-inum`, requires an argument `--url-suffix inum: _____`

The post-config-scripts and put-config-scripts require various details about the scripts. The following command gives the basic schema of the custom scripts to pass to these operations. 
## Basic schema of a custom script
Command:

`/opt/jans/jans-cli/config-cli.py --schema /components/schemas/CustomScript `

Output:
```
{
  "dn": null,
  "inum": null,
  "name": "string",
  "aliases": [],
  "description": null,
  "script": "string",
  "scriptType": "IDP",
  "programmingLanguage": "PYTHON",
  "moduleProperties": {
    "value1": null,
    "value2": null,
    "description": null
  },
  "configurationProperties": {
    "value1": null,
    "value2": null,
    "description": null,
    "hide": true
  },
  "level": "integer",
  "revision": 0,
  "enabled": false,
  "scriptError": {
    "raisedAt": null,
    "stackTrace": null
  },
  "modified": false,
  "internal": false
}
```
To add or modify a script first, we need to create the script's python file (e.g. /tmp/sample.py) and then create a JSON file by following the above schema and update the fields as :

/tmp/sample.json
```
{
  "name": "mySampleScript",
  "aliases": null,
  "description": "This is a sample script",
  "script": "_file /tmp/sample.py",
  "scriptType": "PERSON_AUTHENTICATION",
  "programmingLanguage": "PYTHON",
  "moduleProperties": [
    {
      "value1": "mayvalue1",
      "value2": "myvalues2",
      "description": "description for property"
    }
  ],
  "configurationProperties": null,
  "level": 1,
  "revision": 0,
  "enabled": false,
  "scriptError": null,
  "modified": false,
  "internal": false
}
```
## Add, Modify and Delete a script

The following command will add a new script with details given in /tmp/sample.json file. __The jans-cli will generate a unique inum of this new script if we skip inum in the json file.__ 
``` 
/opt/jans/jans-cli/config-cli.py --operation-id post-config-scripts --data /tmp/sampleadd.json 
```
The following command will modify/update the existing script with details given in /tmp/samplemodify.json file. __Remember to set inum field in samplemodify.json to the inum of the script to update.__ 

``` 
/opt/jans/jans-cli/config-cli.py --operation-id put-config-scripts --data /tmp/samplemodify.json 
```

To delete a custom script by its inum, use the following command: 

``` 
/opt/jans/jans-cli/config-cli.py --operation-id delete-config-scripts-by-inum --url-suffix inum:SAMPLE-TEST-INUM 
```

### List existing custom scripts
These commands to print the details are important, as using them we can get the inum of these scripts which is required to perform update or delete operation.

1. The following command will display the details of all the existing custom scripts. This will be helpful to get the inum of scripts to perform the update and delete operation. 
``` 
/opt/jans/jans-cli/config-cli.py --operation-id get-config-scripts 
```

2. Following command displays the details of selected custom script (by inum). 

``` 
/opt/jans/jans-cli/config-cli.py --operation-id get-config-scripts-by-inum --url-suffix inum:_____  
```

3. Use the following command to display the details of existing custom scripts of a given type (for example: INTROSPECTION). 
``` 
/opt/jans/jans-cli/config-cli.py --operation-id get-config-scripts-by-type --url-suffix type:INTROSPECTION 
```
:memo: **Note:** Incase the AS's Access token is bound to the client's MTLS certificate, you need to add the certificate and key files to the above commands. 
E.g: 
```
/opt/jans/jans-cli/config-cli.py --operation-id post-config-scripts --data /tmp/sampleadd.json -cert-file sampleCert.pem -key-file sampleKey.key 
 ``` 


# Client specific implementations
# Debugging a custom script:
### Setup
Assuming that the Jans-auth server is already installed, perform the following steps:

1. Install https://repo.gluu.org/tools/tools-install.sh
1. Run opt/jans/bin/prepare-dev-tools.py
1. Log in to CE
1. Run /opt/jans/bin/eclipse.sh
Once complete, start the PyDev debug server:

* Open the Eclipse Debug perspective
* From the menu: `Pydev` > `Start Debug Server`

### Development & Debugging
1. Now we are ready to perform script development and debugging. Here is a quick overview:
1. In order to simplify development, put the script into a shared folder like /root/eclipse-workspace
1. Then instruct jans-auth to load the script from the file system instead of LDAP
1. Add debug instructions to the script, as specified in the next section
1. Execute the script

### Enable Remote Debug in Custom Script
1. After the import section, add:
```
REMOTE_DEBUG = True

if REMOTE_DEBUG:
    try:
        import sys
        sys.path.append('/opt/libs/pydevd')
        import pydevd
    except ImportError as ex:
        print "Failed to import pydevd: %s" % ex
        raise

```
1. Add the following lines wherever breakpoints are needed:

```
if REMOTE_DEBUG:
    pydevd.settrace('localhost', port=5678, stdoutToServer=True, stderrToServer=True)
```

### Example

1. Copy the [script](https://github.com/JanssenProject/jans/blob/main/jans-linux-setup/jans_setup/static/extension/person_authentication/BasicExternalAuthenticator.py) to /root/eclipse-workspace/basic.py
2. Change script Location type to File using a jans-cli 
3. Assume that we're using basic.py,  Specify the Script Path location to: /root/eclipse-workspace/basic.py
4. Enable the script using jans-cli 
Check the following log to verify that jans-auth loaded the script properly: /opt/jans/jetty/jans-auth/logs/jans-auth_script.log. It should look like this:
```
... (PythonService.java:239) - Basic. Initialization

... (PythonService.java:239) - Basic. Initialized successfully
```
5. Open the following file in Eclipse: /root/eclipse-workspace/basic.py

6. When opening the Python file for the first time, we need to instruct Eclipse to use a specific interpreter. Follow these steps:

* Press the "Manual Config" button in the dialog box after opening the Python file
* Open "PyDev->Interpreters->Jython Interpreters"
* Click the "New..." button in the right panel. Name it "Jython" and specify the interpreter executable "/opt/jython/jython.jar"
* Click "OK", then confirm the settings by clicking "OK" again, then "Apply and Close"
* In the final dialog, confirm the settings by clicking "OK"

7. Open basic.py in a file editor. After the import section, add the following lines to load the PyDev libraries:
```
REMOTE_DEBUG = True  

if REMOTE_DEBUG:  
    try:  
        import sys  
        sys.path.append('/opt/libs/pydevd')  
        import pydevd  
    except ImportError as ex:  
        print "Failed to import pydevd: %s" % ex  
        raise  

```
8. Add this break condition to the first line in the authenticate method:

```
if REMOTE_DEBUG:   
    pydevd.settrace('localhost', port=5678, stdoutToServer=True, stderrToServer=True) 
``` 
9. Save basic.py

10. Within one minute, jans-auth should load the changed file. Check the following log file again to make sure there are no load errors: /opt/jans/jetty/jans-auth/logs/jans-auth_script.log
11. To check if the script works, update the default authentication method to Basic Authentication. Use jans-cli.

12. Open another browser or session and try to log in. Make sure to keep the first session open in order to disable the Basic Authentication method in case the script doesn't work as expected.
13. After executing `pydevd.settrace` the script will transfer execution control to the PyDev server in Eclipse. You can use any debug commands. For example: Step Over (F6), Resume (F8), etc
14. After debugging is finished, resume script execution to transfer execution control back to jans-auth

### Useful link : https://www.pydev.org/manual_adv_remote_debugger.html

### X Server troubleshooting : Running `/opt/jans/bin/prepare-dev-tools.py` allows Eclipse to access X server.

It runs the following commands:
```
# Only this one key is needed to access from chroot 
xauth -f /root/.Xauthority-jans generate :0 . trusted 2>1 >> /root/prepare-dev-tools.log

# Generate our own key, xauth requires 128 bit hex encoding
xauth -f /root/.Xauthority-jans add ${HOST}:0 . $(xxd -l 16 -p /dev/urandom)

# Copy result key to chroot
cp -f /root/.Xauthority-jans /root/.Xauthority

# Allow to access local server X11   
sudo su $(logname) -c "xhost +local:
```
### Unable to access x11
If Eclipse is unable to access X11, run the following command from the host to check if it has the necessary permissisons:
```
user@u144:~$ xhost +local:
non-network local connections being added to access control list
user@u144:~$ xhost 
access control enabled, only authorized clients can connect
LOCAL:
SI:localuser:user
```
If the user is still unable to access X11, remove .Xauthority from user home and log out/log in again.

###
# FAQs and troubleshooting
# Useful links
1. [Custom scripts and jans-cli](https://github.com/JanssenProject/jans-cli/blob/main/docs/cli/cli-custom-scripts.md#find-list-of-custom-scripts)
