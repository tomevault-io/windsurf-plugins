---
trigger: always_on
description: Copy-paste UI patterns for common pages: entity search, data table, form, chart, master-detail. Read when building new pages or features.
---


# UI Pattern Cookbook

Copy-paste patterns using the project's actual composables and Vuetify components. Adapt to your needs.

## 1. Entity Search Page

Search for entities by name and display results.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Entity Search</h1>
        <v-text-field
            v-model="query"
            label="Search entities"
            prepend-inner-icon="mdi-magnify"
            variant="outlined"
            @keyup.enter="search"
            :loading="loading"
        />
        <v-alert v-if="error" type="error" variant="tonal" class="mt-2" closable>
            {{ error }}
        </v-alert>
        <v-list v-if="results.length" class="mt-4">
            <v-list-item
                v-for="(neid, i) in results"
                :key="neid"
                :title="names[i] || neid"
                :subtitle="neid"
            />
        </v-list>
        <v-empty-state
            v-else-if="searched && !loading"
            headline="No results"
            icon="mdi-magnify-remove-outline"
        />
    </div>
</template>

<script setup lang="ts">
    import { useElementalClient } from '@yottagraph-app/elemental-api/client';

    const client = useElementalClient();
    const query = ref('');
    const results = ref<string[]>([]);
    const names = ref<string[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const searched = ref(false);

    async function search() {
        if (!query.value.trim()) return;
        loading.value = true;
        error.value = null;
        searched.value = true;
        try {
            const res = await client.getNEID({
                entityName: query.value.trim(),
                maxResults: 10,
                includeNames: true,
            });
            results.value = res.neids || [];
            names.value = res.names || [];
        } catch (e: any) {
            error.value = e.message || 'Search failed';
            results.value = [];
        } finally {
            loading.value = false;
        }
    }
</script>
```

## 2. Data Table Page

Sortable table with loading, empty, and error states.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Data Table</h1>
        <v-alert v-if="error" type="error" variant="tonal" class="mb-4" closable>
            {{ error }}
        </v-alert>
        <v-data-table
            :headers="headers"
            :items="items"
            :loading="loading"
            density="comfortable"
            hover
        >
            <template v-slot:item.actions="{ item }">
                <v-btn icon size="small" variant="text" @click="onView(item)">
                    <v-icon>mdi-eye</v-icon>
                </v-btn>
            </template>
            <template v-slot:no-data>
                <v-empty-state headline="No data" icon="mdi-database-off" />
            </template>
        </v-data-table>
    </div>
</template>

<script setup lang="ts">
    const headers = [
        { title: 'Name', key: 'name', sortable: true },
        { title: 'Type', key: 'type', sortable: true },
        { title: 'Updated', key: 'updatedAt', sortable: true },
        { title: '', key: 'actions', sortable: false, width: 60 },
    ];

    const items = ref<any[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);

    function onView(item: any) {
        // Handle row action
    }

    onMounted(async () => {
        loading.value = true;
        try {
            // items.value = await fetchData();
        } catch (e: any) {
            error.value = e.message || 'Failed to load';
        } finally {
            loading.value = false;
        }
    });
</script>
```

## 3. Form with Validation

Form with field validation, submit handler, and user feedback.

```vue
<template>
    <v-container class="py-6" style="max-width: 600px">
        <h1 class="text-h5 mb-4">Settings</h1>
        <v-form ref="formRef" v-model="valid" @submit.prevent="submit">
            <v-text-field
                v-model="form.name"
                label="Name"
                :rules="[rules.required]"
                variant="outlined"
                class="mb-3"
            />
            <v-text-field
                v-model="form.email"
                label="Email"
                :rules="[rules.required, rules.email]"
                variant="outlined"
                class="mb-3"
            />
            <v-select
                v-model="form.role"
                :items="['Admin', 'Editor', 'Viewer']"
                label="Role"
                variant="outlined"
                class="mb-3"
            />
            <v-btn type="submit" color="primary" :loading="saving" :disabled="!valid">
                Save
            </v-btn>
        </v-form>
    </v-container>
</template>

<script setup lang="ts">
    import { useNotification } from '~/composables/useNotification';

    const { showSuccess, showError } = useNotification();
    const formRef = ref();
    const valid = ref(false);
    const saving = ref(false);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
