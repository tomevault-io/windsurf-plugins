---
trigger: always_on
description: You are an expert in GraphQL and Simfinity.js framework. Your role is to help generate complete, production-ready GraphQL type definitions that are fully compatible with Simfinity.js patterns.
---

# Simfinity GraphQL Type Generation Rules

You are an expert in GraphQL and Simfinity.js framework. Your role is to help generate complete, production-ready GraphQL type definitions that are fully compatible with Simfinity.js patterns.

## Core Principles

1. **Simfinity Compatibility**: All generated code must follow Simfinity.js patterns exactly as shown in the series-sample project
2. **Type Safety**: Use proper GraphQL types with validation and business rules
3. **Relationship Management**: Handle embedded objects, collections, and relations correctly
4. **Validation**: Include comprehensive field and type-level validations
5. **Controllers**: Implement proper lifecycle hooks and business logic
6. **State Machines**: Support state management when needed

## Project Structure

When generating a new Simfinity project, follow this exact structure:

```
project-name/
├── types/
│   ├── index.js                    # Central type loading with dependency order
│   ├── scalars.js                  # Custom validated scalars
│   ├── validators/
│   │   ├── customErrors.js         # Custom error classes
│   │   ├── fieldValidators.js      # Field-level validations
│   │   └── typeValidators.js       # Type-level business rules
│   ├── controllers/
│   │   └── [typeName]Controller.js # Type-specific controllers
│   └── [typeName].js               # Individual type definitions
├── package.json                    # Dependencies and scripts with server options
├── index.js                        # Main application entry (Simfinity server)
├── index.apollo.js                 # Apollo Server implementation
├── index.yoga.js                   # GraphQL Yoga implementation
└── .cursorrules                    # This file
```

## Type Definition Patterns

### Basic Type Structure

```javascript
import * as graphql from 'graphql';
import * as simfinity from '@simtlix/simfinity-js';
import { validateFieldName } from './validators/fieldValidators.js';
import { validateTypeBusinessRules } from './validators/typeValidators.js';
import { CustomScalar } from './scalars.js';

const { GraphQLObjectType, GraphQLString, GraphQLID, GraphQLNonNull, GraphQLList } = graphql;

const typeName = new GraphQLObjectType({
  name: 'typeName',
  extensions: {
    validations: {
      create: [validateTypeBusinessRules],
      update: [validateTypeBusinessRules]
    }
  },
  fields: () => ({
    id: { type: GraphQLID },
    // Field definitions with validations
  })
});

export default typeName;
simfinity.connect(null, typeName, 'typeName', 'collectionName', controller, null, stateMachine);
```

### Field Types and Extensions

#### Basic Fields
```javascript
// Required string field
name: { 
  type: new GraphQLNonNull(GraphQLString),
  extensions: {
    validations: {
      save: [validateName],
      update: [validateName]
    }
  }
}

// Optional string field
description: { type: GraphQLString }

// List field
categories: { 
  type: new GraphQLList(GraphQLString),
  extensions: {
    validations: {
      save: [validateCategories],
      update: [validateCategories]
    }
  }
}
```

#### Custom Scalars
```javascript
// Use custom validated scalars
episodeNumber: { 
  type: EpisodeNumberScalar,
  extensions: {
    validations: {
      save: [validateEpisodeNumber],
      update: [validateEpisodeNumber]
    }
  }
}
```

#### Relations

**Embedded Objects (addNoEndpointType)**
```javascript
director: {
  type: new GraphQLNonNull(simfinity.getType('director')),
  extensions: {
    relation: {
      embedded: true,
      displayField: 'name'
    }
  }
}
```

**Collection Relations**
```javascript
// One-to-many relation
seasons: {
  type: new GraphQLList(simfinity.getType('season')),
  extensions: {
    relation: { 
      connectionField: 'serie' 
    }
  }
}

// Many-to-many through junction table
stars: {
  type: new GraphQLList(simfinity.getType('assignedStarAndSerie')),
  extensions: {
    relation: {
      connectionField: 'serie'
    }
  }
}
```

**Foreign Key Relations**
```javascript
serie: {
  type: simfinity.getType('serie'),
  extensions: {
    relation: {
      connectionField: 'serie',
      displayField: 'name'
    }
  }
}
```

### Enums

```javascript
const stateEnum = new GraphQLEnumType({
  name: 'stateEnum',
  values: {
    SCHEDULED: { value: 'SCHEDULED' },
    ACTIVE: { value: 'ACTIVE' },
    FINISHED: { value: 'FINISHED' }
  }
});

// Usage in type
state: { type: stateEnum }
```

### State Machines

```javascript
const stateMachine = {
  initialState: stateEnum.getValue('SCHEDULED'),
  actions: {
    activate: {
      from: stateEnum.getValue('SCHEDULED'),
      to: stateEnum.getValue('ACTIVE'),
      action: async (params) => {
        console.log('Activating:', JSON.stringify(params));
      }
    },
    finalize: {
      from: stateEnum.getValue('ACTIVE'),
      to: stateEnum.getValue('FINISHED'),
      action: async (params) => {
        console.log('Finalizing:', JSON.stringify(params));
      }
    }
  }
};
```

### Controllers

```javascript
const typeController = {
  onSaving: async (doc, args, session) => {
    console.log(`Creating ${doc.name}`);
    // Pre-save logic
  },


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simtlix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
