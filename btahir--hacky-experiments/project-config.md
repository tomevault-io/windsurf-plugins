---
trigger: always_on
description: route handler rules
---


# Next.js 15 Route Handler Best Practices

This guide provides a standardized approach for creating efficient, maintainable API routes in Next.js 15.

## Route Handler Template

```typescript
// app/api/[resource]/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod'; // Recommended for validation

// -------------------------------------------
// Route Configuration Options
// -------------------------------------------
export const dynamic = 'auto'; // 'auto' | 'force-dynamic' | 'force-static'
export const revalidate = false; // false | 0 | number (seconds)
export const runtime = 'nodejs'; // 'nodejs' | 'edge'
export const preferredRegion = 'auto'; // 'auto' | 'global' | 'home' | string[]

// -------------------------------------------
// Request Validation
// -------------------------------------------
const createResourceSchema = z.object({
  name: z.string().min(1),
  status: z.enum(['active', 'inactive']),
  // Add other fields as needed
});

type CreateResourceInput = z.infer<typeof createResourceSchema>;

// -------------------------------------------
// Route Handlers
// -------------------------------------------

/**
 * GET: Retrieve resources
 * 
 * @route GET /api/[resource]?query=value
 * @param request - The incoming request object
 * @param params - URL parameters object
 * @returns JSON response with resources or error
 */
export async function GET(
  request: NextRequest,
  { params }: { params?: { [key: string]: string } }
) {
  try {
    // Access query parameters (using URL objects)
    const searchParams = request.nextUrl.searchParams;
    const query = searchParams.get('query');
    const page = parseInt(searchParams.get('page') || '1', 10);
    const limit = parseInt(searchParams.get('limit') || '10', 10);
    
    // Access cookies (if needed)
    const token = request.cookies.get('token')?.value;
    
    // Access headers
    const authorization = request.headers.get('authorization');
    
    // Your data fetching logic here
    // const resources = await fetchResources({ query, page, limit });
    
    // Mock response for template
    const resources = [
      { id: '1', name: 'Example Resource' }
    ];
    
    // Return successful response
    return NextResponse.json(
      { 
        data: resources,
        pagination: { page, limit, total: 100 }
      }, 
      { status: 200 }
    );
  } catch (error) {
    console.error('[API ERROR] GET /api/resource:', error);
    
    // Return appropriate error response
    return NextResponse.json(
      { error: 'Failed to fetch resources' },
      { status: error instanceof z.ZodError ? 400 : 500 }
    );
  }
}

/**
 * POST: Create a new resource
 * 
 * @route POST /api/[resource]
 * @param request - The incoming request with JSON body
 * @returns JSON response with created resource
 */
export async function POST(request: NextRequest) {
  try {
    // Parse and validate request body
    const body = await request.json();
    const validatedData = createResourceSchema.parse(body);
    
    // Your creation logic here
    // const newResource = await createResource(validatedData);
    
    // Mock response for template
    const newResource = { id: 'new-id', ...validatedData };
    
    // Return created response with 201 status
    return NextResponse.json(
      { data: newResource }, 
      { status: 201 }
    );
  } catch (error) {
    console.error('[API ERROR] POST /api/resource:', error);
    
    if (error instanceof z.ZodError) {
      // Handle validation errors
      return NextResponse.json(
        { error: 'Invalid input', details: error.errors },
        { status: 400 }
      );
    }
    
    // Handle other errors
    return NextResponse.json(
      { error: 'Failed to create resource' },
      { status: 500 }
    );
  }
}

/**
 * PUT: Replace a resource completely
 * 
 * @route PUT /api/[resource]/[id]
 * @param request - The incoming request with JSON body
 * @param params - URL parameters with resource ID
 * @returns JSON response with updated resource
 */
export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const { id } = params;
    
    // Parse and validate request body
    const body = await request.json();
    const validatedData = createResourceSchema.parse(body);
    
    // Your update logic here
    // const updatedResource = await replaceResource(id, validatedData);
    
    // Mock response for template
    const updatedResource = { id, ...validatedData };
    
    // Return success response
    return NextResponse.json(
      { data: updatedResource },
      { status: 200 }
    );
  } catch (error) {
    console.error(`[API ERROR] PUT /api/resource/${params?.id}:`, error);
    
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Invalid input', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Failed to update resource' },
      { status: 500 }
    );
  }
}

/**
 * PATCH: Update a resource partially
 * 
 * @route PATCH /api/[resource]/[id]
 * @param request - The incoming request with JSON body
 * @param params - URL parameters with resource ID
 * @returns JSON response with updated resource
 */
export async function PATCH(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btahir/hacky-experiments](https://github.com/btahir/hacky-experiments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
