---
id: 735
title: 'Azure Resource Graph Queries to see who has access to Azure Subscriptions'
summary: 'Azure Resource Graph Queries to see who has access to Azure Subscriptions'
date: '2024-01-15T14:30:41+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=735'
permalink: /2024/01/15/azzure-resource-graph-query-access/
categories:
    - Azure
---

## Azure Resource Graph Queries to see who has access to Azure Subscriptions

Occasionally, you may need to gather data regarding all resource groups, identities, and users with access to your Azure subscriptions. This can be efficiently achieved using the Azure Resource Graph.  
  
Some samples below.

**Shows all resources:**

```plaintext
authorizationresources
| where type =~ 'microsoft.authorization/roleassignments'
| extend principalType = tostring(properties['principalType'])
| extend principalId = tostring(properties['principalId'])
| extend roleDefinitionId = tolower(tostring(properties['roleDefinitionId']))
| join kind=inner ( 
    authorizationresources
    | where type =~ 'microsoft.authorization/roledefinitions'
    | extend id = tolower(id)
) on $left.roleDefinitionId == $right.id
| summarize count() by roleDefinitionId, principalType
| where count_ > 1
| sort by count_ desc
```

**Shows identities with contributor access:**

```plaintext
authorizationresources
| where type =~ 'microsoft.authorization/roleassignments'
| extend principalType = tostring(properties['principalType'])
| extend principalId = tostring(properties['principalId'])
| extend roleDefinitionId = tolower(tostring(properties['roleDefinitionId']))
| join kind=inner ( 
    authorizationresources
    | where type =~ 'microsoft.authorization/roledefinitions'
    | where properties['roleName'] =~ 'Contributor'
    | extend id = tolower(id)
) on $left.roleDefinitionId == $right.id
| summarize count() by roleDefinitionId, principalType
| where count_ > 1
| sort by count_ desc
```

**Shows everyone who has an access to operate Virtual Machines:**

```plaintext
authorizationresources
| where type =~ 'microsoft.authorization/roledefinitions'
| extend roleDefinitionId = tolower(id), roleName = tostring(properties['roleName']), permissions = properties['permissions']
| mv-expand perm = permissions
| mv-expand action = perm['actions']
| where tostring(action) contains "Microsoft.Compute"
| project roleDefinitionId, roleName
| distinct roleDefinitionId, roleName
| join kind=inner (
    authorizationresources
    | where type =~ 'microsoft.authorization/roleassignments'
    | extend principalType = tostring(properties['principalType']), principalId = tostring(properties['principalId']), roleDefinitionId = tolower(tostring(properties['roleDefinitionId']))
) on roleDefinitionId
| project principalType, principalId, roleName
| distinct principalType, principalId, roleName
```
