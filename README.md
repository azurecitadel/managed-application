# arm-templates

Development area for building block ARM templates

<https://docs.microsoft.com/en-us/azure/managed-applications/publish-service-catalog-app>
<https://github.com/Azure/azure-managedapp-samples/tree/master/samples/201-managed-web-app>

## Managed Applications

### Overview

1. Package the files
    * Create an ARM template called mainTemplate.json (and optional nested templates)
    * Create a createUiDefinition.json to control the user interface
    * Put the files into a zip that is accessible (can use Azure blob storage and SAS tokens)
1. Create the managed application definition
    * Get an AAD group or application ID
    * Get the role definition ID, e.g. Owner
    * Create a resource group for the application
    * Create the managed application definition resource
    * Allow others to read the group
1. Create the managed application (CLI)
    * Get the managed application ID
    * Deploy using PowerShell or CLI
1. Create the managed application (Portal)
    * Service Catalog
    * Select Service Catalog Managed Application
    * Create
    * Choose the managed application

### Create the managed application definition

The <https://docs.microsoft.com/en-us/azure/managed-applications/publish-service-catalog-app> page has the PowerShell cmdlets. This section will have the Azure CLI commands.

Find (or create) a group by going to <https://aad.portal.azure.com>.  You will need either a Security group of a Mail Enables Security group.  The Office and Distribution groups are no use.  I will use the UK OCP CSA Team in the Microsoft directory as this is a Mail Enabled Security group.  (I believe that these are a combination of Distribution and Security.)  Click into the group and the Properties in the blade to get the Object ID.

UK OCP CSA Team: `fe816505-0da8-4150-8bdc-d195057644c3`
