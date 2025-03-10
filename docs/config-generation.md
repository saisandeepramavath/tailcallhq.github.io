---
title: GraphQL Configuration Generation with Tailcall
description: Migrate REST or Protobuff to GraphQL within minutes
slug: graphql-configuration-generation-with-tailcall
sidebar_label: Auto Generation
---

<head>
<meta property="og:type" content="article"/>
  <title>Generate GraphQL Configuration</title>
</head>

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

## What is Configuration Generation?

Configuration generation is the process of automatically generating graphQL configurations from the various sources such as **REST**, **gRPC** and already existing **GraphQL configuration** files.

### Why is it Hard to Write GraphQL Schemas by Hand?

Writing GraphQL schemas manually presents several challenges that can complicate and slow down the development process:

1. **Complex API Responses**:
   - **Large and Detailed Responses**: APIs often return extensive and intricate data, making it laborious to map these responses accurately to GraphQL types.
   - **Nested Structures**: Dealing with deeply nested JSON objects requires careful handling to ensure all relationships and data hierarchies are correctly represented in the schema.
2. **Data Consistency**:
   - **Missing Properties**: APIs can have inconsistent data where some items might lack certain properties, necessitating meticulous examination to define accurate types and optional fields.
   - **Dynamic Data**: Handling APIs with dynamic data fields adds another layer of complexity, requiring flexible and robust schema definitions to accommodate various data shapes.
3. **Migration Efforts**:
   - **Manual Workload**: Converting existing REST APIs or gRPC to GraphQL involves substantial manual effort, such as
     - **Type and Schema Writing**: Each endpoint must be meticulously mapped to corresponding GraphQL types and queries.
     - **Type Merging**: Identifying types that are similar in configuration and merging them into single type is tedious and time taking task and prone to errors.
     - **Duplicate Type**: Identifying and eliminating duplicate in the entire configuration is challenging especially for large scheams, to ensure a clean schema.
     - **Type Naming**: Inferring and naming types manually, which requires a deep understanding of the underlying data structures and their relationships.
   - **Error-Prone Process**: The manual creation of schemas increases the likelihood of errors, leading to potential issues in data fetching and integration.

These challenges highlight the need for automated tools, which streamline the process of generating GraphQL schemas, ensuring accuracy and efficiency while reducing the manual workload and error potential.

For more insights on why manual GraphQL schema writing is becoming obsolete, you can read this [blog post by Tailcall](https://blog.tailcall.run/writing-a-graphql-backend-by-hand-is-long-gone).

## Features

<hr/>

### Effortless REST Integration

Tailcall simplifies GraphQL schema generation from REST APIs, supporting various request types and scenarios. Let's understand this through various examples.

1.  **Simple GET Request:** In the following example, we demonstrate how to generate a GraphQL schema from `https://jsonplaceholder.typicode.com/posts` endpoint.

    This configuration allows Tailcall to fetch data from the specified endpoint and generate a GraphQL schema and save it to output path provided in configuration.

    <Tabs>
    <TabItem value="json" label="JSON Config Format">

    ```json showLineNumbers
    {
      "inputs": [
        {
          "curl": {
            "src": "https://jsonplaceholder.typicode.com/posts",
            "fieldName": "posts",
            "headers": {
              "Accept": "application/json",
              "secretToken": "{{.env.TOKEN}}"
            }
          }
        }
      ],
      "preset": {
        "mergeType": 1.0
      },
      "output": {
        "path": "./jsonplaceholder.graphql",
        "format": "graphQL"
      },
      "schema": {
        "query": "Query"
      }
    }
    ```

    </TabItem>
    <TabItem value="yml" label="YML Config Format">

    ```yml showLineNumbers
    inputs:
      - curl:
          src: "https://jsonplaceholder.typicode.com/posts"
          fieldName: "posts"
          headers:
            Accept: "application/json"
            secretToken: "{{.env.TOKEN}}"
    preset:
      mergeType: 1.0
    output:
      path: "./jsonplaceholder.graphql"
      format: "graphQL"
    schema:
      query: "Query"
    ```

    </TabItem>
    </Tabs>

    Let's understand the above configuration file.

    **Input**: Defines the API endpoints that the configuration interacts with. Each input specifies:

    - **src**: Specifies the endpoint URL (`https://jsonplaceholder.typicode.com/posts`) in this example.
    - **fieldName**: A unique name that should be used as the field name, which is then used in the operation type. In the example above, it's set to `posts`.

      :::important

      Ensure that each **field name** is unique across the entire configuration to prevent overwriting previous definitions.

      :::

    - **headers**: Optional section for specifying HTTP headers required for the API request.
      :::tip
      Never store sensitive information like access tokens directly in configuration files. Leverage templates to securely reference secrets from [environment variables](environment-variables.md).
      :::

    **Preset**: We've applied only one tuning parameter for the configuration. let's understand it in short.

    - We've set [mergeType](config-generation.md#mergetype) to `1.0`, which basically tells config generator to merge any two GraphQL types that are exactly similar.

      if you're interested in understanding preset's in detail head over to [preset](config-generation.md#understanding-presets) section.

    **Output**: Specifies where and in what format the output data should be saved.

    - **path**: Defines the output file path (in above example, it's `./jsonplaceholder.graphql`).
    - **format**: Specifies the output format as GraphQL (in above example, it's `graphQL`).

To generate the GraphQL configuration run following command
<Tabs>
<TabItem value="json" label="JSON Config Format">
`bash
    tailcall gen ./config.json
    `
</TabItem>
<TabItem value="yml" label="YML Config Format">
`bash
    tailcall gen ./config.yml
    `
</TabItem>
</Tabs>
**Schema**: Specifies the name of the Query operation type, which is `Query` in this example.

```graphql showLineNumbers title="Generated GraphQL Configuration"
schema
  @server
  @upstream(
    baseURL: "https://jsonplaceholder.typicode.com"
  ) {
  query: Query
}

type Post {
  body: String
  id: Int
  title: String
  userId: Int
}

type Query {
  posts: [Post] @http(path: "/posts")
}
```

  <hr />

2.  **Simple Post Request**

    In the following example, we demonstrate how to generate a GraphQL schema from `https://jsonplaceholder.typicode.com/posts` endpoint which requires some request body in order to produce the response.

    This configuration allows Tailcall to make a POST request to the upstream API and retrieve the response to generate a GraphQL schema, which is then saved to the output path specified in the configuration.

    <Tabs>
    <TabItem value="json" label="JSON Config Format">
    ```json showLineNumbers
    {
      "inputs":[
        {
          "curl": {
            "src": "https://jsonplaceholder.typicode.com/posts",
            "method": "POST",
            "body": {
              "title": "Tailcall - Modern GraphQL Runtime",
              "body": "Tailcall - Modern GraphQL Runtime",
              "userId": 1
            },
            "headers": {
              "Content-Type": "application/json",
              "Accept": "application/json"
            },
            "isMutation": true,
            "fieldName": "createPost"
          }
        }
      ],
       "preset": {
        "mergeType": 1.0
      },
      "output":{
        "path":"./jsonplaceholder.graphql",
        "format":"graphQL"
      },
      "schema":{
        "mutation":"Mutation"
      }
    }
    ```
    </TabItem>
    <TabItem value="yml" label="YML Config Format">
        ```yml showLineNumbers
        inputs:
          - curl:
              src: "https://jsonplaceholder.typicode.com/posts/1"
              fieldName: "createPost"
              method: "POST"
              isMutation: true
              body:
                title: "Tailcall - Modern GraphQL Runtime"
                body: "Tailcall - Modern GraphQL Runtime"
                userId: 1
              headers:
                Accept: "application/json"
                Content-Type: "application/json"
        preset:
          mergeType: 1.0
        output:
          path: "./jsonplaceholder.graphql"
          format: "graphQL"
        schema:
          query: "Query"        
        ```
    </TabItem>
    </Tabs>

    Let's understand the above configuration file.

    **Input**: Defines the API endpoints that the configuration interacts with. Each input specifies:

    - **src**: Specifies the endpoint URL (https://jsonplaceholder.typicode.com/posts in this example).
    - **fieldName**: A unique name that should be used as the field name, which is then used in the operation type. In the example above, it's set to `createPost`.

      :::important

      Ensure that each **field name** is unique across the entire configuration to prevent overwriting previous definitions.

      :::

    - **headers**: Users can specify the required headers to make the HTTP request in the headers section.

      :::tip
      Never store sensitive information like access tokens directly in configuration files. Leverage templates to securely reference secrets from [environment variables](environment-variables.md).
      :::

    - **body**: This property allows you to specify the request body for methods like POST or PUT. If the endpoint requires a payload, include it here.

    - **method**: Specify the HTTP method for the request (e.g. GET, POST, PUT, DEL). If not provided, the default method is `GET`. in above example, it's set to `POST`.

    - **isMutation**: This flag indicates whether the request should be treated as a GraphQL Mutation. Set `isMutation` to `true` to configure the request as a `Mutation`. If not specified or set to false, the request will be treated as a `Query by default`. in above example it's set to true.

    **Preset**: We've applied only one tuning parameter for the configuration. let's understand it in short.

    - We've set [mergeType](config-generation.md#mergetype) to `1.0`, which basically tells config generator to merge any two GraphQL types that are exactly similar.

      if you're interested in understanding preset's in detail head over to [preset](config-generation.md#understanding-presets) section.

    **Output**: Specifies where and in what format the output data should be saved.

    - **path**: Defines the output file path (in above example, it's `./jsonplaceholder.graphql`).
    - **format**: Specifies the output format as GraphQL (in above example, it's `graphQL`).

To generate the GraphQL configuration run following command
<Tabs>
<TabItem value="json" label="JSON Config Format">

```bash
tailcall gen ./config.json
```

  </TabItem>
  <TabItem value="yml" label="YML Config Format">
    ```bash
    tailcall gen ./config.yml
    ```
  </TabItem>
  </Tabs>

**Schema**: Specifies the operation type. In this example, it's a `Mutation` operation with the name `Mutation`.

```graphql showLineNumbers title="Generated GraphQL Configuration"
schema @server @upstream {
  mutation: Mutation
}

input PostInput {
  body: String
  title: String
  userId: Int
}

type Mutation {
  createPost(createPostInput: PostInput): Post
    @http(
      baseURL: "https://jsonplaceholder.typicode.com"
      body: "{{.args.createPostInput}}"
      method: "POST"
      path: "/posts"
    )
}

type Post {
  body: String
  id: Int
  title: String
  userId: Int
}
```

:::info

This flexible configuration approach allows you to adapt Tailcall for various HTTP methods by modifying key sections like `method`, `body`, `isMutation` and `headers`. Tailcall will handle generating the appropriate GraphQL schema based on the provided API interactions.

:::

### Effortless gRPC Integration

Tailcall simplifies the process of generating GraphQL schemas from gRPC. By specifying the proto file path, Tailcall parses it and generates the corresponding GraphQL types and queries within minutes.

1. **gRPC Integration**: In the following example, we demonstrate how to generate a GraphQL schema from a `news.proto` file.

   This configuration allows Tailcall to parse the proto file, generate a GraphQL schema and save it to the output path provided in the configuration.

    <Tabs>
    <TabItem value="json" label="JSON Config Format">
          ```json showLineNumbers
          {
            "inputs": [
              {
                "proto": {
                  "src": "./news.proto"
                }
              }
            ],
            "preset": {
              "mergeType": 1.0
            },
            "output": {
              "path": "./jsonplaceholder.graphql",
              "format": "graphQL"
            },
            "schema": {
              "query": "Query"
            }
          }
          ```
    </TabItem>

    <TabItem value="yml" label="YML Config Format">
      
      ```yml showLineNumbers
        inputs:
          - proto:
            src: "./news.proto"
        preset:
          mergeType: 1.0
        output:
          path: "./jsonplaceholder.graphql"
          format: "graphQL"
        schema:
          query: "Query"
        ```

    </TabItem>
    </Tabs>

Let's understand the above configuration file.

**Proto**: Defines the path to the proto file that the configuration interacts with.

- **src**: Specifies the path to the proto file (`./news.proto` in this example).

**Preset**: We've applied only one tuning parameter for the configuration. let's understand it in short.

- We've set [mergeType](config-generation.md#mergetype) to `1.0`, which basically tells config generator to merge any two GraphQL types that are exactly similar.

  if you're interested in understanding preset's in detail head over to [preset](config-generation.md#understanding-presets) section.

**Output**: Specifies where and in what format the output data should be saved.

- **path**: Defines the output file path (in above example, it's `./jsonplaceholder.graphql`).
- **format**: Specifies the output format as GraphQL (in above example, it's `graphQL`).

**Schema**: Specifies the name of the Query operation type, which is `Query` in this example.

```graphql showLineNumbers title="Generated GraphQL Configuration"
schema @link(src: "./news.proto", type: Protobuf) @server {
  query: Query
}

type News @tag(id: "news.News") {
  id: Int
  title: String
  content: String
  author: String
}

type Query {
  news: [News] @grpc(method: "news.NewsService.GetNews")
}
```

for more insights on how gPRC works with GraphQL, you can read this [GraphQL over gRPC](grpc.md) article.

### Hybrid Integration (REST + gRPC)

The Configuration Generator with Tailcall supports a hybrid integration of REST and gRPC. This feature allows you to leverage the strengths of both REST APIs and gRPC to create a unified GraphQL schema. By integrating both sources, you can ensure that your GraphQL schema is comprehensive and up-to-date with your existing APIs and data definitions.

#### Example Configuration

Here is an example configuration that demonstrates how to set up a hybrid integration using a REST and gRPC:
<Tabs>
<TabItem value="json" label="JSON Config Format">

```json showLineNumbers
{
  "inputs": [
    {
      "curl": {
        "src": "https://jsonplaceholder.typicode.com/posts",
        "fieldName": "posts"
      }
    },
    {
      "proto": {
        "src": "./news.proto"
      }
    }
  ],
  "preset": {
    "mergeType": 1.0
  },
  "output": {
    "path": "./output.graphql",
    "format": "graphQL"
  },
  "schema": {
    "query": "Query"
  }
}
```

</TabItem>
<TabItem value="yml" label="YML Config Format">
```yaml showLineNumbers
inputs:
  - curl:
      src: "https://jsonplaceholder.typicode.com/posts"
      fieldName: "posts"
  - proto:
      src: "./news.proto"
preset:
  mergeType: 1.0
output:
  path: "./output.graphql"
  format: "graphQL"
schema:
  query: "Query"
```
</TabItem>
</Tabs>

Let's understand the above configuration file.

#### Inputs

- **curl** - section where we can specify the REST endpoint.
  - _src:_ The URL of the REST API endpoint.
  - _fieldName:_ The field name to use in the GraphQL schema for the REST data.
- **proto** - section where we can specify the Proto File.
  - _src:_ The path to the Proto file.

**Preset**: We've applied only one tuning parameter for the configuration. let's understand it in short.

- We've set [mergeType](config-generation.md#mergetype) to `1.0`, which basically tells config generator to merge any two GraphQL types that are exactly similar.

  if you're interested in understanding preset's in detail head over to [preset](config-generation.md#understanding-presets) section.

**Output**: Specifies where and in what format the output data should be saved.

- **path**: Defines the output file path (in above example, it's `./jsonplaceholder.graphql`).
- **format**: Specifies the output format as GraphQL (in above example, it's `graphQL`).

To generate the GraphQL configuration run following command
<Tabs>
<TabItem value="json" label="JSON Config Format">

```bash
tailcall gen ./config.json
```

  </TabItem>
  <TabItem value="yml" label="YML Config Format">
    ```bash
    tailcall gen ./config.yml
    ```
  </TabItem>
  </Tabs>
**Schema**: Specifies the name of the Query operation type, which is `Query` in this example.

```graphql showLineNumbers
schema
  @link(src: "./news.proto", type: Protobuf)
  @upstream(baseURL: "https://jsonplaceholder.typicode.com")
  @server {
  query: Query
}

type News @tag(id: "news.News") {
  id: Int
  title: String
  content: String
  author: String
}

type Post {
  body: String
  id: Int
  title: String
  userId: Int
}

type Query {
  posts: [Post] @http(path: "/posts")
  news: [News] @grpc(method: "news.NewsService.GetNews")
}
```

## Understanding Presets

This section is optional and can be used to generate a more optimized configuration by applying various transformers that improve the config generation process, such as automatically inferring meaningful names of the types, merging duplicate types, removing unused types, and more. If you find that the generated GraphQL configuration is sufficient for your needs, you can skip this section.

The config generator provides a set of tuning parameters that can make the generated configurations more readable by reducing duplication and making configuration more readable. This can be configured using the `preset` section present in configuration.

<Tabs>
<TabItem value="json" label="JSON">
```json showLineNumbers
{
   "preset": {
    "mergeType": 0.8,
    "consolidateURL": 0.8,
    "treeShake": true,
    "unwrapSingleFieldTypes": true,
    "inferTypeNames": true,
  }
}
```
</TabItem>
<TabItem value="yml" label="YML">
```yml showLineNumbers
preset:
  mergeType: 0.8
  consolidateURL: 0.8
  treeShake: true
  unwrapSingleFieldTypes: true
  inferTypeNames: true
```
</TabItem>
</Tabs>

Let's understand how each of the parameter works.

### mergeType

This setting merges types in the configuration that satisfy the threshold criteria. It takes a threshold value between `0.0` and `1.0` to determine if two types should be merged or not. The default is `1.0`. MergeType also supports union types as well as interface types but merging of these types will happen only when they match exactly.

        **Example 1**: following types `T1` and `T2` are exactly similar, and with a threshold value of `1.0`, they can be merged into a single type called `M1`:

        ```graphql {14} showLineNumbers title="Merging type T1 and T2 into M1"
        # BEFORE
        type T1 {
            id: ID
            firstName: String
            lastName: String
        }

        type T2 {
            id: ID
            firstName: String
            lastName: String
        }

        # AFTER: T1 and T2 are merged into M1.
        type M1 {
            id: ID
            firstName: String
            lastName: String
        }
        ```

        **Example 2**: following types `T1` and `T2` are similar with a threshold value of `0.5`, they can be merged into a single type called `M1`:

        ```graphql {14} showLineNumbers title="Merging type T1 and T2 into M1"
        # BEFORE
        type T1 {
            id: ID
            firstName: String
            age: Int
        }

        type T2 {
            id: ID
            firstName: String
            lastName: String
        }

        # AFTER: T1 and T2 are merged into M1.
        type M1 {
            id: ID
            firstName: String
            lastName: String
            age: Int
        }
        ```

        **Example 3**: following types `T1` and `T2` are similar with a threshold value of `0.5` but we can't merge them as they have same field name but different types:

        ```graphql {5,11} showLineNumbers title="Can't Merge type T1 and T2 as they've same field name but different type"
        # BEFORE
        type T1 {
            id: ID
            firstName: String
            age: Int
        }

        type T2 {
            id: ID
            firstName: String
            age: Float
        }
        ```

        **Example 4**: following types `Foo` and `Bar` will be merged into type `M1` as they match exactly and same change will reflected in union type `FooBar`.

        ```graphql {24} showLineNumbers title="Merging type Foo and Bar into M1"
        # BEFORE
        type Foo {
            id: ID
            firstName: String
            age: Int
        }

        type Bar {
            id: ID
            firstName: String
            age: Int
        }

        union FooBar = Foo | Bar

        # After merging

        type M1 {
            id: ID
            firstName: String
            age: Int
        }

        union FooBar = M1
        ```

        **Example 5**: following types `Foo` and `Bar` won't be merged into type `M1` as they don't match exactly.

        ```graphql {5,11} showLineNumbers title="Can't Merge type T1 and T2 as they've same field name but different type"
        # BEFORE
        type Foo {
            id: ID
            firstName: String
            age: Float
        }

        type Bar {
            id: ID
            firstName: String
            age: Int
        }

        union FooBar = Foo | Bar
        ```

        <hr/>

### consolidateURL

The setting identifies the most common base URL among multiple REST endpoints and uses this URL in the [upstream](directives.md#upstream-directive) directive. It takes a threshold value between 0.0 and 1.0 to determine the most common endpoint. The default is `0.5`.

For example, if the `Query` type has three base URLs, using the `consolidateURL` setting with a `0.5` threshold will pick the base URL that is used in more than 50% of the [http](directives.md#http-directive) directives, `http://jsonplaceholder.typicode.com`, and add it to the upstream, cleaning the base URLs from the `Query` type.

```graphql showLineNumbers
schema
  @server(hostname: "0.0.0.0", port: 8000)
  @upstream(httpCache: 42) {
  query: Query
}

type Query {
  post(id: Int!): Post
    @http(
      baseURL: "http://jsonplaceholder.typicode.com"
      path: "/posts/{{.args.id}}"
    )
  posts: [Post]
    @http(
      baseURL: "http://jsonplaceholder.typicode.com"
      path: "/posts"
    )
  user(id: Int!): User
    @http(
      baseURL: "http://jsonplaceholder.typicode.com"
      path: "/users/{{.args.id}}"
    )
  users: [User]
    @http(
      baseURL: "http://jsonplaceholder-1.typicode.com"
      path: "/users"
    )
}
```

After enabling the `consolidateURL` setting:

```graphql showLineNumbers
schema
  @server(hostname: "0.0.0.0", port: 8000)
  @upstream(
    baseURL: "http://jsonplaceholder.typicode.com"
    httpCache: 42
  ) {
  query: Query
}

type Query {
  post(id: Int!): Post @http(path: "/posts/{{.args.id}}")
  posts: [Post] @http(path: "/posts")
  user(id: Int!): User @http(path: "/users/{{.args.id}}")
  users: [User]
    @http(
      baseURL: "http://jsonplaceholder-1.typicode.com"
      path: "/users"
    )
}
```

  <hr />

### unwrapSingleFieldTypes

This setting instructs Tailcall to flatten out types with single field.

for example:

```graphql showLineNumbers
type Query {
  foo: Foo
}

# Type with only one field
type Foo {
  bar: Bar
}

# Type with only one field
type Bar {
  a: Int
}
```

After setting `unwrapSingleFieldTypes` to true:

```graphql showLineNumbers
type Query {
  foo: Int
}
```

This helps in flattening out types into single field.

  <hr />

### treeShake

This setting removes unused types from the configuration. When enabled, any type that is defined in the configuration but not referenced anywhere else (e.g., as a field type, union member, or interface implementation) will be removed. This helps to keep the configuration clean and free from unnecessary definitions.

```graphql showLineNumbers title="Before applying treeShake, the configuration might look like this."
type Query {
  foo: Foo
}

type Foo {
  bar: Bar
}

# Type not used anywhere else
type UnusedType {
  baz: String
}

type Bar {
  a: Int
}
```

```graphql showLineNumbers title="After enabling treeShake, the UnusedType will be removed."
type Query {
  foo: Foo
}

type Foo {
  bar: Bar
}

type Bar {
  a: Int
}
```

  <hr />

### inferTypeNames

The `inferTypeNames` setting aims to enhance type naming consistency and readability by suggesting meaningful type names derived from its usage and shape.

**Heuristic Algorithm**

This is the default algorithm used to infer the name of the types in the configuration.

1.  **Generates Type Names**: Creates type names from field names using pluralization and other heuristics.
2.  **Updates Configuration**: Replaces existing type names with the inferred names and updates all references.

```graphql title="Before enabling inferTypeNames setting"
type T1 {
  id: ID
  name: String
  email: String
  post: [T2]
}

type T2 {
  id: ID
  title: String
  body: String
}

type Query {
  users: [T1] @http(path: "/users")
}
```

- **User**: Derived from T1, since T1 is linked to user data through the users field in the Query type. The new name User clearly indicates the type represents user information.

- **Post**: Derived from T2, since T2 is linked to post data through the post field within User. The new name Post clearly indicates the type represents post information.

```graphql title="After enabling inferTypeNames setting"
type User {
  id: ID
  name: String
  email: String
  post: [Post]
}

type Post {
  id: ID
  title: String
  body: String
}

type Query {
  user: User @http(path: "/users")
}
```

By leveraging field names to derive type names, the schema becomes more intuitive and aligned with the data it represents, making it easier to understand and maintain.

**Additional Considerations:**

- **Priority Handling:** Types directly associated with root operations are given higher priority during inference. For example, if `T2` were associated with a root query or mutation type, it might have a higher priority for inference compared to other types.

- **Pluralization Rules:** The inferred type names are converted to singular form to align with typical GraphQL naming conventions. For instance, a type derived from a plural field name like `comments` would be singularized to `Comment`.

## LLM Powered Inference

This is a more advanced completely opt-in feature. Sometimes it's not possible to infer names correctly based on usage, or a name is not available because its been used already. In such scenarios we leverage LLMs that understand relationships between fields, their schema and other meta information to infer type names. To allow Tailcall to connect to LLMs, you need to provide the API key in the configuration file.

<Tabs>
<TabItem value="json" label="JSON">

```json showLineNumbers
{
  "llm": {
    "model": "gpt-4o",
    "secret": "{{env.LLM_API_KEY}}"
  }
}
```

</TabItem>
<TabItem value="yml" label="YML">

```yaml showLineNumbers
llm:
  model: "gpt-4o"
  secret: "{{env.LLM_API_KEY}}"
```

</TabItem>
</Tabs>

:::tip
Checkout our [LLM](llm.md) section to get a list of all the LLM models that Tailcall supports.
:::

## Best Practices

When setting up your configuration file for GraphQL generation with Tailcall, consider these key parameters to optimize and customize your setup:

1. **[Merge Type](config-generation.md#mergetype)**:
   Controls the merging of similar GraphQL types to reduce duplication. Adjust the threshold (0.0 to 1.0) based on how strictly you want types to match for merging.
   the closer the number to 1.0, you get the best type inference in graphQL playground. Recommended threshold is anything above `0.9`.

    <Tabs>
    <TabItem value="json" label="JSON">
    ```json showLineNumbers
    {
      "preset": {
          "mergeType": 0.9
      }
    }
    ```
    </TabItem>
    <TabItem value="yml" label="YML">
    ```yml showLineNumbers
    preset:
        mergeType: 0.9
    ```
    </TabItem>
    </Tabs>

2. **[Consolidate URL](config-generation.md#consolidateurl)**: Identifies the most common base URL among multiple REST endpoints and uses it in the [@upstream](directives.md#upstream-directive) directive. Set a threshold (0.0 to 1.0) to determine when to consolidate URLs. Recommended threshold is anything above `0.5`.
   <Tabs>
   <TabItem value="json" label="JSON">
   ```json showLineNumbers
   {
     "preset": {
       "consolidateURL": 0.5
     }
   }
   ```
   </TabItem>
   <TabItem value="yml" label="YML">
   ```yml showLineNumbers
   preset:
       consolidateURL: 0.5
   ```
   </TabItem>
   </Tabs>
3. **Headers**: Never store sensitive information like access tokens directly in configuration files. Leverage templates to securely reference secrets from [environment variables](environment-variables.md).
   <Tabs>
   <TabItem value="json" label="JSON">
   ```json showLineNumbers
   {
     "headers": {
       "secretToken": "{{.env.TOKEN}}"
     }
   }
   ```
   </TabItem>
   <TabItem value="yml" label="YML">
   ```yml showLineNumbers
   headers:
       secretToken: "{{.env.TOKEN}}"
   ```
   </TabItem>
   </Tabs>

## FAQ's

**Q. Can I use environment variables in my configuration?**

**Answer:** Yes, you can use [environment variables](environment-variables.md) to securely reference sensitive information like access tokens. Here is an example:

  <Tabs>
  <TabItem value="json" label="JSON">
  ```json showLineNumbers
  {
    "curl": {
      "src": "https://jsonplaceholder.typicode.com/posts/1",
      "fieldName": "post",
      "headers": {
        "secretToken": "{{.env.TOKEN}}"
      }
    }
  }
  ```
  </TabItem>
  <TabItem value="yml" label="YML">
```yml showLineNumbers
curl:
  src: "https://jsonplaceholder.typicode.com/posts/1"
  fieldName: "post"
  headers:
      secretToken: "{{.env.TOKEN}}"
  ```
  </TabItem>
  </Tabs>

**Q. How do I merge similar types in the configuration?**

**Answer:** Adjust the [mergeType](config-generation.md#mergetype) parameter in the preset section to control the merging of similar types. A threshold value between 0.0 and 1.0 determines if two types should be merged or not. if you to understand this in detail then please head over to [preset](config-generation.md#understanding-presets) section. Here is an example:
<Tabs>
<TabItem value="json" label="JSON">

```json showLineNumbers
{
  "preset": {
    "mergeType": 0.9
  }
}
```

  </TabItem>
  <TabItem value="yml" label="YML">
  ```yml showLineNumbers
  preset:
      mergeType: 0.9
  ```
  </TabItem>
  </Tabs>

**Q. What if I have multiple REST endpoints with different base URLs?**

**Answer:** Use the [consolidateURL](config-generation.md#consolidateurl) parameter to identify the most common base URL among multiple REST endpoints and it will automatically select the most common base url and add it to the [@upstream](directives.md#upstream-directive) directive. Here is an example:

  <Tabs>
  <TabItem value="json" label="JSON">
  ```json showLineNumbers
  {
    "preset": {
      "consolidateURL": 0.5
    }
  }
  ```
  </TabItem>
  <TabItem value="yml" label="YML">
  ```yml showLineNumbers
  preset:
      consolidateURL: 0.5
  ```
  </TabItem>
  </Tabs>

**Q. Can I specify multiple input sources in a single configuration?**

**Answer:** Yes, you can specify multiple input sources, such as different REST endpoints or Proto files, in a single configuration. Here is an example:

<Tabs>
<TabItem value="json" label="JSON">

```json showLineNumbers
{
  "inputs": [
    {
      "curl": {
        "src": "https://jsonplaceholder.typicode.com/posts",
        "fieldName": "posts"
      }
    },
    {
      "proto": {
        "src": "./news.proto"
      }
    }
  ],
  "schema": {
    "query": "Query"
  }
}
```

</TabItem>

<TabItem value="yml" label="YML">

```yml showLineNumbers
inputs:
  - curl:
      src: "https://jsonplaceholder.typicode.com/posts"
      fieldName: "posts"
  - proto:
      src: "./news.proto"
schema:
  query: "Query"
```

</TabItem>
</Tabs>
