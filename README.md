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
    * Create a resource group for the application definition
    * Create the managed application definition resource
    * Allow others to read the definition
1. Create the managed application (CLI)
    * Get the managed application ID
    * Deploy using PowerShell or CLI
1. Create the managed application (Portal)
    * Service Catalog
    * Select Service Catalog Managed Application
    * Create
    * Choose the managed application

### Package the files

Create as a GitHub repo and then it is easy.  Or use the following:

```bash
az group create --name managedApplications --location eastus

subscriptionId=$(az account show --query id --output tsv)
storageAccount=manageapps$(echo $subscriptionId | md5sum | tr -dc "[:lower:][:digit:]" | head -c12)

az storage account create --name $storageAccount \
  --kind Storage \
  --sku Standard_LRS \
  --resource-group managedApplications \
  --location eastus

storageKey=$(az storage account keys list \
  --account-name $storageAccount \
  --resource-group managedApplications \
  --query "[1].value" --output tsv)

az storage container create --name appcontainer \
  --account-name $storageAccount \
  --account-key $storageKey

az storage blob upload --name app.zip \
  --account-name $storageAccount \
  --account-key $storageKey \
  --container appcontainer \
  --file app.zip
```

### Create the managed application definition

The <https://docs.microsoft.com/en-us/azure/managed-applications/publish-service-catalog-app> page has the PowerShell cmdlets. This section will have the Azure CLI commands.

#### Get the security group's objectId

Find (or create) a group by going to <https://aad.portal.azure.com>.  You will need either a Security group of a Mail Enables Security group.  The Office and Distribution groups are no use.  I will use the UK OCP CSA Team in the Microsoft directory as this is a Mail Enabled Security group.  (I believe that these are a combination of Distribution and Security.)  Click into the group and the Properties in the blade to get the Object ID.

Or via the commands.  Find your object ID from your user principal name

```bash
az ad user show --upn-or-object-id richeney@microsoft.com --query objectId --output tsv
95efbdcd-1e74-4428-a1a0-bf9a8884faa2
az ad user get-member-groups --upn-or-object-id 95efbdcd-1e74-4428-a1a0-bf9a8884faa2 --security-enabled-only --output json --query "[?contains(displayName, 'OCP')]"
az ad group show --group "UK OCP CSA Team" --query objectId --output tsv
fe816505-0da8-4150-8bdc-d195057644c3
objectId=$(az ad group show --group "UK OCP CSA Team" --query objectId --output tsv)
```

#### Get the inbuilt role's objectId

We'll use the unbuilt Owner role.

```bash
az role definition list --name Owner --query [].name --output tsv
8e3af657-a8ff-443c-a75c-2fe8c4bcb635
roleId=$(az role definition list --name Owner --query [].name --output tsv)
```

**ToDo: Script this based on being provided a standard AD security group name.**

#### Create the resource group

```bash
az group create --location westeurope --name managedApplications --output json
```

#### Create the managed application

You'll need the raw path for the zip file.

This repo is <https://github.com/azurecitadel/arm-templates> and the zip is in the root and called myFirstManagedApplication.zip.

If you navigate through to it and right click the Download button then the Copy Link then the URL is:

`https://github.com/azurecitadel/arm-templates/raw/master/myFirstManagedApplication.zip`

If it is in an Azure blob then you can use the following:

```bash

```

```bash
objectId=$(az ad group show --group "UK OCP CSA Team" --query objectId --output tsv)
roleId=$(az role definition list --name Owner --query [].name --output tsv)
zipUri="https://github.com/azurecitadel/arm-templates/raw/master/myFirstManagedApplication.zip"

az managedapp definition create \
    --name "MyFirstManagedApplication" \
    --location "West Europe" \
    --resource-group "managedApplications" \
    --lock-level ReadOnly \
    --display-name "Managed Application: Network" \
    --description "Core network with optional VPN Gateway" \
    --authorizations "$objectId:$roleId" \
    --package-file-uri $zipUri
```

Example output json:

```json
{
  "artifacts": [
    {
      "name": "ApplicationResourceTemplate",
      "type": "Template",
      "uri": "https://prdsapplianceprodam01.blob.core.windows.net/applicationdefinitions/55993_2D31BE49D9594415BB658AEC2C90BA62_872BF3F66509FB610503D8504C4F684830CB4723/applicationResourceTemplate.json?sv=2017-04-17&sr=b&sig=7uCWEvPO74zV1scQ%2FmRmq72BDoC5INND9tqTBGGQVgk%3D&se=2118-10-03T11:10:59Z&sp=r"
    },
    {
      "name": "CreateUiDefinition",
      "type": "Custom",
      "uri": "https://management.azure.com/subscriptions/2d31be49-d959-4415-bb65-8aec2c90ba62/resourceGroups/managedApplications/providers/Microsoft.Solutions/applicationDefinitions/MyFirstManagedApplication/applicationArtifacts/CreateUiDefinition?api-version=2017-09-01"
    }
  ],
  "authorizations": [
    {
      "principalId": "fe816505-0da8-4150-8bdc-d195057644c3",
      "roleDefinitionId": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
    }
  ],
  "createUiDefinition": null,
  "description": "Core network with optional VPN Gateway",
  "displayName": "Managed Application: Network",
  "id": "/subscriptions/2d31be49-d959-4415-bb65-8aec2c90ba62/resourceGroups/managedApplications/providers/Microsoft.Solutions/applicationDefinitions/MyFirstManagedApplication",
  "identity": null,
  "isEnabled": "True",
  "location": "westeurope",
  "lockLevel": "ReadOnly",
  "mainTemplate": null,
  "managedBy": null,
  "name": "MyFirstManagedApplication",
  "packageFileUri": null,
  "resourceGroup": "managedApplications",
  "sku": null,
  "tags": null,
  "type": "Microsoft.Solutions/applicationDefinitions"
}
```

#### Allow others to read the definition

One thing to note is that RBAC inheritance (from Management Group -> Subscription -> Resource Group -> Resource) cannot be blocked, so it is advisable to limit the users and groups at the top level.

The managed application definition can be seen in the resource group:

```bash
az resource list --resource-group managedApplications --output table
Name                       ResourceGroup        Location    Type                                        Status
-------------------------  -------------------  ----------  ------------------------------------------  --------
MyFirstManagedApplication  managedApplications  westeurope  Microsoft.Solutions/applicationDefinitions

az managedapp definition show --name "MyFirstManagedApplication" --resource-group managedApplications --output json
```

Let's add another group as readers for the definition.  For this I will use the OCP CRM Users group:

```bash
readerObjectId=$(az ad group show --group "OCP CRM Users" --query objectId --output tsv)
```

You can either add the Reader role to the resource group which makes sense if you keep all of the managed applications together.  (Put the resource group in the shared subscription and call it managedApplications? Test with a CSP partner.)  And then you can skip this step when adding additional managed application definitions in the future.

```bash
az role assignment create --role Reader --resource-group managedApplications --assignee-object-id $readerObjectId --output json
```

If you want to be more granular then assign to the individual resource:

```bash
managedAppId=$(az managedapp definition show --name "MyFirstManagedApplication" --resource-group managedApplications --output tsv --query id)
az role assignment create --role Reader --scope $managedAppId --assignee-object-id $readerObjectId --output json
```

If you wanted to give the original group (UK OCP CSA Team) as Owner for the resource group then the command would be:

```bash
objectId=$(az ad group show --group "UK OCP CSA Team" --query objectId --output tsv)
az role assignment create --role Owner --resource-group managedApplications --assignee-object-id $objectId --output json
```

#### Updating the managed application

If you change the files then recreate the zip and rerun the `az managedapp definition create` command.

You can check the uploaded version of the template reflects the change. (You'll need to install jq first)

```bash
templateUri=$(az managedapp definition show --name "MyFirstManagedApplication" --resource-group managedApplications --query "artifacts[?type == 'Template'].uri" --output tsv)
curl $templateUri | jq
```

### Deploy the managed application via the Azure CLI

The uri for the UI definition cannot be seen in the same way as it requires an authorization header.

## Partner Admin Link

<https://aka.ms/adminpartnerlink>

az extension add --name managementpartner
az login --tenant <tenant>
az managementpartner create --partner-id 12345

(this works in the Cloud Shell as well)

Dummy MPN ID (actually linked back to Microsoft): **3583635**

    Location ID is 3583636 (Redmond)
    There is a UK test location ID: 4166758

OK, so PAL looks like it is just linking an account (user or service principal) in a tenancy to an MPN.  And some magic does the rest for reporting.  Auto DPoR?
