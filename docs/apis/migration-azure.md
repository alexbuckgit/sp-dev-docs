---
title: "Use Azure Blob Storage Containers and Azure Queues with SharePoint Migration API"
description: "This article provides in-depth information on how to use the SharePoint Migration API with Azure Containers and Queues."
ms.date: 05/08/2024
ms.author: ranren
author: underreview
manager: dapodean
audience: ITPro
ms.subservice: migration-tool
ms.topic: article
ms.localizationpriority: high
ms.collection:
  - SPMigration
  - m365-collaboration
---

# Use Azure Blob Storage Containers and Azure Queues with SharePoint Migration API

Use Azure Blob Storage Containers to store contents, manifest files, and logs. Access migration status updates with Azure Queues.

This guide provides provisioning, permission, and other requirements with SharePoint-provided Azure resources.

## Azure Blob Storage Containers

Migration API uses Azure Blob Storage Containers for temporary storage of content and manifest. SharePoint provides default containers for migration. Alternatively, you can provide your own containers.

### Using SharePoint-provided containers

SharePoint-provided containers have no extra cost to the customer. Provision SharePoint-provided containers with `ProvisionMigrationContainers` method, without the need to manually set up in the Azure admin console.

Migration API provisions the containers in the same datacenter of the SharePoint instance. Migration API uses a container exclusively once for each request, to ensure security.

Migration API destroys Used containers 30-90 days after completing migration jobs.

#### Decorate the traffic to avoid throttling

[Decorate your HTTP traffic](/sharepoint/dev/general-development/how-to-avoid-getting-throttled-or-blocked-in-sharepoint-online#how-to-decorate-your-http-traffic) to avoid throttling.

#### Encryption

The contents stored in SharePoint-provided containers are encrypted at rest with AES256CBC algorithm. This practice is mandatory. Migration API rejects migration jobs generated from unencrypted SharePoint-provided containers.

This encryption requirement doesn't apply to user-provided containers.

When using `CreateMigrationJobEncrypted` method, encrypt each file with AES CBC 256 standard as the encryption algorithm.

Use the `AesCryptoServiceProvider.GenerateIV` method to generate a unique cryptographically random IV for each file, including the manifests in the package. Store the IV as a property on each file.

##### Name

IV

##### Value

A `BASE64` encoded **Byte Array** of the generated IV

#### ProvisionMigrationContainers method

Provisions new SharePoint-provided migration containers for content and manifest.

##### Syntax

```csharp
public SPProvisionedMigrationContainersInfo ProvisionMigrationContainers()
```

##### Return values

An `SPProvisionedMigrationContainersInfo` object, containing the URI, access tokens, and encryption key of the provisioned containers.

###### DataContainerUri value

A **Uri** value containing the URI of the newly created container for storing migration **content**, along with the SAS access token.

Pass this value to `CreateMigrationJob` method as `azureContainerSourceUri` parameter.

The SAS access token contains `Read` and `Write` permissions only. It doesn't contain `List`.

###### MetadataContainer value

A **Uri** value containing the URI of the newly created container for storing **manifest** files, along with the SAS access token.

Pass this value to `CreateMigrationJob` method as `azureContainerManifestUri` parameter.

The SAS access token contains `Read` and `Write` permissions only. It doesn't contain `List`.

###### EncryptionKey

A **Byte Array** value containing the AES256CBC encryption key for both containers provisioned.

### Use your own containers

You can also provide your own Azure Blob Storage Containers. Provide the URI with SAS access tokens as required in the Migration API reference.

## Azure Queues

SharePoint-provided Azure Queues have no extra cost to the customer. Provision the queue with `ProvisionMigrationQueue` method, without the need to manually set up in Azure admin console.

Migration API provisions Azure Queues in the same datacenter of the SharePoint instance.

Alternatively, use user-provided Azure Queues if desired. Check the requirement in the Migration API Reference document.

### Azure Queue Encryption

When using `CreateMigrationJobEncrypted` method, Migration API encrypts the messages written to the Azure Queue.

Make sure you preserve `JobId` and the `IV` values returned by `CreateMigrationJobEncrypted` method, to decrypt the message.

#### Encrypted message sample

```json
{"Label": "Encrypted"},
{"JobId": "[JobId value]"},
{"IV": "[IV value, encoded in BASE64]"},
{"Content": "[encrypted message, encoded in BASE64]"}
```

### ProvisionMigrationQueue method

Provisions a new instance of Azure Queue for migration use.

#### ProvisionMigrationQueue Syntax

```csharp
public SPProvisionedMigrationQueueInfo ProvisionMigrationQueue()
```

#### Return value

A `SPProvisionedMigrationQueueInfo` object containing the information of the newly created Azure Queue.

##### JobQueueUri value

A `Uri` value containing the URI and SAS access token of the newly created Azure Queue.

Pass this value to `CreateMigrationJob` method's `azureQueueReportUri` parameter.

## Use endpoints in Azure Government

Use Azure Government endpoints for Microsoft 365 Government.

### Required endpoints in Azure Government

- `https://*.blob.core.usgovcloudapi.net`
- `https://*.queue.core.usgovcloudapi.net`
