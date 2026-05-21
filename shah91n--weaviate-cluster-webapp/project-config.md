---
trigger: always_on
description: | Language | Python 3.10+ |
---

# Weaviate Cluster WebApp — Agent Reference

## Stack & Dependencies

| Item | Detail |
|---|---|
| Language | Python 3.10+ |
| Framework | Streamlit |
| Database | Weaviate |
| Key packages | `streamlit`, `weaviate-client`, `weaviate-client[agents]`, `pandas`, `Pillow`, `requests` |

## Quick Start

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

## Docker

```bash
docker build -t weaviateclusterapp:latest .
docker run -p 8501:8501 --add-host=localhost:host-gateway weaviateclusterapp
```

## Project Structure

```
streamlit_app.py                     App entrypoint — connection UI + cluster dashboard
core/                                Business logic only — no Streamlit imports
  connection/
    weaviate_connection_manager.py   WeaviateConnectionManager singleton + get_weaviate_manager() / get_weaviate_client()
    weaviate_client.py               initialize_weaviate_connection() / disconnect_weaviate()
  cluster/
    cluster_health.py                diagnose_schema(), get_shards_info(), process_shards_data(),
                                     check_shard_consistency(), get_cluster_statistics(),
                                     process_statistics(), get_metadata()
  collection/
    overview.py                      aggregate_collections(), list_collections(), get_schema(),
                                     fetch_collection_config(), process_collection_config()
    create.py                        get_supported_vectorizers(), validate_file_format(),
                                     check_vectorizer_keys(), create_collection(),
                                     batch_upload() [generator], get_collection_info(),
                                     get_collection_objects(), sanitize_keys()
    delete.py                        delete_all_collections(), delete_collections(),
                                     delete_tenants_from_collection()
    update_collection_config.py      get_collection_config(), update_description_and_inverted_index(),
                                     update_multi_tenancy_and_replication(),
                                     update_hnsw_vector_index(), update_pq_quantizer()
  object/
    read.py                          get_tenant_names(), read_objects_batch() [iterator, 1 000 cap]
    update_object.py                 get_object_in_collection(), get_object_in_tenant(),
                                     display_object_as_table(), update_object_properties()
  search/
    hybrid.py                        hybrid_search(), hybrid_search_with_multiple_vectors()
    keyword.py                       keyword_search()
    vector.py                        vector_search(), vector_search_with_multiple_vectors(),
                                     parse_vector_input()
  multitenancy/
    tenantdetails.py                 get_tenant_details(), aggregate_tenant_states()
  rbac/
    read.py                          list_all_users(), list_all_roles(), list_all_permissions(),
                                     list_users_roles_permissions_combined()
  agents/
    query_agent.py                   run_query_agent(), capture_display(), sanitize_display(),
                                     strip_ansi(), extract_known_fields()
  backup/
    list.py                          detect_backup_storage(), get_backup_backend_label(),
                                     list_backups() [top 10 most recent]
pages/                               Streamlit UI — one file per feature, no SDK calls
  cluster/
    cluster_operations_handlers.py  action_nodes_and_shards(), action_aggregate_collections_tenants(),
                                     action_collection_schema(), action_collections_configuration(),
                                     action_statistics(), action_metadata(), action_diagnose()
  utils/
    navigation.py                    navigate() — sidebar nav + logo
    helper.py                        update_side_bar_labels(), clear_session_state()
    page_config.py                   set_custom_page_config()
  agent.py                           QueryAgent natural-language Q&A UI
  backup.py                          Backup list page (auto-detected S3/GCS/Azure backend)
  create.py                          Create collection + batch upload (CSV/JSON)
  delete.py                          Delete collections and/or tenants
  multitenancy.py                    Multi-tenancy browser — config + tenant details
  rbac.py                            RBAC report — users, roles, permissions, combined report
  read.py                            Paginated object browser (1 000 obj cap, 100/page)
  search.py                          Hybrid / keyword / vector search with named-vector support
  update.py                          Update object properties + collection config
assets/                              Static files (weaviate-logo.png)
```

---

## Architecture

### Layers
- **Entrypoint** (`streamlit_app.py`) — Session state init, connection widgets, auto-connect from URL params, cluster dashboard buttons.
- **Core layer** (`core/`) — Pure business logic. **No `st.*` calls ever.** Each module calls `get_weaviate_client()` from the connection manager.
- **Pages layer** (`pages/`) — Streamlit UI only. **No direct Weaviate SDK calls.** Pages call `core/` functions and render results.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shah91n/weaviate-cluster-webapp](https://github.com/Shah91n/weaviate-cluster-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
