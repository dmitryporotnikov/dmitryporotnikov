---
title: "Sample KQL how to filter dismissed Azure Advisor recommendations from Resource Graph Response"
summary: "Sample KQL how to filter dismissed Azure Advisor recommendations from Resource Graph Response"
date: '2024-07-12T00:00:00+00:00'
draft: false
---

# KQL

```kql
advisorresources
| where type =~ 'microsoft.advisor/recommendations'
| project id, stableId = name, subscriptionId, resourceGroup, properties
| join kind=leftouter (
advisorresources
| where type=~'microsoft.advisor/suppressions'
| extend tokens = split(id, '/')
| extend stableId = iff(array_length(tokens) > 3, tokens[(array_length(tokens)-3)], '')
| extend expirationTimeStamp = todatetime(iff(strcmp(tostring(properties.ttl), '-1') == 0, '9999-12-31', properties.expirationTimeStamp))
| where expirationTimeStamp > now()
| project suppressionId = tostring(properties.suppressionId), stableId, expirationTimeStamp
) on stableId
| project id, stableId, subscriptionId, resourceGroup, properties, expirationTimeStamp, suppressionId
| summarize expirationTimeStamp = max(expirationTimeStamp), suppressionIds = make_list(suppressionId), properties=any(properties) by id, stableId
| extend isNotDismissed=isnull(expirationTimeStamp) or isempty(expirationTimeStamp)
| where (isNotDismissed)
| project id, properties
```
