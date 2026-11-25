---
title: "Upgrading Teamcenter Connector 3.7.0 to 2512.x"
url: /appstore/industry/teamcenter-connector/upgrade-teamcenter-connector-3.7-to-2512/
weight: 45
description: "Describes how to upgrade an app from Teamcenter Connector 3.7.0 (Mendix 9) to Teamcenter Connector 2512.x (Mendix 10), including prerequisites, breaking changes, data migration, and behavior changes."
---

## Introduction

This document guides you through the upgrade of an app from Teamcenter Connector 3.7.0 to Teamcenter Connector 2512.x.

This is a large upgrade:

* Teamcenter Connector 3.7.0 is built for Mendix 9. Teamcenter Connector 2512.x requires Mendix 10.
* The upgrade skips three release lines (2406, 2412, and 2506). This document combines the changes of all of them.
* Version 2512.0.0 reimplemented the login and Single Sign-On (SSO) options. Most breaking changes come from this rework.

Use the latest 2512.x patch.

To see the changes per version, refer to the [Teamcenter Connector](https://marketplace.mendix.com/link/component/111627) release notes.

## Pre-Upgrade Checklist {#pre-upgrade-checklist}

Check these items before you start:

1. Make sure that your Teamcenter version is supported. Check the supported versions in the [Compatibility Matrix](/appstore/industry/teamcenter-connector/compatibility-matrix/). For example, Teamcenter Connector 2512.x does not support Teamcenter 14.x. If your Teamcenter version is not supported, upgrade Teamcenter first.
2. Plan for Mendix Studio Pro 10.24.8 or above. Mendix recommends the latest Mendix 10 LTS version.
3. Make a backup. Commit all changes to version control, or create a full backup of your app.
4. Have only one developer do the upgrade. Other developers pull the result from version control.
5. Check if you changed the `TcConnector` module itself. The import of the new version overwrites these changes. Write down each change, so that you can redo it in your own module after the upgrade.
6. Write down your Teamcenter configurations. For each configuration, note the name, the Teamcenter URL, the FMS URL, and whether **SSO Enabled** is selected. The **SSO Enabled** setting does not survive the upgrade. For details, refer to [Data Migration](#data-migration).
7. Check custom Java code. Version 2512.x runs on Java 21, and the generated Java action classes changed. Custom Java code that uses the connector's Java action classes must be recompiled.
8. If you use Teamcenter SSO, plan the change in Teamcenter Security Services with its administrator. The callback URL of your Mendix application changes, and Teamcenter SSO login does not work until Teamcenter Security Services has the new URL. For details, refer to step 9 of [Upgrading the Teamcenter Connector and Its Dependencies](#upgrade-connector).

## Upgrading the Teamcenter Connector

The upgrade has two stages. Do them in this order:

1. Upgrade the app from Mendix 9 to Mendix 10.
2. Upgrade the Teamcenter Connector and its dependencies to their Mendix 10 versions.

You do not have to do the two stages in one go.

{{% alert color="warning" %}}
Teamcenter Connector 3.7.0 might run on Mendix 10, but this is not recommended. After you move to a new major Mendix version, download the module versions that support that major version. Do stage 2 directly after stage 1.
{{% /alert %}}

### Upgrading from Mendix 9 to Mendix 10

This is stage 1. Follow the steps in [Upgrading from Mendix Studio Pro 9 to 10](/refguide10/upgrading-from-9-to-10/).

### Upgrading the Teamcenter Connector and Its Dependencies {#upgrade-connector}

This is stage 2. Start from the app on Mendix 10. Then follow these steps in this order:

1. Remove the old JAR files.    
    Since version 2406.3.1, the connector uses managed dependencies (the **vendorlib** folder). In the **userlib** folder of your app, delete the following files of Teamcenter Connector 3.7.0, and the matching *.TcConnector.RequiredLib* files:

    * *commons-io-2.21.0.jar*
    * *commons-logging-1.2.jar*
    * *fmsutil.jar*
    * *fscjavaclientproxy.jar*
    * *httpclient-4.5.13.jar*
    * *httpmime-4.5.13.jar*
    * *jackson-annotations-2.13.4.jar*
    * *jackson-core-2.13.4.jar*
    * *jackson-databind-2.13.4.2.jar*
    * *log4j-api-2.17.1.jar*
    * *org.apache.httpcomponents.httpcore_4.4.1.jar*
    * *teamcenter_sso_applib.jar*
    * *teamcenter_sso_common.jar*
    * *teamcenter_sso_ldapcustom.jar*
    * *teamcenter_sso_webtoolkit.jar*

    If another module in your app also needs one of these files, keep that file. A file is needed by another module when a *.{ModuleName}.RequiredLib* file with the same name exists. The import of version 2512.x adds *fmsutil.jar*, *fscjavaclientproxy.jar*, and *TcLogging.jar* again.

2. Add the required Mendix Marketplace modules.    
    Teamcenter Connector 2512.x uses these Marketplace modules, which version 3.7.0 did not use:

    * [Community Commons](https://marketplace.mendix.com/link/component/170)
    * [Encryption](https://marketplace.mendix.com/link/component/1011)

    Download the latest version of each module that supports your Mendix 10 version. If they are already in your app, update them.

3. Set the encryption key.    
    The connector uses the Encryption module to store SSO secrets and tokens. Set the `Encryption.EncryptionKey` constant in each environment, as described in the [Encryption](/appstore/modules/encryption/) documentation.

4. Import the Teamcenter Connector.    
    Download the latest 2512.x version of the [Teamcenter Connector](https://marketplace.mendix.com/link/component/111627). When the **Import Module** dialog box opens, select **Replace existing module**, select the `TcConnector` module in **Module to replace**, and click **Import**. For details, refer to [Using Marketplace Content](/appstore/use-content/#install).

5. Resolve the errors.    
    Studio Pro now shows errors where your app uses elements that were removed or changed. Resolve them with the [Breaking Changes](#breaking-changes) table.

6. Replace the navigation and login microflows:

    * Replace `UserLogin` and `ShowLoginPage` with `NAV_UserLogin`.
    * Replace `LoginToMultipleTeamcenters` with `NAV_UserLoginMultipleActive`.
    * Replace the `AdminHomePage` page with `NAV_AdminHomePage`.
    * If you use Teamcenter SSO, remove `SSO_RegisterRequestHandlers` from your after-startup microflow. It is no longer needed.
    * (Optional) If you use the Teamcenter widgets in Active Workspace, for example the TcBOM widget, call the `TcConnector.Startup` microflow from your after-startup microflow.

7. Update security:

    1. Open the domain model of your module and click **Update security**.
    2. If an entity shows the error `Non-persistable entity (entityName) contains access rules restricting members to have no access rights, which is not allowed`, open the entity and give read access to the attributes that have no rights.
    3. Assign the TcConnector **Administrator** module role to the user roles that configure the Teamcenter connection.
    4. Assign the TcConnector **User** module role to the user roles that log in to Teamcenter. The **Administrator** module role can no longer log in to Teamcenter.

8. Check the Teamcenter configurations.    
    Run the app, log in as an administrator, and open `NAV_AdminHomePage`. For each configuration:

    1. Set **Authentication Method**. All configurations start as **Credentials** after the upgrade. Select **Teamcenter SSO** for each configuration that had **SSO Enabled** selected in version 3.7.0.
    2. Check that the Teamcenter URL begins with `http://` or `https://` and ends with `/tc`, for example `https://tcserver.example.com/tc`. The configuration page no longer accepts URLs that do not begin with `http://` or `https://`.
    3. Check that the FMS URL contains only one URL. Lists of FMS URLs separated by commas are no longer supported.
    4. Save the configuration.

    For details, refer to [Configuring the Connection to Teamcenter with Teamcenter Connector 2512.0.0 and Above](/appstore/industry/teamcenter-connector/configuring-connection-2512/).

9. Update the Teamcenter SSO callback URL (only if you use Teamcenter SSO).    
    The callback URL of the app changed. It was the app URL followed by `/TCSSO`. It is now the app URL followed by `/rest/tcsso/v1/callback`, for example `https://myapp.mendixcloud.com/rest/tcsso/v1/callback`. Do these steps for each environment:

    1. In Teamcenter Security Services, replace the registered callback URL of your Mendix application with the new callback URL. Teamcenter Security Services still has the old `/TCSSO` URL registered. Teamcenter SSO login does not work until you change it.
    2. Make sure that the path `/rest/tcsso/v1/` of your app is reachable from the browsers of your users. Since version 2512.0.0, the published REST service `TcConnector.TCSSO` handles the Teamcenter SSO login and callback on this path. Check path-based access restrictions, proxies, and firewalls.
    3. Set the `TcConnector.SSO_ContextURLPath` constant to `rest/tcsso/v1/callback`.
{{% alert color="warning" %}}Environments where the app is already deployed keep their current constant values. A new default value in the module does not overwrite them. An environment that ran version 3.7.0 keeps `TCSSO` as the value of `SSO_ContextURLPath` until you change it manually. The same applies to `TcConnector.Version`: set it to the installed connector version, because the connector sends this value to Teamcenter as its client version. For Mendix Cloud, refer to the [Constants](/developerportal/deploy/environments-details/#constants) section in *Environment Details*.{{% /alert %}}

10. (Optional) Set up user provisioning for Teamcenter SSO.    
    After the Teamcenter SSO login, the connector calls the `CUSTOM_UserProvisioning` microflow to decide which Mendix user gets the Teamcenter session. By default, this is the Mendix user who started the login, as in version 3.7.0. A user who is not logged in to the app gets an error. If users must be able to log in to the app itself with Teamcenter SSO, change `CUSTOM_UserProvisioning` to use `EXAMPLE_UserProvisioningAnonymous` or your own logic. Keep your logic in a microflow in your own module, and only call it from `CUSTOM_UserProvisioning`. Each import of the connector overwrites `CUSTOM_UserProvisioning`, so you must add the call again after each upgrade.

11. (Optional) Enable the React client.    
    Teamcenter Connector 2512.x is compatible with the React client. To enable it, go to **App Settings** > **Runtime** and select **Yes** for **Use React client**. For details, refer to [React Client](/refguide/mendix-client/react/).

12. Test the app.    
    Test the login for each authentication type that you use, and each Teamcenter operation that your app calls. Pay attention to the [Behavior Changes](#behavior-changes): several Java actions now throw an error where version 3.7.0 returned a result.

## Breaking Changes {#breaking-changes}

The following table lists the changes that can break an app built on version 3.7.0. The **Introduced in** column shows the first version with the change.

| Area | Change | Introduced in | Action required |
| --- | --- | --- | --- |
| Platform | Studio Pro 10.24.8 or above is required. | 2406.3.1 | Upgrade the app to Mendix 10 first (stage 1). |
| Platform | Older Teamcenter versions, such as Teamcenter 14.x, are not supported. | 2512.x | Upgrade Teamcenter to a supported version. Refer to the [Compatibility Matrix](/appstore/industry/teamcenter-connector/compatibility-matrix/). |
| Dependencies | Connector JAR files moved from the **userlib** folder to managed dependencies. | 2406.3.1 | Delete the old JAR files from the **userlib** folder before the import. |
| Dependencies | Community Commons and Encryption are required. | 2412.3.0 | Install both modules and set `Encryption.EncryptionKey`. |
| Java actions | The generated Java action classes extend `UserAction` instead of `CustomJavaAction`. | 2406.3.1 | Recompile custom Java code that uses the connector's Java action classes. |
| Microflows | `UserLogin` and `ShowLoginPage` were removed. | 2512.0.0 | Use `NAV_UserLogin`. |
| Microflows | `LoginToMultipleTeamcenters` was removed. | 2512.0.0 | Use `NAV_UserLoginMultipleActive`. |
| Microflows | `AdminLogin` and `ExecuteAdminLogin` were removed. | 2512.0.0 | Remove the calls. Assign the **User** module role to user roles that log in to Teamcenter. |
| Microflows | `SSO_RegisterRequestHandlers` was removed. | 2512.0.0 | Remove it from the after-startup microflow. |
| Pages | `AdminHomePage`, `HomePage`, and `LoginPage_SSO` were removed. | 2512.0.0 | Use `NAV_AdminHomePage` and `NAV_UserLogin` in the navigation. |
| Pages | `LoginPage`, `LoginPage_Multiple_Active_TeamcenterConfiguration`, `TeamcenterConfiguration_NewEdit`, and `TeamcenterConfiguration_Overview` moved to the internal folder. `TeamcenterConfiguration_NewEdit` has a second page parameter. | 2512.0.0 | Do not open these pages from your app. Use the `NAV_` microflows. |
| Layouts | The `TeamcenterLogin` layout was removed. | 2512.x | Change your pages that use it to another layout. |
| Domain model | The `HomePage` entity was removed. | 2512.0.0 | Remove the references. |
| Domain model | `TcSession` lost the `SSOEnabled` and `SSORedirectURL` attributes. | 2512.0.0 | Remove the references. |
| Domain model | `TeamcenterConfiguration` lost the `SSOEnabled` attribute. `AuthenticationType` replaces it. | 2512.0.0 | Set **Authentication Method** for each configuration. Refer to [Data Migration](#data-migration). |
| Domain model | The `RootTargetAttachments` entity was removed. `root_target_attachments` now connects `EPMTask` to `WorkspaceObject` as a reference set. | 2506.0.1 | Update retrieves, loops, and mappings that use them. |
| Domain model | `release_status_list` changed from a reference to a reference set. | 2406.3.1 | Update microflows that expect one release status: a retrieve now returns a list. |
| Constants | The default value of `SSO_ContextURLPath` changed from `TCSSO` to `rest/tcsso/v1/callback`. Deployed environments keep their old value. | 2512.0.0 | Set the new value in each deployed environment, and register the new callback URL in Teamcenter Security Services. |
| Java actions | `UploadFiles` no longer returns `false` when the upload fails. It throws an error. | 2506.0.1 | Replace checks on a `false` result with an error handler. |
| Java actions | `ExpandPSOneLevel2` creates `FileDocument` objects only when `DownloadFiles` is true. | 2512.1.3 | Set `DownloadFiles` to true where you use the returned `FileDocument` objects. |
| Configuration | A comma-separated list of FMS URLs is no longer supported. | 2512.1.3 | Use one FMS URL per configuration. |

### Security Changes

| Area | Change | Introduced in | Action required |
| --- | --- | --- | --- |
| `TeamcenterConfiguration` | Access rules list the attributes. The **Administrator** module role no longer has create and delete rights. The **User** module role reads only `ConfigName`, `TCURL`, `FMSURL`, `AuthenticationType`, and `Active`. | 2512.0.0 | Click **Update security**. Change custom pages that show other configuration attributes to users. |
| `ExecuteLogin`, `ExecuteLogout` | Only the **User** module role can run them. | 2512.0.0 | Assign the **User** module role to user roles that log in to Teamcenter. |
| `TcConnector.TCSSO` (published REST service) | The connector publishes a REST service on `/rest/tcsso/v1/` for the Teamcenter SSO login and callback. | 2512.0.0 | Include this path in your security review and make sure it is reachable for Teamcenter SSO users. |
| `RetrieveConfigNameFromSingleActiveConfiguration`, `RetrieveTcSessionBasedOnConfigName`, `RetrieveHttpHeaderList` | No module role can run them from the client. | 2412.3.0 and 2512.0.0 | Call them only from other microflows. |

## Data Migration {#data-migration}

Only one change affects data that your app stores:

* The `SSOEnabled` attribute of the persistent `TeamcenterConfiguration` entity was removed in version 2512.0.0. The new `AuthenticationType` attribute replaces it. The configuration page shows this attribute as **Authentication Method**. Its values are **Credentials**, **Teamcenter SSO**, and **Teamcenter X SSO**. Its default value is **Credentials**.
* When you run the app after the upgrade, the database synchronization removes the `SSOEnabled` column. All existing configurations get **Credentials** as their authentication type.
* The other SSO attributes (`SSOLoginURL`, `SSOIdentityURL`, `SSOTCAppId`, and `SSOMendixAppId`) keep their values.

To restore the SSO setting, set **Authentication Method** to **Teamcenter SSO** for each configuration that had **SSO Enabled** selected. You wrote these down in the [pre-upgrade checklist](#pre-upgrade-checklist).

Several string attributes changed their length to match the Teamcenter data model. For example, `ReleaseStatus.name` changed from 200 to 32 and `BOMLine.object_string` changed from 200 to 4000. All these entities are non-persistent, so no stored data changes. If your app copies these values into its own persistent entities, check the lengths of your attributes.

| Entity | Attribute | 3.7.0 | 2512.x |
| --- | --- | --- | --- |
| `BOMLine`, `POM_application_object` | `object_string` | 200 | 4000 |
| `EPMTask` | `comments` | 200 | 4000 |
| `EPMTask` | `task_name` | 200 | 32 |
| `EPMTaskTemplate` | `template_name` | 200 | 128 |
| `Group` | `name` | 200 | 128 |
| `ImanQuery` | `query_desc` | 200 | 512 |
| `ImanQuery` | `query_name` | 200 | 128 |
| `Item` | `item_id` | 200 | 128 |
| `ReleaseStatus` | `name` | 200 | 32 |
| `Role` | `role_name` | 200 | 32 |
| `TC_Project` | `project_desc` | 200 | 240 |
| `TC_Project` | `project_id` | 200 | 64 |
| `TC_Project` | `project_name` | 200 | 32 |
| `ValuesInput` | `value` | 200 | Unlimited |

## Behavior Changes {#behavior-changes}

These changes do not cause errors in Studio Pro, but they change how the app behaves at runtime:

| Area | Change | Introduced in | Action required |
| --- | --- | --- | --- |
| Error handling | The connector throws errors instead of returning `false`, `null`, or showing a message. This applies to `UploadFiles`, `DownloadFiles`, `DownloadImages`, `GetProperties` (when the input is empty), `Logout`, and `RetrieveConfigNameFromSingleActiveConfiguration`. | 2506.0.1 to 2512.1.3 | Add error handlers around these calls. |
| Error handling | `DownloadFiles` and `DownloadImages` return `false` only when the dataset has no files. | 2512.1.3 | Use an error handler for all other failures. |
| `UploadFiles` | A dataset with an empty name or an empty dataset type is rejected. | 2512.1.3 | Set both values before the call. |
| `RetrieveTeamcenterConifgurationByName` | The microflow no longer validates the configuration and no longer shows a message. | 2512.0.0 | Check if the result is empty. |
| Logging | Failed Teamcenter calls are logged at the ERROR level, also for expected failures such as wrong credentials. | 2506.0.1 and 2512.0.0 | Review your log monitoring. |
| `TeamcenterConfiguration` | The validation of configurations moved from the entity to the configuration page. Unique names and URLs are no longer enforced by the entity. | 2512.0.0 | If your logic creates or changes configurations, validate them in your logic. |
| `BOMLine_childrenObj` | Deleting a `BOMLine` object also deletes its child objects. | 2406.3.1 | Check logic that deletes `BOMLine` objects. |
| Business object mapping | For each child object, the connector creates the nearest matching specialization. A child that does not match is skipped. Before, the mapping stopped at the first such child. | 2406.3.1 and 2506.0.1 | Retest mappings that use specializations. |
| `CreateObjects` | All compound create inputs of the same entity type are sent. Before, some were dropped. | 2506.0.1 | Retest create calls with several compound inputs of one type. |
| Partial errors | The client index of partial errors is read correctly. | 2506.0.1 | Retest logic that reads partial errors by client index. |
| Request body | Unicode escape sequences in SOA request bodies are sent as plain characters. | 2412.3.0 | Retest calls that send non-ASCII data. |
| Associations for Teamcenter properties | The connector has its own associations for the Teamcenter properties `structure_revisions` (`PSBOMViewRevision` to `ItemRevision`), `bl_revision` (`BOMLine` to `ItemRevision`), `top_line` (`BOMWindow` to `BOMLine`), and `revision_rule` (`BOMWindow` to `RevisionRule`). If your app maps one of these properties with its own association, Teamcenter can return the related objects without properties. | 2406.3.1 | Refer to [Related Objects Returned Without Properties](#empty-bvr-properties). |
| Property values | When a Teamcenter property contains at least one object UID, the connector treats the whole property as a reference. Values in that property that are not UIDs become empty. | 2512.0.1 | Retest mappings of properties that mix UIDs and other values. |
| Type hierarchy | When `ConfigurationName` is empty, the connector loads the type hierarchy of the active configuration. Business object mapping can then create a more specific specialization than before. | 2406.3.1 | Retest mappings with specializations when you leave `ConfigurationName` empty. |
| Client version | The client version sent to Teamcenter no longer starts with `MxTcC_`. | 2406.3.1 | Update Teamcenter-side reports or filters that use this prefix. |
| New data | The database synchronization adds the persistent entities `OAuthToken`, `SSOCallbackHelper`, and `TcXCallbackHelper`, and two attributes on `FileDocument` (`datasetUID`, `imanFileID`). The scheduled event `SCE_CleanupOldSSOCallbackHelpers` removes old `SSOCallbackHelper` objects every hour. | 2406.3.1 to 2512.1.3 | Make sure that scheduled events run in your environments. |

## Removed, Moved, and Deprecated Elements

| Element | Status | Replacement |
| --- | --- | --- |
| `UserLogin`, `ShowLoginPage` (microflows) | Removed in 2512.0.0 | `NAV_UserLogin` |
| `LoginToMultipleTeamcenters` (microflow) | Removed in 2512.0.0 | `NAV_UserLoginMultipleActive` |
| `AdminLogin`, `ExecuteAdminLogin` (microflows) | Removed in 2512.0.0 | **User** module role |
| `SSO_RegisterRequestHandlers` (microflow) | Removed in 2512.0.0 | Not needed. `TcConnector.Startup` for Active Workspace widgets only. |
| `AdminHomePage` (page) | Removed in 2512.0.0 | `NAV_AdminHomePage` |
| `HomePage`, `LoginPage_SSO` (pages) | Removed in 2512.0.0 | `NAV_UserLogin` |
| `TeamcenterLogin` (layout) | Removed in 2512.x | Your own layout |
| `HomePage` (entity) | Removed in 2512.0.0 | None |
| `RootTargetAttachments` (entity) | Removed in 2506.0.1 | `root_target_attachments` from `EPMTask` to `WorkspaceObject` |
| `CloseBOMWindows` (Java action) | Moved to the internal folder in 2406.3.1 | Still available |
| `CreateBOMWindows`, `CreateBOMWindows2` (Java actions) | Deprecated in 2512.0.0 | `CreateOrReConfigureBOMWindows` |
| `GetItemFromId` (Java action) | Deprecated in 2512.0.0 | `GetItemAndRelatedObjects` |

Deprecated Java actions still work in 2512.x. Plan to replace them before you move to version 2606.

The following microflows are new since version 3.7.0 and available for use: `DownloadFile`, `CloseBOMWindow`, `HandleActiveConfigErrors`, and `TeamcenterConfiguration_ShowLoginPage`.

The following Java actions are new since version 3.7.0 and available for use: `ExecuteSavedQueriesWithObjectRefresh`, `DownloadImages`, `GetImanFiles`, `GetWorkflowTemplates2`, `PerformAction3`, `WhereUsed2`, `CreateOrReConfigureBOMWindows`, `GetItemAndRelatedObjects`, and `GetAttachedPropDescs2`.

## Known Issues {#known-issues}

### Related Objects Returned Without Properties {#empty-bvr-properties}

`GetProperties` can return related objects without properties, and without an error, when both of these conditions are true:

* Your app has its own association for a Teamcenter property for which the connector also has an association since version 2406.3.1: `structure_revisions`, `bl_revision`, `top_line`, or `revision_rule`.
* The business object mapping maps the Teamcenter type to an entity of your app that is a specialization of `TcConnector.ModelObject`, and not of the matching connector entity (for example, `TcConnector.PSBOMViewRevision`).

The connector then requests the same property twice from Teamcenter, and the request without properties wins. The Teamcenter response contains the related object with an empty `props` object. This is confirmed for `structure_revisions` with BOM view revisions. The other three properties use the same mechanism.

Use one of these workarounds:

* Change your entity, so that it is a specialization of the matching connector entity, for example `TcConnector.PSBOMViewRevision`.
* Call `GetProperties` again, with only the related objects as input.

### English Locale with Teamcenter SSO

With Teamcenter SSO, the Teamcenter session always uses the `en_US` locale. In version 3.7.0, the session used the language of the Mendix user.

## Next Steps

After you finish the upgrade to 2512.x, you can move to Teamcenter Connector 2606. For details, refer to [Migrating to Teamcenter Connector 2606](/appstore/industry/teamcenter-connector/migrating-to-unified-teamcenter-connector/).
