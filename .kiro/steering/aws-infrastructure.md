# AWS Infrastructure Guidelines

## Infrastructure as Code (IaC)

### AWS CDK (TypeScript)

All infrastructure MUST be defined using AWS CDK in TypeScript.

**Why CDK?**
- Type-safe infrastructure
- Reusable constructs
- Native TypeScript support
- Better than CloudFormation templates
- Easier testing

### CDK Project Structure

```
backend/
├── infrastructure/
│   ├── bin/
│   │   └── app.ts              # CDK app entry point
│   ├── lib/
│   │   ├── stacks/
│   │   │   ├── auth-stack.ts
│   │   │   ├── database-stack.ts
│   │   │   ├── api-stack.ts
│   │   │   ├── storage-stack.ts
│   │   │   ├── jobs-stack.ts
│   │   │   └── frontend-stack.ts
│   │   └── constructs/
│   │       ├── lambda-function.ts
│   │       └── graphql-api.ts
│   ├── test/
│   │   └── stacks.test.ts
│   ├── cdk.json
│   └── tsconfig.json
└── src/                        # Lambda handlers
```

## Stack Organization

### Separate Stacks by Concern

**AuthStack** - Authentication & Authorization
- Cognito User Pool
- Identity Pool
- OAuth providers

**DatabaseStack** - Data Storage
- RDS PostgreSQL
- DynamoDB tables
- VPC (if needed)

**ApiStack** - GraphQL API
- AppSync API
- Lambda resolvers
- VTL resolvers
- Direct resolvers

**StorageStack** - File Storage & CDN
- S3 buckets
- CloudFront distributions

**JobsStack** - Background Jobs
- Step Functions
- SQS queues
- EventBridge rules

**FrontendStack** - Web Application
- Lambda@Edge functions
- CloudFront distribution
- S3 bucket (static assets)

### Stack Dependencies

```typescript
// bin/app.ts
import * as cdk from 'aws-cdk-lib'
import { AuthStack } from '../lib/stacks/auth-stack'
import { DatabaseStack } from '../lib/stacks/database-stack'
import { ApiStack } from '../lib/stacks/api-stack'

const app = new cdk.App()

const authStack = new AuthStack(app, 'KazeAuthStack')
const databaseStack = new DatabaseStack(app, 'KazeDatabaseStack')

const apiStack = new ApiStack(app, 'KazeApiStack', {
  userPool: authStack.userPool,
  database: databaseStack.database,
})
```

## Resource Naming Convention

Format: `{project}-{environment}-{service}-{resource}`

**Examples:**
- `kaze-prod-api-graphql`
- `kaze-prod-db-postgres`
- `kaze-prod-storage-images`
- `kaze-prod-lambda-scraper`

**Code:**
```typescript
const bucket = new s3.Bucket(this, 'ImagesBucket', {
  bucketName: `kaze-${environment}-storage-images`,
})
```

## Cross-Stack Communication

### SSM Parameters (Recommended)

```typescript
// In DatabaseStack
new ssm.StringParameter(this, 'DbEndpoint', {
  parameterName: '/kaze/database/endpoint',
  stringValue: database.clusterEndpoint.hostname,
})

// In ApiStack
const dbEndpoint = ssm.StringParameter.valueFromLookup(
  this,
  '/kaze/database/endpoint'
)
```

### CloudFormation Exports

```typescript
// In DatabaseStack
new cdk.CfnOutput(this, 'DatabaseArn', {
  value: database.clusterArn,
  exportName: 'KazeDatabaseArn',
})

// In ApiStack
const dbArn = cdk.Fn.importValue('KazeDatabaseArn')
```

### Direct References (Preferred)

```typescript
// Pass resources directly between stacks
const apiStack = new ApiStack(app, 'ApiStack', {
  database: databaseStack.database,
  userPool: authStack.userPool,
})
```

## Lambda Functions

### Function Structure

```typescript
import * as lambda from 'aws-cdk-lib/aws-lambda'
import * as nodejs from 'aws-cdk-lib/aws-lambda-nodejs'

const scraperFunction = new nodejs.NodejsFunction(this, 'ScraperFunction', {
  functionName: 'kaze-prod-scraper-handler',
  entry: 'src/handlers/scraper.ts',
  handler: 'handler',
  runtime: lambda.Runtime.NODEJS_20_X,
  timeout: cdk.Duration.seconds(30),
  memorySize: 512,
  environment: {
    DATABASE_URL: databaseUrl,
    S3_BUCKET: bucket.bucketName,
  },
  bundling: {
    minify: true,
    sourceMap: true,
    externalModules: ['aws-sdk'],
  },
})
```

### Lambda Best Practices

1. **Timeout**: Set appropriate timeout (default 3s, max 15min)
2. **Memory**: Start with 512MB, adjust based on metrics
3. **Environment Variables**: Use for configuration
4. **Bundling**: Minify and tree-shake
5. **Layers**: Use for shared dependencies
6. **VPC**: Only if needed (adds cold start time)

### Lambda Permissions

```typescript
// Grant S3 access
bucket.grantReadWrite(scraperFunction)

// Grant DynamoDB access
table.grantReadWriteData(scraperFunction)

// Grant Secrets Manager access
secret.grantRead(scraperFunction)

// Custom policy
scraperFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['ses:SendEmail'],
  resources: ['*'],
}))
```

## AppSync GraphQL API

### API Definition

```typescript
import * as appsync from 'aws-cdk-lib/aws-appsync'

const api = new appsync.GraphqlApi(this, 'Api', {
  name: 'kaze-prod-api-graphql',
  schema: appsync.SchemaFile.fromAsset('schema.graphql'),
  authorizationConfig: {
    defaultAuthorization: {
      authorizationType: appsync.AuthorizationType.USER_POOL,
      userPoolConfig: {
        userPool: props.userPool,
      },
    },
    additionalAuthorizationModes: [
      {
        authorizationType: appsync.AuthorizationType.IAM,
      },
    ],
  },
  logConfig: {
    fieldLogLevel: appsync.FieldLogLevel.ERROR,
  },
})
```

### Lambda Resolvers

```typescript
const dataSource = api.addLambdaDataSource(
  'MangaDataSource',
  getMangaFunction
)

dataSource.createResolver('GetMangaResolver', {
  typeName: 'Query',
  fieldName: 'getManga',
})
```

### VTL Resolvers

```typescript
const dynamoDataSource = api.addDynamoDbDataSource(
  'CacheDataSource',
  cacheTable
)

dynamoDataSource.createResolver('GetCachedMangaResolver', {
  typeName: 'Query',
  fieldName: 'getCachedManga',
  requestMappingTemplate: appsync.MappingTemplate.fromString(`
    {
      "version": "2018-05-29",
      "operation": "GetItem",
      "key": {
        "id": $util.dynamodb.toDynamoDBJson($ctx.args.id)
      }
    }
  `),
  responseMappingTemplate: appsync.MappingTemplate.fromString(`
    $util.toJson($ctx.result)
  `),
})
```

## RDS PostgreSQL

### Database Cluster

```typescript
import * as rds from 'aws-cdk-lib/aws-rds'
import * as ec2 from 'aws-cdk-lib/aws-ec2'

const vpc = new ec2.Vpc(this, 'Vpc', {
  maxAzs: 2,
  natGateways: 1,
})

const cluster = new rds.DatabaseCluster(this, 'Database', {
  engine: rds.DatabaseClusterEngine.auroraPostgres({
    version: rds.AuroraPostgresEngineVersion.VER_15_3,
  }),
  credentials: rds.Credentials.fromGeneratedSecret('postgres'),
  instanceProps: {
    vpc,
    vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
    instanceType: ec2.InstanceType.of(
      ec2.InstanceClass.T4G,
      ec2.InstanceSize.MEDIUM
    ),
  },
  defaultDatabaseName: 'kaze',
  backup: {
    retention: cdk.Duration.days(7),
  },
  storageEncrypted: true,
})
```

### Database Migrations

```typescript
// Custom resource to run migrations
const migrationFunction = new nodejs.NodejsFunction(this, 'MigrationFunction', {
  entry: 'src/migrations/handler.ts',
  vpc,
  environment: {
    DATABASE_URL: cluster.clusterEndpoint.socketAddress,
  },
})

cluster.secret!.grantRead(migrationFunction)

new cdk.CustomResource(this, 'MigrationResource', {
  serviceToken: migrationFunction.functionArn,
})
```

## DynamoDB

### Table Definition

```typescript
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb'

const cacheTable = new dynamodb.Table(this, 'CacheTable', {
  tableName: 'kaze-prod-cache',
  partitionKey: { name: 'pk', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'sk', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  timeToLiveAttribute: 'ttl',
  pointInTimeRecovery: true,
  encryption: dynamodb.TableEncryption.AWS_MANAGED,
})

// Global Secondary Index
cacheTable.addGlobalSecondaryIndex({
  indexName: 'GSI1',
  partitionKey: { name: 'gsi1pk', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'gsi1sk', type: dynamodb.AttributeType.STRING },
})
```

## S3 & CloudFront

### S3 Bucket

```typescript
import * as s3 from 'aws-cdk-lib/aws-s3'

const imagesBucket = new s3.Bucket(this, 'ImagesBucket', {
  bucketName: 'kaze-prod-manga-images',
  encryption: s3.BucketEncryption.S3_MANAGED,
  blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
  versioned: false,
  lifecycleRules: [
    {
      expiration: cdk.Duration.days(365),
      transitions: [
        {
          storageClass: s3.StorageClass.INFREQUENT_ACCESS,
          transitionAfter: cdk.Duration.days(90),
        },
      ],
    },
  ],
})
```

### CloudFront Distribution

```typescript
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront'
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins'

const distribution = new cloudfront.Distribution(this, 'Distribution', {
  defaultBehavior: {
    origin: new origins.S3Origin(imagesBucket),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
  },
  priceClass: cloudfront.PriceClass.PRICE_CLASS_100,
  enableLogging: true,
})
```

## Cognito

### User Pool

```typescript
import * as cognito from 'aws-cdk-lib/aws-cognito'

const userPool = new cognito.UserPool(this, 'UserPool', {
  userPoolName: 'kaze-prod-users',
  selfSignUpEnabled: true,
  signInAliases: { email: true },
  autoVerify: { email: true },
  passwordPolicy: {
    minLength: 8,
    requireLowercase: true,
    requireUppercase: true,
    requireDigits: true,
    requireSymbols: false,
  },
  accountRecovery: cognito.AccountRecovery.EMAIL_ONLY,
})

// Google OAuth
const googleProvider = new cognito.UserPoolIdentityProviderGoogle(
  this,
  'GoogleProvider',
  {
    userPool,
    clientId: googleClientId,
    clientSecret: googleClientSecret,
    scopes: ['email', 'profile'],
    attributeMapping: {
      email: cognito.ProviderAttribute.GOOGLE_EMAIL,
      givenName: cognito.ProviderAttribute.GOOGLE_GIVEN_NAME,
      familyName: cognito.ProviderAttribute.GOOGLE_FAMILY_NAME,
    },
  }
)

// App Client
const userPoolClient = userPool.addClient('WebClient', {
  authFlows: {
    userPassword: true,
    userSrp: true,
  },
  oAuth: {
    flows: {
      authorizationCodeGrant: true,
    },
    scopes: [cognito.OAuthScope.EMAIL, cognito.OAuthScope.OPENID],
    callbackUrls: ['https://kaze.example.com/auth/callback'],
  },
  supportedIdentityProviders: [
    cognito.UserPoolClientIdentityProvider.GOOGLE,
  ],
})
```

## Step Functions

### State Machine

```typescript
import * as sfn from 'aws-cdk-lib/aws-stepfunctions'
import * as tasks from 'aws-cdk-lib/aws-stepfunctions-tasks'

const checkChaptersTask = new tasks.LambdaInvoke(this, 'CheckChapters', {
  lambdaFunction: checkChaptersFunction,
  outputPath: '$.Payload',
})

const sendNotificationsTask = new tasks.LambdaInvoke(this, 'SendNotifications', {
  lambdaFunction: sendNotificationsFunction,
})

const definition = checkChaptersTask.next(sendNotificationsTask)

const stateMachine = new sfn.StateMachine(this, 'ChapterCheckStateMachine', {
  stateMachineName: 'kaze-prod-chapter-check',
  definition,
  timeout: cdk.Duration.minutes(5),
})
```

### EventBridge Schedule

```typescript
import * as events from 'aws-cdk-lib/aws-events'
import * as targets from 'aws-cdk-lib/aws-events-targets'

const rule = new events.Rule(this, 'DailyChapterCheck', {
  schedule: events.Schedule.cron({ hour: '0', minute: '0' }),
})

rule.addTarget(new targets.SfnStateMachine(stateMachine))
```

## SQS

### Queue

```typescript
import * as sqs from 'aws-cdk-lib/aws-sqs'

const scraperQueue = new sqs.Queue(this, 'ScraperQueue', {
  queueName: 'kaze-prod-scraper-queue',
  visibilityTimeout: cdk.Duration.seconds(300),
  retentionPeriod: cdk.Duration.days(14),
  deadLetterQueue: {
    queue: new sqs.Queue(this, 'ScraperDLQ', {
      queueName: 'kaze-prod-scraper-dlq',
    }),
    maxReceiveCount: 3,
  },
})

// Lambda trigger
scraperFunction.addEventSource(
  new lambdaEventSources.SqsEventSource(scraperQueue, {
    batchSize: 10,
  })
)
```

## Secrets Management

### Secrets Manager

```typescript
import * as secretsmanager from 'aws-cdk-lib/aws-secretsmanager'

const apiSecret = new secretsmanager.Secret(this, 'ApiSecret', {
  secretName: 'kaze/api/keys',
  generateSecretString: {
    secretStringTemplate: JSON.stringify({ username: 'admin' }),
    generateStringKey: 'password',
  },
})

// Grant read access
apiSecret.grantRead(lambdaFunction)
```

### Environment Variables

```typescript
// ❌ Bad - hardcoded secret
environment: {
  API_KEY: 'secret-key-123',
}

// ✅ Good - from Secrets Manager
environment: {
  API_SECRET_ARN: apiSecret.secretArn,
}

// In Lambda handler
const secret = await secretsManager.getSecretValue({
  SecretId: process.env.API_SECRET_ARN,
}).promise()
```

## Monitoring & Logging

### CloudWatch Alarms

```typescript
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch'
import * as actions from 'aws-cdk-lib/aws-cloudwatch-actions'

const errorAlarm = new cloudwatch.Alarm(this, 'LambdaErrorAlarm', {
  metric: lambdaFunction.metricErrors(),
  threshold: 10,
  evaluationPeriods: 1,
  alarmDescription: 'Lambda function errors',
})

// SNS notification
const topic = new sns.Topic(this, 'AlarmTopic')
errorAlarm.addAlarmAction(new actions.SnsAction(topic))
```

### Lambda Insights

```typescript
const lambdaFunction = new nodejs.NodejsFunction(this, 'Function', {
  // ...
  insightsVersion: lambda.LambdaInsightsVersion.VERSION_1_0_229_0,
})
```

## Security Best Practices

### IAM Least Privilege

```typescript
// ❌ Bad - too permissive
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['*'],
  resources: ['*'],
}))

// ✅ Good - specific permissions
lambdaFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['s3:GetObject', 's3:PutObject'],
  resources: [`${bucket.bucketArn}/*`],
}))
```

### Encryption

```typescript
// S3 encryption
const bucket = new s3.Bucket(this, 'Bucket', {
  encryption: s3.BucketEncryption.S3_MANAGED,
})

// RDS encryption
const cluster = new rds.DatabaseCluster(this, 'Database', {
  storageEncrypted: true,
})

// DynamoDB encryption
const table = new dynamodb.Table(this, 'Table', {
  encryption: dynamodb.TableEncryption.AWS_MANAGED,
})
```

### VPC Security

```typescript
// Lambda in VPC (only if needed)
const lambdaFunction = new nodejs.NodejsFunction(this, 'Function', {
  vpc,
  vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
  securityGroups: [securityGroup],
})

// Security group rules
securityGroup.addIngressRule(
  ec2.Peer.ipv4('10.0.0.0/16'),
  ec2.Port.tcp(5432),
  'Allow PostgreSQL from VPC'
)
```

## Cost Optimization

### Lambda

- Use ARM64 (Graviton2) for 20% cost savings
- Right-size memory (use Lambda Power Tuning)
- Use provisioned concurrency only when needed

### RDS

- Use Aurora Serverless v2 for variable workloads
- Use T4g instances for dev/staging
- Enable auto-pause for dev databases

### S3

- Use lifecycle policies to transition to cheaper storage
- Enable intelligent tiering
- Delete old versions

### DynamoDB

- Use on-demand billing for unpredictable workloads
- Use provisioned capacity for predictable workloads
- Enable auto-scaling

## Testing Infrastructure

### CDK Assertions

```typescript
import { Template } from 'aws-cdk-lib/assertions'
import { ApiStack } from '../lib/stacks/api-stack'

test('API Stack creates AppSync API', () => {
  const app = new cdk.App()
  const stack = new ApiStack(app, 'TestStack')
  
  const template = Template.fromStack(stack)
  
  template.hasResourceProperties('AWS::AppSync::GraphQLApi', {
    Name: 'kaze-prod-api-graphql',
  })
})
```

## Deployment

### CDK Commands

```bash
# Synthesize CloudFormation template
cdk synth

# Show differences
cdk diff

# Deploy all stacks
cdk deploy --all

# Deploy specific stack
cdk deploy KazeApiStack

# Destroy stack
cdk destroy KazeApiStack
```

### CI/CD Deployment

```yaml
- name: Deploy to AWS
  run: |
    npm ci
    npm run build
    npx cdk deploy --all --require-approval never
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Best Practices Checklist

- [ ] Use CDK for all infrastructure
- [ ] Follow naming conventions
- [ ] Separate stacks by concern
- [ ] Use SSM for cross-stack communication
- [ ] Enable encryption everywhere
- [ ] Follow least privilege for IAM
- [ ] Set appropriate Lambda timeouts and memory
- [ ] Enable CloudWatch alarms
- [ ] Use Secrets Manager for secrets
- [ ] Test infrastructure with CDK assertions
- [ ] Document architecture decisions
- [ ] Monitor costs
- [ ] Enable backups for databases
- [ ] Use VPC only when necessary
- [ ] Enable logging for debugging
