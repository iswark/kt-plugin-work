# kt-plugin-work
Rational Team Concert Java based value provider x 2 to automate individual access control

This plugin adds two new Attribute Customization options under [Configuration Data] > [Work Item] > [Attribute Customization] in the Eclipse client. 

When the plugin is fully configured, it automatically sets the hidden category of a work item to its team area, so that access control is performed without any user knowledge or additional effort. Should the update not be possible due to lack of permission, an informative message is displayed telling the user that they either do not have permission to save against that user, or they need to be added to the relevant team area, and who to contact. Each user who may become owner of a work item (PMR Review or KT One On One) is given their own team area and category. The team area hierarchy is flat. The names of team areas and categories is identical and is a short name derived from the Owned By owner display name. This display name follows set rules and is consistent across our organisation. In addition, there is one team area (of any name) which should be associated with the Unassigned category which contains all users who might create a work item. This allows users to save their work item, even when team areas and categories have not been set up. (We don't want to block productivity.) All other team areas should have users added as members. Those members will have read access to the work items of the user whose name is the same as the team area they were added to.

Already existing work items will have this access control applied by setting up the necessary categories and team areas and then having an administrator who belongs to all the team areas save all the work items. (Can be automated easily.)

This allows the setting of user-level access control to work items using categories and team areas. It operates at an individual level, rather than at team level.

We tested using Jetty and then did integration testing on RTC 6.0.1 and RTC 6.0.2 using the installation steps below. There is ZERO impact on other users. The reason is that you need to install the plugin into an Eclipse client to see the custom attributes and then for anything to happen you need to apply them to specific fields in specific work items. Basically.. other users will be unaffected and unaware of any change.

<Server Installation Steps>

The attached plugin uses jar versions from RTC 6.0.1 and was built under RTC 6.0.2. It will work on RTC 6.0.2 and above.

See the steps under Provisioning the RTC server here for details:
https://jazz.net/wiki/bin/view/Main/AttributeCustomization

1. While the server is started go to https://servername/ccm/admin?showinternal=true and click Server Reset link.
Click the Request Server Reset button to reprovision plugins on next server startup.

2. Log out and stop the server.

3. Copy the attribute-customization-site folder to <installdir>/JazzTeamServer/server/conf/ccm/sites
This folder contains the features folder, plugins folder and site.xml for the plugin.

4. Copy the attribute-customization-site.ini file to <installdir>/JazzTeamServer/server/conf/ccm/provision_profiles
directory. There are other ini files in this directory.

5. Start the server and open the link https://servername/ccm/admin?showinternal=true once more.
Log in as JazzAdmin.

6. Click on Provision Status link.
It will be something like: https://servername/ccm/admin?showinternal=true#action=com.ibm.team.repository.admin.provisionStatus

7. If you get the below message, enable repodebug in Advanced Properties for ccm:
"Error while trying to get provison install log: To access the Provision status you need first to enable the repodebug service in the Advanced Properties." (Search on repodebug and set the Enable repodebug service to true.) 

Click on the Provision Status link again, or if all is displayed, look for the provisioning log messages for the KT plugin by searching on "kt.". You should see something like the below:

CRJAZ0303I The profile install from "file:ccm/sites/attribute-customization-site" was started at "Mon May 29 22:06:57 AEST 2017".

CRJAZ0300I This feature is being installed: "com.ibm.kt.setfiledagainst.calculated.value.provider.feature_1.0.0.201705291714".
CRJAZ0299I Installing bundle from the URL "file:/C%3a/RTC602~1/installs/JAZZTE~1/server/conf/ccm/sites/attribute-customization-site/plugins/com.ibm.kt.setfiledagainst.calculated.value.provider_1.0.0.201705291714.jar".

and further down...

CRJAZ0307I Starting the bundle "com.ibm.kt.setfiledagainst.calculated.value.provider 1.0.0.201705291714".

If you get the third message about starting the bundle, the installation was successful.


Note: The plugin will not display from the https://servername/ccm/admin#action=com.ibm.team.repository.admin.componentStatus link. This is normal. Also, the attribute-customization-site.ini file was obtained on the Windows platform, so you may need to change the line delimiter to Linux. Use the attached file or copy the below two lines to a text file of the same name:

url=file:ccm/sites/attribute-customization-site
featureid=com.ibm.kt.setfiledagainst.calculated.value.provider.feature 

<Client Installation Steps>

These steps must be performed for each Eclipse client that will use the new customization features.

See the steps under Provisioning the Eclipse client here for details:
https://jazz.net/wiki/bin/view/Main/AttributeCustomization

1. Go to [Help] > [Install New Software]
2. Click Add and then Local. Open the directory containing the install site and click OK.

You can find the update site directory here:
--> ktplugin602_final/com.ibm.kt.setfiledagainst.calculated.value.provider.updatesite/

3. Select the new feature called "KT Set Filed Against Feature". You may need to uncheck "Group items by category" to see it. Select to accept the license terms, although there is no license description. Ignore the warning about
installing software with unsigned content and click OK.

<Plugin Configuration -- Part I>

These steps are required once only to set up a project area.

4. Open the project area configuration and select the Process Configuration tab.

5. Click [Configuration Data] > [Work Items] > [Attribute Customization] > [Add] > [Calculated Values]. Choose as Provider "Set Filed Against Value Provider".

6. Click [Configuration Data] > [Work Items] > [Attribute Customization] > [Add] > [Validators]. Choose as Provider "Inform When Owner Is Not Own Student Validator".

7. Under [Configuration Data] > [Work Items] > [Types and Attributes], select the PMR Review type work item.
Add the new calculated value to the Filed Against field and the new validator to the Owned By field. They are built in types, so the change only needs to be done for this work item. Remove dependent fields that are defined, since they will cause the plugin to run more frequently than necessary.

8. Add the Attribute Validation operation behavior to the Everyone Work Item Save operation (otherwise no messages are displayed). It makes sure that a work item can only be saved if all attribute values are valid.

9. (Optional) Add new contacts for the validation error message to the Process Configuration Source. You need to do this after enabling the new validator to make the relevant Global configuration section easy to find. The code is configured as below:

<validator id="com.ibm.team.workitem.valueproviders.VALIDATOR._UEg9oER6EeetT4PB_WqhLg" name="KT Validator" providerId="com.ibm.kt.informWhenOwnerIsNotOwnStudent"/>
    <configuration contactPerson1="xxx" contactPerson2="yyy" severity="warning"/>
</validator>

<Plugin Configuration -- Part II>

The relevant project area needs to have categories and team areas set up as below:

1. Create a team area "WritePermission" and add all users who might create a work item.

2. Create categories for all users who might be set as Owned By in a work item as members. Derive the name of the category from the owner name. Use this algorithm for the category names:

* Remove all sections surrounded in asterisk or round brackets.
* Remove all commas and punctuation and spaces.
* Change all captialization to first letter of a word is captal and the rest is lower case.
E.g. WARK, IAN S (Ian) becomes WarkIanS.

3. Create team areas for the same group of users.

Derive the name of the category from the owner name. Use this algorithm for the team area names:

* Remove all sections surrounded in asterisk or round brackets.
* Remove all commas and punctuation and spaces.
* Change all captialization to first letter of a word is captal and the rest is lower case.
E.g. WARK, IAN S (Ian) becomes WarkIanS.

4. Associate the WritePermission team area with the Unassigned category at the top of the hierarchy.

5. Associate all the other team areas with their corresponding category names.

6. Under Categories section of project area configuration, check to Restrict Category Visibility and Restrict Work Item Access for each team area, except the WritePermission team area. This will hide the category name from the category list and also hide work items for which there is no access permission.

7. Add users as members to the new team areas according to their access permission. Do not forget to add the owner user to their own team area.

<Error messages in ccm.log>

2017-05-27 23:11:31,182 [Default Executor-thread-159 @@ 23:11 WarkIanS <com.ibm.team.workitem.newWorkItem/Save@274856de-bc36-407c-8511-a5bb07e02078> /ccm/service/com.ibm.team.workitem.common.internal.rest.IWorkItemRestService/workItem2] ERROR m.ibm.kt.setfiledagainst.calculated.value.provider  - Missing Category: CorrenteAnthonyA
2017-05-27 23:11:31,292 [Default Executor-thread-159 @@ 23:11 WarkIanS <com.ibm.team.workitem.newWorkItem/Save@274856de-bc36-407c-8511-a5bb07e02078> /ccm/service/com.ibm.team.workitem.common.internal.rest.IWorkItemRestService/workItem2] ERROR m.ibm.kt.setfiledagainst.calculated.value.provider  - Validation Failed: User = WARK, IAN S (Ian) Owner = CORRENTE, ANTHONY A (Anthony)
2017-05-27 23:25:59,633 [Default Executor-thread-2 @@ 23:25 AddisonNigelJ <com.ibm.team.workitem.viewWorkItem/Save@dd57d93c-0c01-4feb-8b41-4df5a27a42dc> /ccm/service/com.ibm.team.workitem.common.internal.rest.IWorkItemRestService/workItem2] ERROR m.ibm.kt.setfiledagainst.calculated.value.provider  - Missing Category: CorrenteAnthonyA
2017-05-27 23:25:59,663 [Default Executor-thread-2 @@ 23:25 AddisonNigelJ <com.ibm.team.workitem.viewWorkItem/Save@dd57d93c-0c01-4feb-8b41-4df5a27a42dc> /ccm/service/com.ibm.team.workitem.common.internal.rest.IWorkItemRestService/workItem2] ERROR m.ibm.kt.setfiledagainst.calculated.value.provider  - Validation Failed: User = ADDISON, NIGEL J (Nigel) Owner = CORRENTE, ANTHONY A (Anthony)

The plugin will occasionally output messages such as the above. The first indicates when categories do not exist as expected, and the second indicates when users do not have permission to save work items against the selected owners.
