---
id: 771
title: 'List services scheduled for retirement in Azure Graph / KQL'
summary: 'List services scheduled for retirement in Azure Graph / KQL'
date: '2024-02-09T11:34:55+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=771'
permalink: /2024/02/09/kql-query-microsoft-services-retirement/
categories:
    - Azure
    - Code
---

## List services scheduled for retirement in Azure Graph / KQL

Below you will find a KQL query to list services that are about to be retired by Microsoft.

This is intended to check against the list of resources announced at
<https://azure.microsoft.com/updates/?updateType=retirements>

As a foundation, I used this workbook:
<https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Azure%20Advisor/AzureServiceRetirement/Azure%20Services%20Retirement.workbook>  
from which I extracted the KQL.

```kql
resources
| extend ServiceID= case(
type contains "microsoft.compute/virtualmachine" and  (tostring(properties.hardwareProfile.vmSize) in~ ('basic_a0','basic_a1','basic_a2','basic_a3','basic_a4','standard_a0','standard_a1','standard_a2','standard_a3','standard_a4','standard_a5','standard_a6','standard_a7','standard_a9')  or tostring(sku.name) in~ ('basic_a0','basic_a1','basic_a2','basic_a3','basic_a4','standard_a0','standard_a1','standard_a2','standard_a3','standard_a4','standard_a5','standard_a6','standard_a7','standard_a9')),1,
type == "microsoft.web/hostingenvironments" and kind in ('ASEV1','ASEV2'),3,
type == "microsoft.compute/virtualmachines" and isempty(properties.storageProfile.osDisk.managedDisk),4,
type == "microsoft.dbforpostgresql/servers" ,5,
type == "microsoft.dbformysql/servers"  ,7,
type == "microsoft.network/loadbalancers" and sku.name=='Basic',8,
type == "microsoft.operationsmanagement/solutions" and plan.product=='OMSGallery/ServiceMap',9,
type == "microsoft.insights/components" and isempty(properties.WorkspaceResourceId) ,10,
type == 'microsoft.datalakeanalytics/accounts' and not(location in ('westcentralus','westus2','westus3')) and properties contains 'defaultDataLakeStoreAccount', 11,
type == 'microsoft.classicstorage/storageaccounts',12,
type == 'microsoft.classiccompute/domainnames', 13,
type == "microsoft.dbforpostgresql/servers" and properties.version == "11",14,
type == "microsoft.logic/integrationserviceenvironments",15,
type == 'microsoft.classicnetwork/virtualnetworks',16,
type == "microsoft.network/applicationgateways" and properties.sku.tier in~ ('Standard','WAF'),17,
type == "microsoft.datalakestore/accounts" and not(location in ('westcentralus','westus2')) ,19,
type == 'microsoft.classicnetwork/reservedips',20,
type == 'microsoft.classicnetwork/networksecuritygroups',21,
type =~ 'Microsoft.CognitiveServices/accounts' and kind=~'QnAMaker',36,
type contains "microsoft.compute/virtualmachine" and  (tostring(properties.hardwareProfile.vmSize) in~ ('Standard_HB60rs','Standard_HB60-45rs','Standard_HB60-30rs','Standard_HB60-15rs')  or tostring(sku.name) in~ ('Standard_HB60rs','Standard_HB60-45rs','Standard_HB60-30rs','Standard_HB60-15rs')) ,40,
type contains "Microsoft.MachineLearning/",41,
type =~ "Microsoft.Network/publicIPAddresses" and sku.name=='Basic',42,
type =~ 'Microsoft.CognitiveServices/accounts' and kind contains 'LUIS',43,
type contains 'Microsoft.TimeSeriesInsights',44,
type =~ "microsoft.dbforpostgresql/servers" and properties.version == "11",45,
type =~ "Microsoft.DBforPostgreSQL/serverGroupsv2" and properties.postgresqlVersion==11,46,
type contains "microsoft.media/mediaservices",50,
type =~ "microsoft.insights/alertrules",51,
type =~ "Microsoft.DataBoxEdge/dataBoxEdgeDevices" and set_has_element(todynamic(properties.configuredRoleTypes), "IOT"),53,
type =~ "microsoft.migrate/projects",54,
type =~ "microsoft.insights/workbooks" and properties.category =~ "tsg",56,
type =~ "microsoft.maps/accounts" and (sku has "S1" or sku has "S0"),58,
type =~ "microsoft.insights/webtests" and properties.Kind =~ "ping",59,
type =~ 'microsoft.healthcareapis/services',60,
type =~ 'microsoft.healthcareapis' and properties.authenticationConfiguration.smartProxyEnabled =~ 'true',63,
type contains "Microsoft.DBforMariaDB",64,
type =~ "microsoft.cache/redis" and properties['minimumTlsVersion'] in ("1.1","1.0") ,65,
-9999)
| where ServiceID >0
| project ServiceID , location, id , resourceGroup
| union 
// Query for Classic Redis caches retired
(advisorresources
| where type =='microsoft.advisor/recommendations'
| where properties.shortDescription contains 'Cloud service caches are being retired'
| project id=tolower(tostring(properties.resourceMetadata.resourceId))
| join 
(resources
| where type contains 'microsoft.cache/redis'
| project id=tolower(id), resourceGroup, location
) on id
| project ServiceID=22 , id, resourceGroup, location 
)
| union 
// Query for synapse Runtime for Apache Spark 2.4 and 3.1
(resources
| where type == "microsoft.synapse/workspaces/bigdatapools"  and todouble(properties.sparkVersion) in (2.4,3.1)
| extend workspaceId = tostring(split(id,'/')[8]) 
| join kind=leftouter
(resources
| where type == "microsoft.synapse/workspaces" and properties.adlaResourceId == ""
| project workspaceId = name, adla=1
) on workspaceId
| where adla==1
| project ServiceID = iff (todouble(properties.sparkVersion)==2.4,31,32),id, resourceGroup, location)
```
