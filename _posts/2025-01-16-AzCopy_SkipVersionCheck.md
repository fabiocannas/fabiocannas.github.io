---
title: "AzCopy - Skip Version Check"
date: 2025-01-16
---
Fun fact (maybe not so funny) for azcopy users.
Last night, a customer's automation task that uses **azcopy sync** command started triggering alerts.
This was the error: 

> RESPONSE STATUS CODE ERROR
> GET https://azcopyvnextrelease.blob.core.windows.net/releasemetadata/latest_version.txt?timeout=901
> Accept: application/xml
> User-Agent: AzCopy/10.27.1 azsdk-go-azblob/v1.4.0 (go1.23.1; Windows_NT)
> X-Ms-Client-Request-Id: <request-id>
> x-ms-version: 2024-05-04
> --------------------------------------------------------------------------------
> RESPONSE Status: 401 Server failed to authenticate the request. Please refer to the information in the www-authenticate header.

Luckily, despite these errors, the automation task execution was successful, so the files were synced correctly.

I did a quick search and i ended up reading a discussion on the [azcopy Github project](https://github.com/Azure/azure-storage-azcopy), where a user was reporting the same problem.

Looks like azcopy performs a version check and it can fail sometimes. I did not find a documented way to disable the version check, but looking at commits i found the **"skip-version-check"** flag [see the commit](https://github.com/Azure/azure-storage-azcopy/pull/1950/commits/b7544becdf0ce161fc2cad58b56db4dd8fbabe5a).

Adding **--skip-version-check=true** to **azcopy sync** command resolved the issue.

To sum up, version check failures do not impact on the correct execution of the azcopy operations and the annoying version check failure logs can be suppressed using the aforementioned flag.
