# Split Up / Include / Reference external configuration in Serverless Framework file

Functional example of using Serverless Framework [`${file()}`](https://www.serverless.com/framework/docs/providers/aws/guide/variables) variable to load external files into `serverless.yml`.

```yml
service: serverless-framework-include-files
frameworkVersion: "3"

provider: ${file(config/provider.yml):provider}

functions:
  function1:
    handler: index.handler

resources:
  - ${file(resources/s3-bucket.yml)}
  - ${file(resources/dynamodb-table.yml)}
```

## How does `provider: ${file(config/provider.yml):provider}` works?

- The variable `${file(...):<key-name>}` use the `key-name` as the property key in the external file to be included in the `serverless.yml` file. Make sure the key name exists in the external file with a colon `<key-name>:`. In this example we are looking for the `provider` key in the `config/provider.yml` file:

```yml
provider: # <--- key name, it will load the properties of the key name as configuration
  name: aws
  runtime: nodejs14.x
```

- You can include the whole file by removing the key name: `${file(...)}`. Make sure the external file contains the correct configuration for target property in the `serverless.yml` file.

```yml
# no key name, it will load the whole file as configuration values in the serverless.yml file
name: aws
runtime: nodejs14.x
```

## How does `resources:` works?

The `serverless.yml` file supports multiple ways to declare AWS resources. These resources use the [CloudFormation Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) syntax.

```yml
resources:
  Resources:
    # <--- Declare any AWS resources here
```

You can declare [AWS resources inline](#1-aws-resources-inline), [include external files](#2-include-external-files-with-the-resources-configuration) with the resources configuration or use the [array syntax](#3-inline-and-external-files-with-aws-resources) to mix inline and external files.

### 1) AWS resources inline

```yml
resources:
  Resources:
    MyBucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: my-bucket
    MyTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: my-table
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
```

### 2) Include external files with the resources configuration

```yml
resources:
  Resources:
    MyBucket: ${file(resources/s3-bucket2.yml)}
    MyTable: ${file(resources/dynamodb-table2.yml)}
```

### 3) Inline and external files with AWS resources

The secret sauce is the `Resources:` key. It allows you to mix inline and external files.

```yml
resources:
  - ${file(resources/s3-bucket.yml)}
  - Resources:
      MyBucket: ${file(resources/s3-bucket2.yml)}
  - Resources:
      MyTable:
        Type: AWS::DynamoDB::Table
        Properties:
          TableName: ${self:service}
          AttributeDefinitions:
            - AttributeName: id
              AttributeType: S
          KeySchema:
            - AttributeName: id
              KeyType: HASH
          ProvisionedThroughput:
            ReadCapacityUnits: 1
            WriteCapacityUnits: 1
```
