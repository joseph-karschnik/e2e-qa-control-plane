# Adapter Interfaces

The E2E QA Automation Control Plane uses an **adapter pattern** to support different technology stacks. This document outlines the adapter interfaces that need to be implemented for client-specific integrations.

## Architecture Philosophy

**The Platform (Reusable):**
- Automation lifecycle orchestration
- AI-assisted test generation
- Data factory + cleanup
- CI/CD enforcement
- Evidence generation
- Governance hooks

**The Adapters (Client-Specific):**
- CRM Platform adapters
- Integration layer adapters
- Test management adapters
- CI/CD adapters

## Adapter Interfaces

### 1. CRM Platform Adapter

**Interface:** `ICRMAdapter`

```typescript
interface ICRMAdapter {
  authenticate(): Promise<string>;
  createRecord(objectType: string, fields: Record<string, any>): Promise<string>;
  getRecord(objectType: string, recordId: string): Promise<Record<string, any>>;
  updateRecord(objectType: string, recordId: string, fields: Record<string, any>): Promise<void>;
  deleteRecord(objectType: string, recordId: string): Promise<void>;
  queryRecords(soql: string): Promise<Record<string, any>[]>;
}
```

**Supported Platforms:**
- Salesforce (✅ Implemented)
- Guidewire (📋 Adapter needed)
- Duck Creek (📋 Adapter needed)
- Custom CRM (📋 Adapter needed)

### 2. Integration Layer Adapter

**Interface:** `IIntegrationAdapter`

```typescript
interface IIntegrationAdapter {
  authenticate(): Promise<string>;
  validateAPIContract(apiId: string): Promise<APIContract>;
  testMessageFlow(flowName: string, payload: any): Promise<MessageFlowResult>;
  sendMessage(endpoint: string, payload: any): Promise<any>;
}
```

**Supported Platforms:**
- MuleSoft (📋 Adapter needed)
- Custom APIs (📋 Adapter needed)

### 3. Work Management Adapter

**Interface:** `IWorkManagementAdapter`

```typescript
interface IWorkManagementAdapter {
  getWorkItem(workItemId: string): Promise<WorkItem>;
  getParentContext(workItemId: string): Promise<ParentContext>;
  createBug(bugData: BugData): Promise<string>;
  linkTestCases(workItemId: string, testCaseIds: string[]): Promise<void>;
}
```

**Supported Platforms:**
- Jira (✅ Implemented)
- Azure DevOps (📋 Adapter needed)
- Rally (📋 Adapter needed)

### 4. Test Case Management Adapter

**Interface:** `ITestCaseManagementAdapter`

```typescript
interface ITestCaseManagementAdapter {
  createTestCase(testCase: TestCase): Promise<string>;
  updateTestCase(testCaseId: string, testCase: TestCase): Promise<void>;
  uploadTestScript(testCaseId: string, script: string): Promise<void>;
  uploadResult(testCaseId: string, result: TestResult): Promise<void>;
  createTestCycle(cycleName: string): Promise<string>;
}
```

**Supported Platforms:**
- Zephyr Scale (✅ Implemented)
- Xray (📋 Adapter needed)
- qTest (📋 Adapter needed)

### 5. CI/CD Adapter

**Interface:** `ICICDAdapter`

```typescript
interface ICICDAdapter {
  triggerPipeline(branch: string): Promise<string>;
  getPipelineStatus(pipelineId: string): Promise<PipelineStatus>;
  blockDeployment(reason: string): Promise<void>;
  allowDeployment(): Promise<void>;
}
```

**Supported Platforms:**
- GitHub Actions (✅ Implemented)
- Azure Pipelines (📋 Adapter needed)
- GitLab CI (📋 Adapter needed)
- Gearset (✅ Implemented)

### 6. Documentation Adapter

**Interface:** `IDocumentationAdapter`

```typescript
interface IDocumentationAdapter {
  uploadEvidence(testCaseId: string, evidence: Evidence): Promise<string>;
  createPage(title: string, content: string): Promise<string>;
  updatePage(pageId: string, content: string): Promise<void>;
}
```

**Supported Platforms:**
- Confluence (✅ Implemented)
- SharePoint (📋 Adapter needed)
- Custom (📋 Adapter needed)

## Implementation Guide

### Creating a New Adapter

1. **Implement the Interface**
   ```typescript
   export class CustomCRMAdapter implements ICRMAdapter {
     async authenticate(): Promise<string> {
       // Implementation
     }
     // ... other methods
   }
   ```

2. **Register the Adapter**
   ```typescript
   // src/adapters/registry.ts
   adapterRegistry.register('crm', 'custom', CustomCRMAdapter);
   ```

3. **Configure in Environment**
   ```json
   {
     "crm": {
       "adapter": "custom",
       "baseUrl": "...",
       "credentials": "..."
     }
   }
   ```

## Example: Salesforce Adapter

See `src/adapters/crm/SalesforceAdapter.ts` for a complete reference implementation.

## Example: Guidewire Adapter (Stub)

```typescript
export class GuidewireAdapter implements ICRMAdapter {
  private client: GuidewireClient;
  
  async authenticate(): Promise<string> {
    // Guidewire-specific authentication
    const token = await this.client.login();
    return token;
  }
  
  async createRecord(objectType: string, fields: Record<string, any>): Promise<string> {
    // Guidewire-specific record creation
    const record = await this.client.createEntity(objectType, fields);
    return record.id;
  }
  
  // ... implement other methods
}
```

## Benefits of Adapter Pattern

- ✅ **Reusability:** Core platform logic unchanged
- ✅ **Flexibility:** Easy to add new tool integrations
- ✅ **Testability:** Mock adapters for testing
- ✅ **Maintainability:** Clear separation of concerns

