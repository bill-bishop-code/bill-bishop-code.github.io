---
title: "Azure Fhir Server Hard Delete"
date: 2022-06-14T18:12
categories:
  - Blog
tags:
  - Azure
  - Fhir
  - Resource
  - Delete
---
When working with Azure FHIR server I noticed that some resources (I was working with Coverage) were not deleting using the DELETE rest API.  Instead, the
FHIR server creates a new version of the resource that is essentially empty.  For example, if there are 21 versions of a Coverage resource and you 
delete the resource itselft, the FHIR server creates a version 22 that is empty, but the other 21 versions still exist.

If you **really** want to delete a resource use the hardDelete parameter.

```https://{{FHIR_URL}}/Coverage?identifier=1032704&hardDelete=true```

![Hard Delete](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/fhir-rest-api-capabilities#conditional-delete)

While you're there check out the addition API operations the Azure FHIR server provides.

