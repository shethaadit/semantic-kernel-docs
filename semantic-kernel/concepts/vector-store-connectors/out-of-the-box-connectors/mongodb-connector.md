---
title: Using the Semantic Kernel MongoDB Vector Store connector (Preview)
description: Contains information on how to use a Semantic Kernel Vector store connector to access and manipulate data in MongoDB.
zone_pivot_groups: programming-languages
author: dmytrostruk
ms.topic: conceptual
ms.author: dmytrostruk
ms.date: 10/25/2024
ms.service: semantic-kernel
---
# Using the MongoDB Vector Store connector (Preview)

> [!WARNING]
> The Semantic Kernel Vector Store functionality is in preview, and improvements that require breaking changes may still occur in limited circumstances before release.

::: zone pivot="programming-language-csharp"

## Overview

The MongoDB Vector Store connector can be used to access and manage data in MongoDB. The connector has the following characteristics.

| Feature Area                      | Support                                                                                                                          |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Collection maps to                | MongoDB Collection + Index                                                                                                       |
| Supported key property types      | string                                                                                                                           |
| Supported data property types     | <ul><li>string</li><li>int</li><li>long</li><li>double</li><li>float</li><li>decimal</li><li>bool</li><li>DateTime</li><li>*and enumerables of each of these types*</li></ul> |
| Supported vector property types   | <ul><li>ReadOnlyMemory\<float\></li><li>ReadOnlyMemory\<double\></li></ul>                                                       |
| Supported index types             | N/A                                                                                                                              |
| Supported distance functions      | <ul><li>CosineSimilarity</li><li>DotProductSimilarity</li><li>EuclideanDistance</li></ul>                                        |
| Supported filter clauses          | <ul><li>EqualTo</li></ul>                                                                                                        |
| Supports multiple vectors in a record | Yes                                                                                                                          |
| IsFilterable supported?           | Yes                                                                                                                              |
| IsFullTextSearchable supported?   | No                                                                                                                               |
| StoragePropertyName supported?    | No, use BsonElementAttribute instead. [See here for more info.](#data-mapping)                              |

## Getting started

Add the MongoDB Vector Store connector NuGet package to your project.

```dotnetcli
dotnet add package Microsoft.SemanticKernel.Connectors.MongoDB --prerelease
```

You can add the vector store to the `IServiceCollection` dependency injection container using extension methods provided by Semantic Kernel.

```csharp
using Microsoft.SemanticKernel;

// Using IServiceCollection with ASP.NET Core.
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMongoDBVectorStore(connectionString, databaseName);
```

Extension methods that take no parameters are also provided. These require an instance of `MongoDB.Driver.IMongoDatabase` to be separately registered with the dependency injection container.

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using MongoDB.Driver;

// Using IServiceCollection with ASP.NET Core.
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IMongoDatabase>(
    sp =>
    {
        var mongoClient = new MongoClient(connectionString);
        return mongoClient.GetDatabase(databaseName);
    });
builder.Services.AddMongoDBVectorStore();
```

You can construct a MongoDB Vector Store instance directly.

```csharp
using Microsoft.SemanticKernel.Connectors.MongoDB;
using MongoDB.Driver;

var mongoClient = new MongoClient(connectionString);
var database = mongoClient.GetDatabase(databaseName);
var vectorStore = new MongoDBVectorStore(database);
```

It is possible to construct a direct reference to a named collection.

```csharp
using Microsoft.SemanticKernel.Connectors.MongoDB;
using MongoDB.Driver;

var mongoClient = new MongoClient(connectionString);
var database = mongoClient.GetDatabase(databaseName);
var collection = new MongoDBVectorStoreRecordCollection<Hotel>(
    database,
    "skhotels");
```

## Data mapping

The MongoDB Vector Store connector provides a default mapper when mapping data from the data model to storage.

This mapper does a direct conversion of the list of properties on the data model to the fields in MongoDB and uses `MongoDB.Bson.Serialization`
to convert to the storage schema. This means that usage of the `MongoDB.Bson.Serialization.Attributes.BsonElement` is supported if a different storage name to the
data model property name is required. The only exception is the key of the record which is mapped to a database field named `_id`, since all MongoDB
records must use this name for ids.

### Property name override

For data properties and vector properties, you can provide override field names to use in storage that is different to the
property names on the data model. This is not supported for keys, since a key has a fixed name in MongoDB.

The property name override is done by setting the `BsonElement` attribute on the data model properties.

Here is an example of a data model with `BsonElement` set.

```csharp
using Microsoft.Extensions.VectorData;

public class Hotel
{
    [VectorStoreRecordKey]
    public ulong HotelId { get; set; }

    [BsonElement("hotel_name")]
    [VectorStoreRecordData(IsFilterable = true)]
    public string HotelName { get; set; }

    [BsonElement("hotel_description")]
    [VectorStoreRecordData(IsFullTextSearchable = true)]
    public string Description { get; set; }

    [BsonElement("hotel_description_embedding")]
    [VectorStoreRecordVector(4, DistanceFunction.CosineSimilarity)]
    public ReadOnlyMemory<float>? DescriptionEmbedding { get; set; }
}
```

::: zone-end
::: zone pivot="programming-language-python"

## Coming soon

More info coming soon.

::: zone-end
::: zone pivot="programming-language-java"

## Coming soon

More info coming soon.

::: zone-end
