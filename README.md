# AzureDevOpsAPI

Recently I have started working on the Azure DevOps API using PowerShell.

One should avoid using the PS core (though it provides a controlled number of retry attempts and retry internvals) due to the below issue.
https://github.com/PowerShell/PowerShell/issues/12764

API reference: https://docs.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/create?view=azure-devops-rest-7.1

**A sample code would look somethig like this:**

You can save the env: values to your self-hosted agent current user environment variables, the current user should be the same which is running the Self-Hosted agent services, check services.msc and look for the log on tab for the user, you must be looged in to the Self-Hosted agent to update the current user env variables.

Param(
   [string]$user = "$env:USERNAME",
   [string]$token = "$env:SYSTEM_ACCESSTOKEN"
)


$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $user,$token)))


$type = "application/json-patch+json" # **You should have the content-type value defined as here for write operation**
$apiVersion="7.1-preview.3"

$projectUrl = "$env:SYSTEM_TEAMFOUNDATIONCOLLECTIONURI$env:SYSTEM_TEAMPROJECT"
$WorkItemURL = $projectUrl + '/_apis/wit/workitems/$task?api-version=' + $apiVersion

# these 2 are only permissible in PS 7.x or PScore
# $retryCount = 3
# $retryInterval = 5

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls -bor [Net.SecurityProtocolType]::Tls11 -bor [Net.SecurityProtocolType]::Tls12


Write-Host "Creatin WorkItem type Task" 

#$result = (Invoke-RestMethod -Uri $WorkItemURL -ContentType $type -Method POST -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} )

$body ='[
    {
        "op": "add",
        "path": "/fields/System.Title",
        "from": null,
        "value": "Sample task"
    }
]'

$CreateWorkItem = Invoke-RestMethod -Method Post -Uri $WorkItemURL -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -ContentType $type -Body $body 
