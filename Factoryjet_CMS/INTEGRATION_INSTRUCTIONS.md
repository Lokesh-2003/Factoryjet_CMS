# Phase 1 Integration Instructions

## Step 1: Update API Routes

Add to `apps/api/src/index.ts`:
```typescript
import branches from './routes/branches';
import pullRequests from './routes/pull-requests';

// Add routes
app.route('/branches', branches);
app.route('/pull-requests', pullRequests);
```

## Step 2: Run Database Migration
```bash
cd apps/api

# For local development
wrangler d1 execute factoryjet-db \
  --file=../../packages/db/migrations/0001_pr_workflow.sql \
  --local

# For production
wrangler d1 execute factoryjet-db \
  --file=../../packages/db/migrations/0001_pr_workflow.sql
```

## Step 3: Test API Endpoints
```bash
# Create branch
curl -X POST http://localhost:8787/branches/create \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "projectId": "PROJECT_ID",
    "description": "add new feature"
  }'

# List branches
curl http://localhost:8787/branches/PROJECT_ID \
  -H "Authorization: Bearer YOUR_TOKEN"

# Create PR
curl -X POST http://localhost:8787/pull-requests/create \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "projectId": "PROJECT_ID",
    "branchId": "BRANCH_ID",
    "title": "Add new feature",
    "changes": [
      {
        "path": "src/app/page.tsx",
        "type": "modified",
        "additions": 10,
        "deletions": 2
      }
    ]
  }'
```

## Step 4: Add Frontend Route

Create `apps/web/src/app/projects/[id]/pr/[prId]/page.tsx`:
```typescript
import { PRDetails } from '@/components/pr/PRDetails';

export default function PRPage({ params }: { params: { id: string; prId: string } }) {
  return (
    <div className="min-h-screen bg-zinc-950">
      <PRDetails prId={params.prId} projectId={params.id} />
    </div>
  );
}
```

## Step 5: Test Frontend

1. Start dev server: `pnpm dev`
2. Navigate to `/projects/PROJECT_ID/pr/PR_ID`
3. Verify all features work:
   - PR details display
   - Status checks show
   - Risk flags visible
   - Files list correctly
   - Diffs expand/collapse
   - Merge button validates
   