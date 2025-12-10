# API Response Guidelines

## Collection Responses

When returning collections of objects, **NEVER** return a raw array. Always wrap the collection in an object with a `data` property.

This pattern enables future extensibility (pagination, metadata, filters info, etc.) without breaking changes.

### Bad Example

```typescript
// ❌ BAD - Returns raw array
@Get()
async list(): Promise<CompanyListItem[]> {
  return this.service.execute();
}

// Response: [ { id: "1", name: "A" }, { id: "2", name: "B" } ]
```

### Good Example

```typescript
// ✅ GOOD - Returns object with data property
@Get()
async list(): Promise<ListCompaniesResult> {
  return this.service.execute();
}

// Response: { data: [ { id: "1", name: "A" }, { id: "2", name: "B" } ] }
```

### Response DTO Pattern

```typescript
// For list endpoints, always define a response DTO like this:

export class CompanyListItem {
  id: string;
  name: string;
}

export interface ListCompaniesResult {
  data: CompanyListItem[];
  // Future: pagination, total count, etc.
}
```

### Future Extensibility

This pattern allows adding pagination without breaking changes:

```typescript
export interface ListCompaniesResult {
  data: CompanyListItem[];
  pagination?: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

## Single Resource Responses

Single resource endpoints (GET by ID, POST create, PUT update) return the resource object directly.

```typescript
// ✅ Single resource - object is OK
@Get(':id')
async get(@Param('id') id: string): Promise<CompanyResponseDto> {
  return this.getService.execute({ id });
}

// Response: { id: "1", name: "Company A" }
```

