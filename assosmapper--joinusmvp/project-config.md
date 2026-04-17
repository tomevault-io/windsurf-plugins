---
trigger: always_on
description: Tu es un développeur senior en VueJS et NestJS travaillant sur une application moderne. Voici le guide complet pour développer des fonctionnalités.
---

# Guide de Développement VueJS & NestJS

Tu es un développeur senior en VueJS et NestJS travaillant sur une application moderne. Voici le guide complet pour développer des fonctionnalités.

Tu dois utiliser les composants de Primevue, les icones primeicon et tailwind (j'utilise primevue tailwind).

## Structure du Projet

```
project/
├── app/                      # Application Frontend Vue
│   ├── src/
│   │   ├── components/      # Composants réutilisables
│   │   ├── views/          # Pages de l'application
│   │   ├── services/       # Services API
│   │   ├── store/          # Stores Pinia
│   │   ├── router/         # Configuration des routes
│   │   └── types/          # Types TypeScript
│   │
├── back-core/               # Backend NestJS
│   ├── src/
│   │   ├── modules/        # Modules NestJS
│   │   ├── utils/          # Utilitaires partagés
│   │   └── config/         # Configuration
│   │
└── shared/                  # Code partagé
    ├── dto/                # Data Transfer Objects
    ├── types/              # Interfaces partagées
    └── validations/        # Schémas de validation
```

## Conventions de Nommage

### Composants Vue

- PascalCase pour les noms de fichiers et composants
- Suffixe descriptif du type de composant (Modal, Form, List)

Exemple:

```1:12:app/src/components/AssociationApplication/AssociationApplicationFormModal.vue
<script setup lang="ts">
import { ApplicationStatus, AssociationApplication } from '@shared/types/association-applications';
import FloatLabel from 'primevue/floatlabel';
import { computed, onMounted, ref } from 'vue';
import associationApplicationService from '@/services/associationApplicationService';
import { useForm } from 'vee-validate';
import { joinAssociationSchema } from '@shared/validations/association-applications.validation';
import { JoinAssociationDto } from '@shared/dto/association-applications.dto';
import JnsField from '@/components/ui/JnsField.vue';
import { useNotificationStore } from '@/store/notificationStore';
import { cp } from 'fs';

```

### Services et Controllers

- Suffixe `Service` ou `Controller`
- camelCase pour les méthodes
- Noms explicites des actions (create, update, delete)

## Création d'une Nouvelle Fonctionnalité

### 1. Types Partagés (shared/)

```typescript
// shared/types/my-feature.ts
export interface MyFeature {
  id: string;
  name: string;
  description: string;
}
```

### 2. DTOs et Validations

```typescript
// shared/dto/my-feature.dto.ts
export class CreateMyFeatureDto {
  name: string;
  description: string;
}

// shared/validations/my-feature.validation.ts
export const myFeatureSchema = yup.object().shape({
  name: yup.string().required(),
  description: yup.string().max(255),
});
```

### 3. Backend (back-core/)

#### Entity

```typescript
// back-core/src/my-feature/entities/my-feature.entity.ts
@Entity()
export class MyFeature extends EntityStructure {
  @Column()
  name: string;

  @Column()
  description: string;
}
```

#### Service

```typescript
// back-core/src/my-feature/my-feature.service.ts
@Injectable()
export class MyFeatureService {
  constructor(
    @Inject("MY_FEATURE_REPOSITORY")
    private repository: Repository<MyFeature>
  ) {}

  async create(dto: CreateMyFeatureDto) {
    return this.repository.save(dto);
  }
}
```

#### Controller

```typescript
// back-core/src/my-feature/my-feature.controller.ts
@Controller("my-feature")
@BearAuthToken()
export class MyFeatureController {
  constructor(private service: MyFeatureService) {}

  @Post()
  create(
    @Body(new YupValidationPipe(myFeatureSchema)) dto: CreateMyFeatureDto
  ) {
    return this.service.create(dto);
  }
}
```

### 4. Frontend (app/)

#### Service API

```typescript
// app/src/services/myFeatureService.ts
const myFeatureService = {
  create: async (data: CreateMyFeatureDto) => {
    const apiStore = useApiStore();
    const { data: response, error } = await useApi(apiStore.myFeature.create)
      .post(data)
      .json();

    if (error.value) throw error.value;
    return response.value;
  },
};
```

#### Composant Vue

```vue
<!-- app/src/components/MyFeature/MyFeatureForm.vue -->
<script setup lang="ts">
import { useForm } from "vee-validate";
import { myFeatureSchema } from "@shared/validations/my-feature.validation";

const { handleSubmit, errors, defineField } = useForm({
  validationSchema: myFeatureSchema,
});

const [name, nameAttrs] = defineField("name");
</script>

<template>
  <form @submit="onSubmit">
    <JnsField :errorMessage="errors.name">
      <FloatLabel>
        <InputText v-model="name" v-bind="nameAttrs" />
      </FloatLabel>
    </JnsField>
  </form>
</template>
```

## Gestion des Erreurs

### Backend

Voir l'exemple dans:

```26:66:back-core/src/association-applications/association-applications.service.ts
  async joinAssociation(
    userId: string,
    joinAssociationDto: JoinAssociationDto,
  ) {
    const { associationId, applicationAnswer } = joinAssociationDto;
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['associations'],
    });
    const association = await this.associationRepository.findOne({
      where: { id: associationId },
    });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AssosMapper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
