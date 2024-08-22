# Comprehensive Tutorial: MS-SQL Embeddings and Embedding Generators

## Table of Contents

1. [Introduction](#introduction)
2. [What are Embeddings?](#what-are-embeddings)
3. [Why Use Embeddings for MS-SQL Schemas?](#why-use-embeddings-for-ms-sql-schemas)
4. [The IEmbeddingGenerator Interface](#the-iembeddinggenerator-interface)
5. [WordIndex Class](#wordindex-class)
6. [Detailed Process of Creating Embeddings](#detailed-process-of-creating-embeddings)
7. [Embedding Generators](#embedding-generators)
   - [EnhancedEmbeddingGenerator](#enhancedembeddinggenerator)
   - [PrimaryKeyAwareEmbeddingGenerator](#primarykeyawareembeddinggenerator)
   - [ForeignKeyAwareEmbeddingGenerator](#foreignkeyawareembeddinggenerator)
   - [MLNetEmbeddingGenerator](#mlnetembeddinggenerator)
8. [Combining Embedding Generators](#combining-embedding-generators)
9. [Using Embeddings for Semantic Search](#using-embeddings-for-semantic-search)
10. [Conclusion and Next Steps](#conclusion-and-next-steps)

## Introduction

This comprehensive tutorial explores the concept of embeddings for MS-SQL database schemas and provides an in-depth look at various embedding generators implemented in our project. We'll focus on explaining each generator, how to build them from scratch, and the detailed process of creating embeddings.

## What are Embeddings?

Embeddings are dense vector representations of data in a continuous vector space. Each dimension of the embedding vector corresponds to a feature of the data, learned from patterns in a large dataset. 

Key properties of embeddings:
1. Similar items are closer together in the embedding space.
2. The relative positions of items can capture semantic relationships.
3. Embeddings can be used as input features for machine learning models.

## Why Use Embeddings for MS-SQL Schemas?

Applying embeddings to MS-SQL schemas offers several advantages:

1. **Semantic Understanding**: Capture the meaning of table names, column names, and relationships.
2. **Efficient Similarity Comparisons**: Allow for quick similarity calculations between schema elements.
3. **Feature Input for ML Models**: Use as input for tasks like schema matching or query recommendation.
4. **Dimensionality Reduction**: Compress complex schema information into a fixed-size vector.

## WordIndex Class

The `WordIndex` class maintains a bidirectional mapping between words and their indices:

```csharp
public class WordIndex
{
    // Class implementation...
}
```

Key methods:
- `GetOrAddWord(string word)`: Returns the index for a given word, adding it if it doesn't exist.
- `GetWord(int index)`: Returns the word corresponding to a given index.
- `Count`: Returns the total number of unique words in the index.

## The IEmbeddingGenerator Interface

Our project defines an `IEmbeddingGenerator` interface to ensure consistency across different embedding generation strategies:

```csharp
public interface IEmbeddingGenerator
{
    float[] GenerateEmbedding(string text, List<string> entities, string primaryKey, List<string> foreignKeys);
}
```

## Detailed Process of Creating Embeddings

1. **Understanding the Input**: Start with a SQL schema text.

2. **Text Preprocessing**:
   - Convert to lowercase
   - Tokenization: Split into individual words or tokens
   - Remove punctuation
   - (Optional) Remove stop words

3. **Creating a Vocabulary**:
   - Iterate through all preprocessed tokens
   - Add each unique token to the vocabulary
   - Assign a unique index to each token

4. **Basic Embedding Creation**:
   - Initialize a zero vector of a chosen size
   - For each word in the schema:
     - Look up its index in the vocabulary
     - Set the corresponding position in the vector to 1 (or increment it)

5. **Incorporating Schema Structure**:
   - Identify important elements: primary keys, foreign keys, data types
   - Assign higher weights to these elements

6. **Handling Relationships**:
   - Identify foreign key declarations
   - Increase weights for both the foreign key and the referenced primary key
   - Add weights to words indicating relationships

7. **Normalization**:
   - Calculate the magnitude of the vector
   - Divide each element by this magnitude

8. **Combining Multiple Approaches**:
   - Generate embeddings using each method
   - Assign a weight to each method
   - Multiply each embedding by its weight
   - Sum all the weighted embeddings
   - Normalize the result

9. **Using ML.NET** (for MLNetEmbeddingGenerator):
   - Convert the schema text into a `TextData` object
   - Pass this through a pre-trained text featurization pipeline
   - Extract the resulting feature vector as the embedding

## Embedding Generators

Our project employs a multi-embedding approach, utilizing several different embedding generators to create a more comprehensive and nuanced representation of SQL schemas. Each generator focuses on different aspects of the schema, allowing us to capture various semantic and structural elements more effectively.

The multi-embedding approach works as follows:
1. We apply each embedding generator to the input schema.
2. Each generator produces its own embedding vector.
3. These embeddings are then combined using a weighted sum.
4. The final combined embedding is normalized to produce the ultimate representation of the schema.

This approach allows us to leverage the strengths of each generator while mitigating their individual weaknesses. Now, let's look at each embedding generator in detail:

### EnhancedEmbeddingGenerator

The `EnhancedEmbeddingGenerator` creates embeddings that consider the importance of primary keys, foreign keys, and entities.

Key features:
- Uses `WordIndex` to efficiently map words to unique indices.
- Applies differential weighting to various schema elements:
  - Basic words receive a weight of 1.
  - Words in the primary key receive an additional weight of 3.
  - Words in foreign keys receive an additional weight of 2.
  - Entity words receive an additional weight of 4.
- Processes the primary key separately, giving it an extra weight of 5.
- Processes foreign keys separately, giving each an extra weight of 3.
- Employs a sliding window approach, where each word's weight affects nearby positions in the embedding vector.
- Normalizes the final embedding to unit length for consistent comparison.

### PrimaryKeyAwareEmbeddingGenerator

This document outlines the complete process of generating an embedding using the `PrimaryKeyAwareEmbeddingGenerator`, from database querying to the final embedding vector.

#### 1. Database Querying

1.1. Connect to the MS-SQL database using a database connection string.

1.2. Query the system tables to retrieve schema information:
```sql
SELECT 
    t.name AS TableName,
    c.name AS ColumnName,
    ty.name AS DataType,
    c.is_nullable AS IsNullable,
    c.is_identity AS IsIdentity,
    CASE WHEN pk.column_id IS NOT NULL THEN 1 ELSE 0 END AS IsPrimaryKey,
    CASE WHEN fk.parent_column_id IS NOT NULL THEN 1 ELSE 0 END AS IsForeignKey
FROM 
    sys.tables t
INNER JOIN 
    sys.columns c ON t.object_id = c.object_id
INNER JOIN 
    sys.types ty ON c.user_type_id = ty.user_type_id
LEFT JOIN 
    sys.index_columns pk ON t.object_id = pk.object_id AND c.column_id = pk.column_id AND pk.index_id = 1
LEFT JOIN 
    sys.foreign_key_columns fk ON t.object_id = fk.parent_object_id AND c.column_id = fk.parent_column_id
ORDER BY 
    t.name, c.column_id;
```

1.3. Store the retrieved schema information in a suitable data structure (e.g., a list of custom `TableSchema` objects).

#### 2. Schema Text Generation

2.1. For each table in the retrieved schema:
   a. Generate a CREATE TABLE statement based on the column information.
   b. Include PRIMARY KEY constraints.
   c. Include FOREIGN KEY constraints (queried separately if needed).

2.2. Combine all generated CREATE TABLE statements into a single string.

#### 3. Text Preprocessing

3.1. Convert the entire schema text to lowercase.

3.2. Tokenize the text:
   a. Split on whitespace.
   b. Treat special characters (e.g., parentheses, commas) as separate tokens.

3.3. Remove any empty tokens.

#### 4. Vocabulary Building

4.1. Initialize a `WordIndex` object.

4.2. Iterate through all tokens:
   a. For each unique token, call `wordIndex.GetOrAddWord(token)`.
   b. This assigns a unique index to each word and builds our vocabulary.

#### 5. Embedding Generation

5.1. Initialize a zero vector of size `embeddingSize` (e.g., 3072).

5.2. Iterate through the tokens again:
   a. Get the index for each token using `wordIndex.GetOrAddWord(token)`.
   b. Update the embedding vector:
      - For regular words: `embedding[index % embeddingSize] += 1`
      - For primary key words: `embedding[index % embeddingSize] += 10`
      - For foreign key words: `embedding[index % embeddingSize] += 3`

5.3. Process the primary key separately:
   a. Get the index for the primary key: `primaryKeyIndex = wordIndex.GetOrAddWord(primaryKey)`
   b. Add extra weight: `embedding[primaryKeyIndex % embeddingSize] += 15`

5.4. Process primary key related words:
   a. For each word in ["primary", "key", "id", "identifier"]:
      - Get the word index
      - Add weight: `embedding[wordIndex % embeddingSize] += 5`

5.5. Process common primary key data types:
   a. For each type in ["int", "bigint", "uuid", "guid"]:
      - If the type is in the schema text:
        - Get the type index
        - Add weight: `embedding[typeIndex % embeddingSize] += 3`

5.6. Process entities (table names, column names):
   a. For each entity:
      - Get the entity index
      - Add weight: `embedding[entityIndex % embeddingSize] += 2`

#### 6. Embedding Normalization

6.1. Calculate the magnitude of the embedding vector:
   ```csharp
   float magnitude = (float)Math.Sqrt(embedding.Sum(x => x * x));
   ```

6.2. Normalize the embedding:
   ```csharp
   for (int i = 0; i < embedding.Length; i++)
   {
       embedding[i] /= magnitude;
   }
   ```

#### 7. Return the Embedding

7.1. Return the normalized embedding vector.

This embedding can now be used for various downstream tasks such as similarity comparison, clustering, or as input to machine learning models.

### ForeignKeyAwareEmbeddingGenerator

The `ForeignKeyAwareEmbeddingGenerator` is a crucial component in our MS-SQL schema embedding system that creates vector representations of database schemas with a special focus on foreign key relationships. This generator recognizes the importance of foreign keys in defining relationships between tables and structuring relational databases. It processes the schema information, giving extra weight to foreign key-related elements, to produce a comprehensive embedding that captures the interconnected nature of the database structure. This document outlines the complete process, from initial database querying to the generation of the final embedding vector, demonstrating how the generator transforms raw schema data into a meaningful numerical representation that emphasizes table relationships.

#### 1. Database Querying

1.1. Connect to the MS-SQL database using a database connection string.

1.2. Query the system tables to retrieve schema information, including foreign key relationships:
```sql
SELECT 
    t.name AS TableName,
    c.name AS ColumnName,
    ty.name AS DataType,
    c.is_nullable AS IsNullable,
    c.is_identity AS IsIdentity,
    CASE WHEN pk.column_id IS NOT NULL THEN 1 ELSE 0 END AS IsPrimaryKey,
    CASE WHEN fk.parent_column_id IS NOT NULL THEN 1 ELSE 0 END AS IsForeignKey,
    fk_t.name AS ReferencedTable,
    fk_c.name AS ReferencedColumn
FROM 
    sys.tables t
INNER JOIN 
    sys.columns c ON t.object_id = c.object_id
INNER JOIN 
    sys.types ty ON c.user_type_id = ty.user_type_id
LEFT JOIN 
    sys.index_columns pk ON t.object_id = pk.object_id AND c.column_id = pk.column_id AND pk.index_id = 1
LEFT JOIN 
    sys.foreign_key_columns fk ON t.object_id = fk.parent_object_id AND c.column_id = fk.parent_column_id
LEFT JOIN
    sys.tables fk_t ON fk.referenced_object_id = fk_t.object_id
LEFT JOIN
    sys.columns fk_c ON fk.referenced_object_id = fk_c.object_id AND fk.referenced_column_id = fk_c.column_id
ORDER BY 
    t.name, c.column_id;
```

1.3. Store the retrieved schema information in a suitable data structure (e.g., a list of custom `TableSchema` objects), ensuring foreign key relationships are properly represented.

#### 2. Schema Text Generation

2.1. For each table in the retrieved schema:
   a. Generate a CREATE TABLE statement based on the column information.
   b. Include PRIMARY KEY constraints.
   c. Include FOREIGN KEY constraints, specifying the referenced tables and columns.

2.2. Combine all generated CREATE TABLE statements into a single string.

#### 3. Text Preprocessing

3.1. Convert the entire schema text to lowercase.

3.2. Tokenize the text:
   a. Split on whitespace.
   b. Treat special characters (e.g., parentheses, commas) as separate tokens.

3.3. Remove any empty tokens.

#### 4. Vocabulary Building

4.1. Initialize a `WordIndex` object.

4.2. Iterate through all tokens:
   a. For each unique token, call `wordIndex.GetOrAddWord(token)`.
   b. This assigns a unique index to each word and builds our vocabulary.

#### 5. Embedding Generation

5.1. Initialize a zero vector of size `embeddingSize` (e.g., 3072).

5.2. Iterate through the tokens again:
   a. Get the index for each token using `wordIndex.GetOrAddWord(token)`.
   b. Update the embedding vector:
      - For regular words: `embedding[index % embeddingSize] += 1`
      - For foreign key words: `embedding[index % embeddingSize] += 10`
      - For primary key words: `embedding[index % embeddingSize] += 3`

5.3. Process foreign keys separately:
   a. For each foreign key:
      - Get the index for the foreign key column: `foreignKeyIndex = wordIndex.GetOrAddWord(foreignKey)`
      - Add extra weight: `embedding[foreignKeyIndex % embeddingSize] += 15`
      - Get the index for the referenced table and column
      - Add weight to referenced elements: `embedding[referencedIndex % embeddingSize] += 5`

5.4. Process foreign key related words:
   a. For each word in ["foreign", "key", "references", "constraint"]:
      - Get the word index
      - Add weight: `embedding[wordIndex % embeddingSize] += 5`

5.5. Process relationship words:
   a. For each word in ["on", "delete", "cascade", "set", "null", "update"]:
      - If the word is in the schema text:
        - Get the word index
        - Add weight: `embedding[wordIndex % embeddingSize] += 3`

5.6. Process common join column patterns:
   a. For each pattern in ["_id", "Id", "_fk", "Fk"]:
      - If any foreign key contains this pattern:
        - Get the pattern index
        - Add weight: `embedding[patternIndex % embeddingSize] += 4`

5.7. Process entities (table names, column names):
   a. For each entity:
      - Get the entity index
      - Add weight: `embedding[entityIndex % embeddingSize] += 2`

#### 6. Embedding Normalization

6.1. Calculate the magnitude of the embedding vector:
   ```csharp
   float magnitude = (float)Math.Sqrt(embedding.Sum(x => x * x));
   ```

6.2. Normalize the embedding:
   ```csharp
   for (int i = 0; i < embedding.Length; i++)
   {
       embedding[i] /= magnitude;
   }
   ```

#### 7. Return the Embedding

7.1. Return the normalized embedding vector.

This embedding can now be used for various downstream tasks such as similarity comparison, clustering, or as input to machine learning models, with a particular strength in capturing and representing foreign key relationships within the database schema.

### MLNetEmbeddingGenerator

The `MLNetEmbeddingGenerator` is an advanced component in our MS-SQL schema embedding system that leverages the power of Microsoft's ML.NET framework to create sophisticated vector representations of database schemas. This generator utilizes machine learning techniques to capture complex patterns and relationships within the schema, going beyond simple keyword matching. It processes the schema information using pre-trained models and advanced text featurization techniques to produce a comprehensive embedding that captures both the structure and semantics of the database schema. This document outlines the complete process, from initial database querying to the generation of the final embedding vector, demonstrating how the generator transforms raw schema data into a high-dimensional, learned representation.

#### 1. Database Querying

1.1. Connect to the MS-SQL database using a database connection string.

1.2. Query the system tables to retrieve schema information:
```sql
SELECT 
    t.name AS TableName,
    c.name AS ColumnName,
    ty.name AS DataType,
    c.is_nullable AS IsNullable,
    c.is_identity AS IsIdentity,
    CASE WHEN pk.column_id IS NOT NULL THEN 1 ELSE 0 END AS IsPrimaryKey,
    CASE WHEN fk.parent_column_id IS NOT NULL THEN 1 ELSE 0 END AS IsForeignKey,
    fk_t.name AS ReferencedTable,
    fk_c.name AS ReferencedColumn
FROM 
    sys.tables t
INNER JOIN 
    sys.columns c ON t.object_id = c.object_id
INNER JOIN 
    sys.types ty ON c.user_type_id = ty.user_type_id
LEFT JOIN 
    sys.index_columns pk ON t.object_id = pk.object_id AND c.column_id = pk.column_id AND pk.index_id = 1
LEFT JOIN 
    sys.foreign_key_columns fk ON t.object_id = fk.parent_object_id AND c.column_id = fk.parent_column_id
LEFT JOIN
    sys.tables fk_t ON fk.referenced_object_id = fk_t.object_id
LEFT JOIN
    sys.columns fk_c ON fk.referenced_object_id = fk_c.object_id AND fk.referenced_column_id = fk_c.column_id
ORDER BY 
    t.name, c.column_id;
```

1.3. Store the retrieved schema information in a suitable data structure (e.g., a list of custom `TableSchema` objects).

#### 2. Schema Text Generation

2.1. For each table in the retrieved schema:
   a. Generate a CREATE TABLE statement based on the column information.
   b. Include PRIMARY KEY constraints.
   c. Include FOREIGN KEY constraints, specifying the referenced tables and columns.

2.2. Combine all generated CREATE TABLE statements into a single string.

#### 3. ML.NET Setup

3.1. Initialize an ML.NET context:
```csharp
var mlContext = new MLContext(seed: 0);
```

3.2. Define the input data schema:
```csharp
public class SchemaInput
{
    public string Text { get; set; }
}
```

3.3. Create a text featurization pipeline:
```csharp
var pipeline = mlContext.Transforms.Text.NormalizeText("Text")
    .Append(mlContext.Transforms.Text.TokenizeIntoWords("Tokens", "Text"))
    .Append(mlContext.Transforms.Text.RemoveDefaultStopWords("Tokens"))
    .Append(mlContext.Transforms.Conversion.MapValueToKey("Tokens"))
    .Append(mlContext.Transforms.Text.ProduceNgrams("Ngrams", "Tokens"))
    .Append(mlContext.Transforms.Conversion.MapValueToKey("Ngrams"))
    .Append(mlContext.Transforms.Text.ProduceWordEmbeddings("Features", "Tokens", 
        WordEmbeddingEstimator.PretrainedModelKind.GloVe100D));
```

#### 4. Embedding Generation

4.1. Create an input data view:
```csharp
var data = mlContext.Data.LoadFromEnumerable(new[] 
{
    new SchemaInput { Text = schemaText }
});
```

4.2. Fit the pipeline to the data and transform:
```csharp
var model = pipeline.Fit(data);
var transformedData = model.Transform(data);
```

4.3. Extract the embedding:
```csharp
var embedding = mlContext.Data.CreateEnumerable<TransformedSchema>(transformedData, reuseRowObject: false)
    .First()
    .Features;
```

#### 5. Post-processing

5.1. (Optional) Dimensionality Reduction:
   If the embedding dimension is too high, apply techniques like PCA:
```csharp
var pcaPipeline = mlContext.Transforms.ReduceDimensions("ReducedFeatures", "Features", 
    rank: desiredDimension, approximationRank: desiredDimension);
var pcaModel = pcaPipeline.Fit(transformedData);
var reducedData = pcaModel.Transform(transformedData);
embedding = mlContext.Data.CreateEnumerable<ReducedSchema>(reducedData, reuseRowObject: false)
    .First()
    .ReducedFeatures;
```

5.2. Normalization:
```csharp
float magnitude = (float)Math.Sqrt(embedding.Sum(x => x * x));
for (int i = 0; i < embedding.Length; i++)
{
    embedding[i] /= magnitude;
}
```

#### 6. Return the Embedding

6.1. Return the normalized embedding vector.

This ML.NET-generated embedding captures complex patterns and semantic relationships within the schema text, potentially offering insights that simpler bag-of-words models might miss.


By combining these diverse embedding generators, our multi-embedding approach can capture a wide range of semantic and structural information from SQL schemas. This comprehensive representation enhances the accuracy and effectiveness of downstream tasks such as semantic search and schema matching.
