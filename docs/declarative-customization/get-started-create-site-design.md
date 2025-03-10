---
title: Get started creating SharePoint site designs and site scripts
description: Create site designs to provide reusable lists, themes, layouts, pages, or custom actions so that your users can quickly build new SharePoint sites with the features they need.
ms.date: 08/05/2021
localization_priority: Priority
---

# Get started creating site designs and site scripts

You can create site designs to provide reusable lists, themes, layouts, pages (experimental), or custom actions so that your users can quickly build new SharePoint sites with the features they need.

This article describes how to build a simple site design that adds a SharePoint list for tracking customer orders. You'll use the site design to create a new SharePoint site with the custom list. You'll learn how to use SharePoint PowerShell cmdlets to create site scripts and site designs. You can also use REST APIs to perform the same actions. The corresponding REST calls are shown for reference in each step.

## Create the site script in JSON

A site script is a collection of actions that SharePoint runs when creating a new site. Actions describe changes to apply to the new site, such as creating a new list or applying a theme. The actions are specified in a JSON script, which is a list of all actions to apply. When a script runs, SharePoint completes each action in the order listed.

Each action is specified by the "verb" value in the JSON script. Also, actions can have subactions that are also "verb" values. In the following JSON, the script specifies to create a new list named **Customer Tracking**, and then subactions set the description and add several fields to define the list.

1. Download and install the [SharePoint Online Management Shell](https://www.microsoft.com/download/details.aspx?id=35588). If you already have a previous version of the shell installed, uninstall it first and then install the latest version.
1. Follow the instructions at [Connect to SharePoint Online PowerShell](https://technet.microsoft.com/library/fp161372.aspx) to connect to your SharePoint tenant.
1. Create - and assign the JSON that describes the new script - to a variable as shown in the following PowerShell code. You can view and reference the latest JSON schema file here: https://developer.microsoft.com/json-schemas/sp/site-design-script-actions.schema.json

   ```powershell
    $site_script = '
    {
        "$schema": "schema.json",
            "actions": [
                {
                    "verb": "createSPList",
                    "listName": "Customer Tracking",
                    "templateType": 100,
                    "subactions": [
                        {
                            "verb": "setDescription",
                            "description": "List of Customers and Orders"
                        },
                        {
                            "verb": "addSPField",
                            "fieldType": "Text",
                            "displayName": "Customer Name",
                            "isRequired": false,
                            "addToDefaultView": true
                        },
                        {
                            "verb": "addSPField",
                            "fieldType": "Number",
                            "displayName": "Requisition Total",
                            "addToDefaultView": true,
                            "isRequired": true
                        },
                        {
                            "verb": "addSPField",
                            "fieldType": "User",
                            "displayName": "Contact",
                            "addToDefaultView": true,
                            "isRequired": true
                        },
                        {
                            "verb": "addSPField",
                            "fieldType": "Note",
                            "displayName": "Meeting Notes",
                            "isRequired": false
                        }
                    ]
                }
            ],
                "bindata": { },
        "version": 1
    }
    '
   ```

The previous script creates a new SharePoint list named **Customer Tracking**. It sets the description and adds four fields to the list. Note that each of these are considered an action. Site scripts are limited to 30 cumulative actions (across one or more scripts that may be called in a site design) if applied programmatically using the `Invoke-SPOSiteDesign` command. If they are applied through the UI or using the `Add-SPOSiteDesignTask` command then the limit is 300 cumulative actions (or 100K characters).

## Add the site script

Each site script must be registered in SharePoint so that it is available to use. Add a new site script by using the **Add-SPOSiteScript** cmdlet. The following example shows how to add the JSON script described previously.

```powershell
C:\> Add-SPOSiteScript
 -Title "Create customer tracking list"
 -Content $site_script
 -Description "Creates list for tracking customer contact information"
```

After running the cmdlet, you get a result that lists the site script **ID** of the added script. Keep track of this ID somewhere because you will need it later when you create the site design.

The REST API to add a new site script is **CreateSiteScript**.

## Create the site design

Next, you need to create the site design. The site design appears in a drop-down list when someone creates a new site from one of the templates. It can run one or more site scripts that have already been added.

- Run the following cmdlet to add a new site design. Replace `<ID>` with the site script ID from when you added the site script.

```powershell
C:\> Add-SPOSiteDesign
 -Title "Contoso customer tracking"
 -WebTemplate "64"
 -SiteScripts "<ID>"
 -Description "Tracks key customer data in a list"
```

The previous cmdlet creates a new site design named Contoso customer tracking. The `-WebTemplate` value selects which base template to associate with. The value `"64"` indicates Team site template, and the value `"68"` indicates the Communication site template. If you have disabled modern Group creation (or restricted to a subset of users) and wish to still allow your users to apply site designs to the "group-less" modern Team site template, publish your site designs using the `-WebTemplate` value `"1"`.

The JSON response displays the **ID** of the new site design. You can use it in subsequent cmdlets to update or modify the site design.

The REST API to add a new site design is **CreateSiteDesign**.

## Use the new site design

Now that you've added a site script and site design, you can use it to create new sites through the self-service site creation experience or apply the site design to an existing site using the **Invoke-SPOSiteDesign** command in PowerShell. If you are using hub sites you can even associate a site design to a hub so it gets applied to all joining sites.

### New site creation

1. Go to the home page of the SharePoint site that you are using for development.
1. Choose **Create site**.
1. Choose **Team site**.
1. In the **Choose a design** drop-down, select your site design **Contoso customer tracking**.
1. In **Site name**, enter a name for the new site **Customer order tracking**.
1. Choose **Next**.
1. Choose **Finish**.
1. A notification bar will be displayed indicating that your script is being applied. To invoke the site design information panel, click the **View progress** link. Once the script(s) have completed the notification banner message will change to **Site Design applied. Refresh this site to see the changes.**, allowing you to either invoke the panel or refresh the page.
1. You will see the custom list on the page.

### Apply to an existing site collection

You can also apply a published site design to an existing site collection using the [Invoke-SPOSiteDesign](/powershell/module/sharepoint-online/Invoke-SPOSiteDesign) cmdlet.

You can apply a published site design to:

1. Group-connected Team site
1. Team site not connected to a Microsoft 365 group
1. Communication site
1. Classic team site
1. Classic publishing site

The REST API to apply a site design to an existing site collection is **ApplySiteDesign**.

### Associate with a hub site

You can also associate a published site design to a hub site in hub site settings so it can be applied to all joining sites. For details on how to associate the site design either through the UI or using the `Set-SPOHubSite` cmdlet please review the [PowerShell cmdlets for SharePoint hub sites](../features/hub-site/hub-site-powershell.md) article.

## See also

- [SharePoint site design and site script overview](site-design-overview.md)
