---
title: Azure Cosmos DB resource model
description: This article describes Azure Cosmos DB resource model which includes the Azure Cosmos account, database, container, and the items. It also covers the hierarchy of these elements in an Azure Cosmos account. 
author: seesharprun
ms.author: sidandrews
ms.reviewer: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/03/2022
---

# Azure Cosmos DB resource model

[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB is a fully managed platform-as-a-service (PaaS). To begin using Azure Cosmos DB, create an Azure Cosmos DB account in an Azure resource group in your subscription. You then create databases and containers within the account.

Your Azure Cosmos DB account contains a unique DNS name and can be managed using the Azure portal, ARM or Bicep templates, Azure PowerShell, Azure CLI, or any of the Azure Management SDK's or REST API. For more information, see [how to manage your Azure Cosmos DB account](how-to-manage-database-account.md). For replicating your data and throughput across multiple Azure regions, you can add and remove Azure regions to your account at any time. You can configure your account to have either a single region or multiple write regions. For more information, see [how to add and remove Azure regions to your account](how-to-manage-database-account.md). You can configure the [default consistency](consistency-levels.md) level on an account.

## Elements in an Azure Cosmos DB account

Currently, you can create a maximum of 50 Azure Cosmos DB accounts under an Azure subscription (this is a soft limit that can be increased via support request). A single Azure Cosmos DB account can virtually manage an unlimited amount of data and provisioned throughput. To manage your data and provisioned throughput, you can create one or more databases within your account, then one or more containers to store your data. The following image shows the hierarchy of elements in an Azure Cosmos DB account:

:::image type="content" source="./media/account-databases-containers-items/hierarchy.png" alt-text="Hierarchy of an Azure Cosmos DB account" border="false":::

The following image shows the hierarchy of different entities in an Azure Cosmos DB account:

:::image type="content" source="./media/account-databases-containers-items/cosmos-entities.png" alt-text="Azure Cosmos DB account entities" border="false":::

## Azure Cosmos DB databases

In Azure Cosmos DB, a database is similar to a namespace. A database is simply a group of containers. The following table shows how a database is mapped to various API-specific entities:

| Azure Cosmos DB entity | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- |
|Azure Cosmos database | Database | Keyspace | Database | Database | NA |

> [!NOTE]
> With Table API accounts, to maintain compatibility with Azure Storage Tables, tables in Azure Cosmos DB are created at the account level.

## Azure Cosmos DB containers

An Azure Cosmos DB container is where data is stored. Unlike most relational databases which scale up with larger VM sizes, Azure Cosmos DB scales out. Data is stored on one or more servers, called partitions. To increase throughput or storage, more partitions are added. This provides a virtually an unlimited amount of throughput and storage for a container. When a container is created, you need to supply a partition key. This is a property you select from your documents to store. The value of that property is then used to route data to the partition to be written, updated, or deleted. It can also be used in the WHERE clause in queries for efficient data retrieval.

The underlying storage mechanism for data in Azure Cosmos DB is called a physical partition. These can have a throughput amount up to 10,000 RU/s and store up to 50 GB of data. Azure Cosmos DB abstracts this with a logical partition which can store up to 20 GB of data. Logical partitions allow the service to provide greater elasticity and better management of data on the underlying physical partitions as more partitions are added. To learn more about partitioning and partition keys, see [Partition data](partitioning-overview.md).

When you create a container, you configure throughput in one of the following modes:

* **Dedicated throughput**: The throughput provisioned on a container is exclusively reserved for that container. There are two types of dedicated throughput, standard and autoscale. To learn more, see [How to provision throughput on a container](how-to-provision-container-throughput.md).

* **Shared throughput**: Here throughput is specified at the database level, then shared with up to 25 containers within the database, (excluding containers that have been configured with dedicated throughput). This can be a good option when all of the containers in the database have similar requests and storage needs or when you don't need predictable performance on the data. To learn more, see [How to provision throughput on a database](how-to-provision-database-throughput.md).

> [!NOTE]
> You can not go between dedicated and shared throughput. Containers created in a shared throughput database, cannot be updated to have dedicated throughput. To change a container from shared to dedicated throughput, a new container must be created and data copied to it.

Containers are schema-agnostic. Items within a container can have arbitrary schemas or different entities so long as they share the same partition key. For example, an item that represents a customer and one or more items representing all their orders, can be placed in the *same container*. By default, all data added to a container is automatically indexed without requiring explicit indexing. You can customize the indexing for a container by configuring its [indexing policy](index-overview.md).

You can set [Time to Live (TTL)](time-to-live.md) on selected items in a container or for the entire container to silently delete those items automatically in the background with unused throughput to avoid impacting performance. However, even if not deleted, any data that has expired will not appear in any reads made. To learn more, see [Configure TTL on your container](how-to-time-to-live.md).

Azure Cosmos DB provides a built-in change data capture capability called, [change feed](change-feed.md) that can be used to subscribe to all the changes to data within your container. For more information, see [Change feed in Azure Cosmos DB](change-feed.md).

You can register [stored procedures, triggers, user-defined functions (UDFs)](stored-procedures-triggers-udfs.md), and [merge procedures](how-to-manage-conflicts.md) for your container.

Data within a container must have a unique `id` property value for each logical partition key value. This can be useful when you want to have a unique constraint within your container.  You can also specify a [unique key constraint](unique-keys.md) on your Azure Cosmos DB container that uses one or more different properties and ensures uniqueness of one or more values per logical partition key. If you create a container by using a unique key policy, no new or updated items with values that duplicate the values specified by the unique key constraint can be created. To learn more, see [Unique key constraints](unique-keys.md).

A container is specialized into API-specific entities as shown in the following table:

| Azure Cosmos entity | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- |
|Azure Cosmos container | Container | Table | Collection | Graph | Table |

> [!NOTE]
> When creating containers, make sure you don’t create two containers with the same name but different casing. Some parts of the Azure platform are not case-sensitive, and this can result in confusion/collision of telemetry and actions on containers with such names.

### Properties of an Azure Cosmos DB container

An Azure Cosmos DB container has a set of system-defined properties. Depending on which API you use, some properties might not be directly exposed. The following table describes the list of system-defined properties:

| System-defined property | System-generated or user-configurable | Purpose | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_rid | System-generated | Unique identifier of container | Yes | No | No | No | No |
|\_etag | System-generated | Entity tag used for optimistic concurrency control | Yes | No | No | No | No |
|\_ts | System-generated | Last updated timestamp of the container | Yes | No | No | No | No |
|\_self | System-generated | Addressable URI of the container | Yes | No | No | No | No |
|id | User-configurable | Name of the container | Yes | Yes | Yes | Yes | Yes |
|indexingPolicy | User-configurable | Provides the ability to change indexes | Yes | No | Yes | Yes | Yes |
|TimeToLive | User-configurable | Provides the ability to delete items automatically from a container after a set time period. For details, see [Time to Live](time-to-live.md). | Yes | No | No | No | Yes |
|changeFeedPolicy | User-configurable | Used to read changes made to items in a container. For details, see [Change feed](change-feed.md). | Yes | No | No | No | Yes |
|uniqueKeyPolicy | User-configurable | Used to ensure the uniqueness of one or more values in a logical partition. For more information, see [Unique key constraints](unique-keys.md). | Yes | No | No | No | Yes |
|AnalyticalTimeToLive | User-configurable | Provides the ability to delete items automatically from a container after a set time period. For details, see [Time to Live](analytical-store-introduction.md). | Yes | No | Yes | No | No |

## Azure Cosmos DB items

Depending on which API you use, data can represent either an item in a container, a document in a collection, a row in a table, or a node or edge in a graph. The following table shows the mapping of API-specific entities to an Azure Cosmos item:

| Cosmos entity | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- |
| Azure Cosmos DB item | Item | Row | Document | Node or edge | Item |

### Properties of an item

Every Azure Cosmos item has the following system-defined properties. Depending on which API you use, some of them might not be directly exposed.

| System-defined property | System-generated or user-defined| Purpose | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_rid | System-generated | Unique identifier of the item | Yes | No | No | No | No |
|\_etag | System-generated | Entity tag used for optimistic concurrency control | Yes | No | No | No | No |
|\_ts | System-generated | Timestamp of the last update of the item | Yes | No | No | No | No |
|\_self | System-generated | Addressable URI of the item | Yes | No | No | No | No |
|id | Either | User-defined unique name in a logical partition. | Yes | Yes | Yes | Yes | Yes |
|Arbitrary user-defined properties | User-defined | User-defined properties represented in API-native representation (including JSON, BSON, and CQL) | Yes | Yes | Yes | Yes | Yes |

> [!NOTE]
> Uniqueness of the `id` property is enforced within each logical partition. Multiple documents can have the same `id` property with different partition key values.

### Operations on items

Azure Cosmos items support the following operations. You can use any of the Azure Cosmos APIs to perform the operations.

| Operation | SQL API | Cassandra API | Azure Cosmos DB API for MongoDB | Gremlin API | Table API |
| --- | --- | --- | --- | --- | --- |
| Insert, Replace, Delete, Upsert, Read | Yes | Yes | Yes | Yes | Yes |

## Next steps

Learn how to manage your Azure Cosmos account and other concepts:

* To learn more, see the [Azure Cosmos DB SQL API](/learn/modules/intro-to-azure-cosmos-db-core-api/) learn module.
* [How-to manage your Azure Cosmos DB account](how-to-manage-database-account.md)
* [Global distribution](distribute-data-globally.md)
* [Consistency levels](consistency-levels.md)
* [VNET service endpoint for your Azure Cosmos DB account](how-to-configure-vnet-service-endpoint.md)
* [IP-firewall for your Azure Cosmos DB account](how-to-configure-firewall.md)
* [How-to add and remove Azure regions to your Azure Cosmos DB account](how-to-manage-database-account.md)
* [Azure Cosmos DB SLAs](https://azure.microsoft.com/support/legal/sla/cosmos-db/)
