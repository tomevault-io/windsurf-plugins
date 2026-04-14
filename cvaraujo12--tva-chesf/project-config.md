---
trigger: always_on
description: "name": "Sistema Integrado de Pesquisa e Produção Acadêmica",
---

# Regras para Busca e Aplicação de Evidências Documentais

## 1. Configuração de Busca Documental
{
    "name": "Sistema Integrado de Pesquisa e Produção Acadêmica",
    "version": "2.0",
    "description": "Sistema integrado para desenvolvimento de pesquisa acadêmica, combinando gestão documental e produção de conteúdo",
    
    "document_search": {
        "priority": "high",
        "search_patterns": [
            "*.md",
            "*.txt",
            "*.pdf"
        ],
        "search_locations": [
            "fontes_pesquisa/",
            "documentos/",
            "referencias/"
        ],
        "metadata_tracking": true
    }
}

## 2. Padrões de Análise Documental
{
    "document_analysis": {
        "required_fields": [
            "codigo_referencia",
            "data_documento",
            "classificacao",
            "status_atual",
            "fonte_original"
        ],
        "cross_reference": true,
        "chronological_order": true,
        "evidence_validation": "strict"
    }
}

## 3. Regras de Citação e Referência
{
    "citation_rules": {
        "format": "chicago",
        "auto_citation": true,
        "cross_linking": true,
        "reference_tracking": {
            "enabled": true,
            "style": "footnote",
            "database": "referencias.db"
        }
    }
}

## 4. Integração de Evidências
{
    "evidence_integration": {
        "validation_levels": [
            "primary_source",
            "secondary_source",
            "cross_reference",
            "contextual"
        ],
        "connection_types": [
            "direct",
            "indirect",
            "contextual",
            "comparative"
        ],
        "priority_ranking": true
    }
}

## 5. Estrutura de Análise
{
    "analysis_structure": {
        "components": [
            "contexto_historico",
            "evidencia_documental",
            "analise_critica",
            "implicacoes",
            "conclusoes"
        ],
        "required_elements": [
            "fonte_primaria",
            "metodologia",
            "argumentacao",
            "conclusao"
        ]
    }
}

## 6. Controle de Qualidade
{
    "quality_control": {
        "verification_steps": [
            "fonte_verificada",
            "contexto_validado",
            "cruzamento_confirmado",
            "analise_revisada"
        ],
        "minimum_sources": 2,
        "cross_validation": true
    }
}

## 7. Automação de Pesquisa
{
    "research_automation": {
        "search_databases": [
            "CIA_FOIA",
            "NARA_Archives",
            "Wilson_Center",
            "Digital_Archives"
        ],
        "update_frequency": "daily",
        "alert_on_new": true
    }
}

## 8. Gestão de Conhecimento
{
    "knowledge_management": {
        "categorization": {
            "enabled": true,
            "taxonomy": [
                "militar",
                "tecnico",
                "diplomatico",
                "estrategico"
            ]
        },
        "relationships": {
            "track": true,
            "visualize": true
        }
    }
}

## 9. Integração com Editores
{
    "editor_integration": {
        "markdown_support": true,
        "citation_tools": true,
        "reference_manager": true,
        "version_control": true
    }
}

## 10. Exportação e Compartilhamento
{
    "export_options": {
        "formats": [
            "markdown",
            "pdf",
            "html",
            "docx"
        ],
        "include_metadata": true,
        "preserve_references": true
    }
}

## 11. Resumo Automático de Fontes
{
    "auto_summary": {
        "enabled": true,
        "output_file": "resumo_geral_pesquisa.md",
        "update_command": "/update-resume",
        "structure": {
            "sections": [
                "fontes_analisadas",
                "fontes_pendentes",
                "conexoes_estabelecidas",
                "proximos_passos",
                "recomendacoes"
            ],
            "per_source": {
                "required_fields": [
                    "status",
                    "relevancia",
                    "descobertas_principais",
                    "pendencias"
                ],
                "metadata": {
                    "track_dates": true,
                    "track_progress": true,
                    "track_connections": true
                }
            }
        },
        "update_triggers": [
            "nova_fonte",
            "nova_analise",
            "nova_conexao",
            "novo_documento"
        ],
        "format": {
            "style": "markdown",
            "include_metadata": true,
            "include_stats": true
        }
    }
}

## 12. Regras de Atualização
{
    "update_rules": {
        "frequency": "on_demand",
        "auto_backup": true,
        "version_control": true,
        "notification": true,
        "summary_sections": {
            "fontes_analisadas": {
                "format": [
                    "nome_arquivo",
                    "tipo_documento",
                    "data_analise",
                    "descobertas_principais"
                ],
                "sort_by": "relevancia"
            },
            "fontes_pendentes": {
                "format": [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cvaraujo12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
