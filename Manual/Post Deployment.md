# Post Deployment #

## Essential post deployment steps ##
### Create an admin account ###
An admin account must be created using a script in the glide folder of the application server, the script must be run with the service user and must be supplied with a desired name.
The password will be generated and presented as an output, and will be changed at logon.
```sh
su servicenow
cd /glide/nodes/<NodeName>_<NodePort>/api
./add-user.sh <adminusername>
```

### Configure the "Assigned version" and "Current version" in sys_properties ###
If this environment has outbound HTTPS blocked to the internet, automated "phone-home" functionality for upgrades
fails and must be performed manually. This procedure requires adding the following properties to the sys_properties
table.
glide.war
glide.war.assigned
1. Navigate to sys_properties by typing sys_properties.LIST into the search search filter.
A new browser tab opens, showing all sys_properties records.
2. Click New.
3. Add the following values:
  Name: glide.war
  Description: Current version
  Private: true
4. Populate the Value field as follows:
 * Determine the build version by viewing the contents found in the file /glide/nodes/'node'/glide-release
 * There will be a parameter in that file named glide.node.dist, which has an entry such as: glide-fuji-12-23-2014__patch11-11-18-2015_11-30-2015_1511.
 * You must append .zip to the end of that string, so the entire entry resembles this: glide-fuji-12-23-2014__patch11-11-18-2015_11-30-2015_1511.zip.
5. Click Submit.
6. Follow the same process for the glide.war.assigned record with the exception of the "Assigned version" for the Description field.

 
### Remove Instance Registration logging ##  
1. Logon to the application with admin privileges
2. In the Filter Navigator enter sys_trigger.list
3. Search for registration in name
4. Select the "Register Instance" trigger
5. Change the Trigger type to On Demand
6. Save the change.
  
  
