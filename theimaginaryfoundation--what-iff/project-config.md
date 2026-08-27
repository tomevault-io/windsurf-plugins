---
trigger: always_on
description: The Personal Assistant UI is built with Angular and follows modern best practices for scalable frontend applications. It interfaces with the Personal Assistant API to provide a user-friendly interface for content creation and management.
---

# Personal Assistant UI Guidelines

## Project Overview
The Personal Assistant UI is built with Angular and follows modern best practices for scalable frontend applications. It interfaces with the Personal Assistant API to provide a user-friendly interface for content creation and management.

## Project Structure

```
src/
├── app/
│   ├── core/                   # Core modules, services, and components
│   │   ├── components/         # Shared components (navbar, layout)
│   │   ├── guards/             # Route guards
│   │   ├── interceptors/       # HTTP interceptors
│   │   ├── models/             # Data models
│   │   └── services/           # Core services
│   ├── features/               # Feature modules
│   │   ├── auth/               # Authentication feature
│   │   │   └── components/     # Login, register, profile components
│   │   └── dashboard/          # Dashboard feature
│   ├── app.ts                  # Root component
│   ├── app.html                # Root component template
│   ├── app.scss                # Root component styles
│   ├── app.routes.ts           # Application routes
│   └── app.config.ts           # Application configuration
├── assets/                     # Static assets
└── environments/               # Environment configuration
```

## Code Organization Guidelines

### Component Structure
Components must be split into separate files for TypeScript, HTML, and SCSS:

```
component-name/
├── component-name.component.ts
├── component-name.component.html
├── component-name.component.scss
└── component-name.component.spec.ts
```

### Component Definition
All components should be standalone and use the following structure:

```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-example',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.scss']
})
export class ExampleComponent implements OnInit {
  // Component properties
  
  constructor() {}
  
  ngOnInit(): void {
    // Initialization logic
  }
  
  // Component methods
}
```

### File Naming Conventions
- Use kebab-case for file names: `user-profile.component.ts`
- Use camelCase for TypeScript variables, properties, and methods
- Use PascalCase for class names: `UserProfileComponent`
- Use kebab-case for selectors: `<app-user-profile>`

## Styling Guidelines

### SCSS Structure
- Use SCSS for all styling
- Component-specific styles should be in the component's `.scss` file
- Global styles should be in `src/styles.scss`
- Use Tailwind CSS utility classes for common styling needs
- Use the `:host` selector to style the component's host element
- Use BEM methodology for custom CSS classes

### Tailwind CSS
Tailwind CSS is used for utility-first styling. Flowbite components are available for more complex UI elements.

Example:
```html
<div class="flex items-center justify-between p-4 bg-white rounded-md shadow-sm">
  <h2 class="text-lg font-semibold text-gray-800">Title</h2>
  <button class="px-4 py-2 text-white bg-indigo-600 rounded-md hover:bg-indigo-700">
    Action
  </button>
</div>
```

## Service Guidelines

### Service Structure
Services should be organized in the appropriate directory based on their scope:
- Core services in `src/app/core/services/`
- Feature-specific services in `src/app/features/[feature]/services/`

### Service Definition
```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '@environments/environment';

@Injectable({
  providedIn: 'root'
})
export class ExampleService {
  private http = inject(HttpClient);
  private apiUrl = `${environment.apiUrl}/examples`;
  
  getAll(): Observable<Example[]> {
    return this.http.get<Example[]>(this.apiUrl);
  }
  
  getById(id: string): Observable<Example> {
    return this.http.get<Example>(`${this.apiUrl}/${id}`);
  }
  
  create(example: Example): Observable<Example> {
    return this.http.post<Example>(this.apiUrl, example);
  }
  
  update(id: string, example: Example): Observable<Example> {
    return this.http.put<Example>(`${this.apiUrl}/${id}`, example);
  }
  
  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

## Model Guidelines

### Model Definition
Models should be defined as TypeScript interfaces in `src/app/core/models/` or in feature-specific `models` directories.

```typescript
export interface User {
  id: string;
  username: string;
  email: string;
  first_name?: string;
  last_name?: string;
  created_at: string;
  updated_at: string;
}
```

## Routing Guidelines

### Route Definition
Routes should be defined in `app.routes.ts` for main routes and in feature-specific modules for feature routes.

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './core/guards/auth.guard';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'dashboard',
    pathMatch: 'full'
  },
  {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theimaginaryfoundation/what-iff](https://github.com/theimaginaryfoundation/what-iff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
