---
trigger: always_on
description: handleServerAction(updateCandidateInfo(data)),
---

# Server Action Pattern & Cache Key Usage

## Rule: Always Use ServerActionReturnType, handleServerAction, and Centralized Cache Keys

When working with server actions and queries, **ALWAYS** follow this consistent pattern for error handling, type safety, and cache management.

### ✅ Server Action Creation Pattern:

```typescript
"use server"

import { ServerActionReturnType } from "@/utils/types"
import { z } from "zod"

// Define your schema
const myServerActionSchema = z.object({
  // your schema here
})

export async function myServerAction(
  data: z.infer<typeof myServerActionSchema>
): Promise<ServerActionReturnType<YourReturnType>> {
  try {
    // Validate input data
    const validatedData = myServerActionSchema.parse(data)
    
    // Your logic here
    const result = await someOperation(validatedData)

    return {
      data: result,
      error: null,
      success: true,
    }
  } catch (error) {
    return {
      data: null,
      error: error instanceof Error ? error.message : "Operation failed",
      success: false,
    }
  }
}
```

### ✅ Client-Side Usage Pattern with Cache Keys:

```typescript
import { handleServerAction } from "@/utils"
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query"
import queryCacheKeys from "@/cache/query-cache-keys"

// For queries - ALWAYS use cache keys from queryCacheKeys
const { data, isLoading } = useQuery({
  queryKey: queryCacheKeys.candidateSheet.getGeneralInfo(resumeId),
  queryFn: () => handleServerAction(getCandidateGeneralInfo(resumeId)),
})

// Multiple parameter queries
const { data: jobMatches } = useQuery({
  queryKey: queryCacheKeys.jobMatches.getJobMatchResumes(
    jobMatchId, 
    forScoring, 
    status, 
    searchTerm, 
    page, 
    itemsPerPage
  ),
  queryFn: () => handleServerAction(getJobMatchResumes({
    jobMatchId,
    forScoring,
    status,
    searchTerm,
    page,
    itemsPerPage
  })),
})

// For mutations with proper cache invalidation
const queryClient = useQueryClient()

const { mutateAsync: updateCandidate } = useMutation({
  mutationFn: (data: UpdateCandidateData) => 
    handleServerAction(updateCandidateInfo(data)),
  onSuccess: (result, variables) => {
    // Invalidate related cache keys
    queryClient.invalidateQueries({
      queryKey: queryCacheKeys.candidateSheet.getGeneralInfo(variables.resumeId),
    })
    queryClient.invalidateQueries({
      queryKey: queryCacheKeys.candidateSheet.getMissingInfo(variables.resumeId),
    })
  },
})

// Usage example
const handleSave = async () => {
  try {
    await updateCandidate(formData)
    // Handle success
  } catch (error) {
    // Handle error
    console.error("Failed to update candidate:", error)
  }
}
```

### 🗂️ Cache Key Structure Examples:

```typescript
// Available cache key categories and their usage:

// User data
queryCacheKeys.user.userInfo()

// Candidate sheet operations
queryCacheKeys.candidateSheet.getGeneralInfo(resumeId)
queryCacheKeys.candidateSheet.getMissingInfo(resumeId)
queryCacheKeys.candidateSheet.getNotes(resumeId, jobMatchId, meMode)
queryCacheKeys.candidateSheet.getCandidateStatus(resumeId, jobMatchId)

// Job matches
queryCacheKeys.jobMatches.getJobMatchList(searchTerm, slice)
queryCacheKeys.jobMatches.getJobMatchResumes(jobMatchId, forScoring, status, searchTerm, page, itemsPerPage)
queryCacheKeys.jobMatches.getScoredResumes(jobMatchId)

// Import methods
queryCacheKeys.importMethods.getImportMethods(searchTerm)
queryCacheKeys.importMethods.getCandidates(importMethodId, searchTerm)
queryCacheKeys.importMethods.getImportMethodData(importMethodId)

// Favorites
queryCacheKeys.favorites.getResumeFavState(resumeId, jobMatchId)
queryCacheKeys.favorites.getFavoritesCount(jobMatchId)

// Settings
queryCacheKeys.settings.getNotificationSettings()
queryCacheKeys.settings.getGDPRState()
queryCacheKeys.settings.getDeletedItems(category, searchTerm)

// Credits and plans
queryCacheKeys.credits.getCredits()
queryCacheKeys.credits.getPlan()

// Candidate database
queryCacheKeys.candidateDatabase.getCandidates(itemsPerPage, candidateId, searchTerm, importMethods)
queryCacheKeys.candidateDatabase.getCounts()

// Resume operations
queryCacheKeys.resume.getResumePresignedUrl(resumeId)

// Form data
queryCacheKeys.form.getFormAnswer(resumeId)

// Requirements
queryCacheKeys.requirements.getCachedFields(jobMatchId)
queryCacheKeys.requirements.getLocations(city)
```

### 🔄 Cache Invalidation Patterns:

```typescript
// Single cache invalidation
queryClient.invalidateQueries({
  queryKey: queryCacheKeys.candidateSheet.getGeneralInfo(resumeId),
})

// Multiple related cache invalidations
queryClient.invalidateQueries({
  queryKey: queryCacheKeys.jobMatches.getJobMatchResumes(jobMatchId, forScoring),
})
queryClient.invalidateQueries({
  queryKey: queryCacheKeys.jobMatches.getJobMatchResumesCount(jobMatchId, forScoring),
})

// Invalidate all caches for a specific candidate
queryClient.invalidateQueries({
  queryKey: ["candidate", resumeId], // This matches candidateSheet.getGeneralInfo
})

// Invalidate by prefix (for broader cache clearing)
queryClient.invalidateQueries({
  predicate: (query) => 
    query.queryKey[0] === "job-match-resumes" && 
    query.queryKey[1] === jobMatchId,
})
```



### 🎯 Key Benefits:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayoubben18) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
