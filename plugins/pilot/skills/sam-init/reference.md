# SAM Init Reference Templates

Use these templates when scaffolding a new SAM project. Replace `{{PROJECT_NAME}}` and `{{FUNCTION_NAME}}` with actual values.

---

## template.yaml

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: >
  {{PROJECT_NAME}} - Serverless backend API

Parameters:
  FrontendUrl:
    Type: String
    Default: "https://your-frontend.com"
    Description: Frontend URL for redirects/CORS. Set via .env and deploy.sh so it is not committed.

Globals:
  Function:
    Timeout: 30
  Api:
    Cors:
      AllowMethods: "'*'"
      AllowHeaders: "'*'"
      AllowOrigin: "'*'"

Resources:
  AppApi:
    Type: AWS::Serverless::Api
    Properties:
      Name: {{PROJECT_NAME}}-api
      StageName: Prod
      Cors:
        AllowMethods: "'*'"
        AllowHeaders: "'*'"
        AllowOrigin: "'*'"

  AppTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: {{PROJECT_NAME}}
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: pk
          AttributeType: S
        - AttributeName: sk
          AttributeType: S
      KeySchema:
        - AttributeName: pk
          KeyType: HASH
        - AttributeName: sk
          KeyType: RANGE
      TimeToLiveSpecification:
        AttributeName: ttl
        Enabled: true

  AppFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: {{PROJECT_NAME}}-{{FUNCTION_NAME}}
      CodeUri: src/lambda-functions/{{FUNCTION_NAME}}
      Handler: index.handler
      Runtime: nodejs20.x
      Environment:
        Variables:
          TABLE_NAME: !Ref AppTable
          FRONTEND_URL: !Ref FrontendUrl
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref AppTable
      Events:
        Health:
          Type: Api
          Properties:
            RestApiId: !Ref AppApi
            Path: /health
            Method: get
        GetItems:
          Type: Api
          Properties:
            RestApiId: !Ref AppApi
            Path: /items
            Method: get
        CreateItem:
          Type: Api
          Properties:
            RestApiId: !Ref AppApi
            Path: /items
            Method: post

Outputs:
  ApiUrl:
    Description: Base URL of the API
    Value: !Sub "https://${AppApi}.execute-api.${AWS::Region}.amazonaws.com/Prod"
    Export:
      Name: !Sub "${AWS::StackName}-ApiUrl"
```

---

## samconfig.toml

```toml
version = 0.1

[default.global.parameters]
stack_name = "{{PROJECT_NAME}}"

[default.build.parameters]
cached = true
parallel = true

[default.validate.parameters]
lint = true

[default.deploy.parameters]
capabilities = "CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND"
confirm_changeset = true
resolve_s3 = true

[default.package.parameters]
resolve_s3 = true

[default.sync.parameters]
watch = true

[default.local_start_api.parameters]
warm_containers = "EAGER"

[default.local_start_lambda.parameters]
warm_containers = "EAGER"
```

---

## deploy.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

# ── Load environment variables ───────────────────────────────────────────────
if [ -f .env ]; then
  set -a
  source .env
  set +a
fi

# ── Build parameter overrides from .env ──────────────────────────────────────
PARAMS=""
if [ -n "${FrontendUrl:-}" ]; then
  PARAMS="--parameter-overrides FrontendUrl=${FrontendUrl}"
fi

# ── Build & Deploy ───────────────────────────────────────────────────────────
sam build && sam deploy $PARAMS
```

---

## Lambda index.js (handler skeleton)

```javascript
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const {
  DynamoDBDocumentClient,
  GetCommand,
  PutCommand,
  QueryCommand,
} = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);
const TABLE_NAME = process.env.TABLE_NAME;

// ─── Response Helpers ───────────────────────────────────────────────────────

function jsonResponse(body, statusCode = 200) {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': '*',
    },
    body: JSON.stringify(body),
  };
}

// ─── Request Helpers ────────────────────────────────────────────────────────

function parseBody(event) {
  if (!event.body) return {};
  try {
    return typeof event.body === 'string' ? JSON.parse(event.body) : event.body;
  } catch {
    return {};
  }
}

function getQueryParams(event) {
  const q = event.queryStringParameters || {};
  return (key) => q[key] || null;
}

// ─── Route Handlers ─────────────────────────────────────────────────────────

async function handleHealth() {
  return jsonResponse({ status: 'ok', timestamp: new Date().toISOString() });
}

async function handleGetItems(event) {
  // TODO: implement
  return jsonResponse({ items: [] });
}

async function handleCreateItem(event) {
  const body = parseBody(event);
  // TODO: implement
  return jsonResponse({ message: 'Created' }, 201);
}

// ─── Main Handler ───────────────────────────────────────────────────────────

exports.handler = async (event) => {
  const path = event.requestContext?.http?.path ?? event.path ?? '';
  const method = (event.requestContext?.http?.method ?? event.httpMethod ?? '').toUpperCase();

  try {
    if (method === 'GET' && path.includes('/health')) {
      return await handleHealth();
    }
    if (method === 'GET' && path.includes('/items')) {
      return await handleGetItems(event);
    }
    if (method === 'POST' && path.includes('/items')) {
      return await handleCreateItem(event);
    }

    return jsonResponse({ error: 'Not found' }, 404);
  } catch (err) {
    console.error('Handler error:', err);
    return jsonResponse({ error: 'Internal server error' }, 500);
  }
};
```

---

## Lambda package.json

```json
{
  "name": "{{FUNCTION_NAME}}",
  "version": "1.0.0",
  "description": "{{PROJECT_NAME}} Lambda function",
  "main": "index.js",
  "dependencies": {
    "@aws-sdk/client-dynamodb": "^3.700.0",
    "@aws-sdk/lib-dynamodb": "^3.700.0"
  }
}
```

---

## .gitignore

```gitignore
# Created by https://www.toptal.com/developers/gitignore/api/osx,node,linux,windows,sam

### Linux ###
*~
.fuse_hidden*
.directory
.Trash-*
.nfs*

### Node ###
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
report.[0-9]*.[0-9]*.[0-9]*.[0-9]*.json
pids
*.pid
*.seed
*.pid.lock
lib-cov
coverage
*.lcov
.nyc_output
.grunt
bower_components
.lock-wscript
build/Release
node_modules/
jspm_packages/
typings/
*.tsbuildinfo
.npm
.eslintcache
.stylelintcache
.rpt2_cache/
.rts2_cache_cjs/
.rts2_cache_es/
.rts2_cache_umd/
.node_repl_history
*.tgz
.yarn-integrity
.env
.env.test
.env*.local
deploy.sh
.cache
.parcel-cache
.next
.nuxt
dist
dist/
.out
.storybook-out
storybook-static
.cache/
.vuepress/dist
.serverless/
.fusebox/
.dynamodb/
.tern-port
.vscode-test
tmp/
temp/

### OSX ###
.DS_Store
.AppleDouble
.LSOverride
Icon
._*
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

### SAM ###
**/.aws-sam

### Windows ###
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db
*.stackdump
[Dd]esktop.ini
$RECYCLE.BIN/
*.cab
*.msi
*.msix
*.msm
*.msp
*.lnk
```

---

## events/event.json

```json
{
  "httpMethod": "GET",
  "path": "/health",
  "queryStringParameters": null,
  "headers": {
    "Content-Type": "application/json"
  },
  "body": null,
  "isBase64Encoded": false
}
```

---

## .env.example

```env
# Copy this to .env and fill in real values
# .env is gitignored — never commit it

FrontendUrl=https://your-frontend.com
```

---

## CLAUDE.md skeleton

The generated CLAUDE.md should follow the structure of the existing project's CLAUDE.md but adapted for the new project:

1. **Project Overview** - Name, purpose (placeholder), tech stack
2. **Architecture** - Components (API Gateway, Lambda, DynamoDB), API routes, DynamoDB schema
3. **Project Structure** - Directory tree
4. **Code Patterns** - Handler structure, response helpers, DynamoDB patterns
5. **Development Workflow** - Setup, local dev, deployment commands
6. **Important Considerations** - Security, CORS, single-table design
7. **AI Assistant Guidelines** - Rules for AI assistants working on the project
