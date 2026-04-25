---
trigger: always_on
description: Este documento define o padrão completo para criação de histórias técnicas de infraestrutura no repositório.
---


# Regras para Criação de Histórias (Stories)

Este documento define o padrão completo para criação de histórias técnicas de infraestrutura no repositório.

## 📁 Estrutura de Pastas

### Nomenclatura da Pasta da História

**Formato obrigatório:**
```
Storie-XX-Descricao_Breve_Da_Historia
```

**Regras:**
- Sempre começar com `Storie-` seguido do número com 2 dígitos (01, 02, 03...)
- Número deve ser sequencial e único
- Descrição deve ser curta (máximo 5-6 palavras)
- Usar underscore (`_`) para separar palavras na descrição
- Descrição deve ser clara e permitir identificar o conteúdo apenas pelo nome da pasta
- Primeira letra de cada palavra em maiúscula (PascalCase para palavras, snake_case para separação)

**Exemplos corretos:**
- `Storie-01-Backend_Terraform_S3`
- `Storie-05-Modulo_Cluster_EKS`
- `Storie-12-Security_Groups_EKS`
- `Storie-15-Padronizacao_Variaveis_Ambientes`

**Exemplos incorretos:**
- `Storie-1-Backend` (número sem zero à esquerda)
- `Storie-01-backend-terraform` (usar hífen em vez de underscore)
- `Storie-01-BackendTerraformS3` (sem separação, difícil de ler)

### Estrutura Interna da Pasta

```
Storie-XX-Descricao/
├── story.md                    # Arquivo principal da história
└── subtask/                    # Pasta contendo todas as subtasks
    ├── Subtask-01-Nome.md
    ├── Subtask-02-Nome.md
    └── Subtask-03-Nome.md
```

**Regras:**
- Sempre ter um arquivo `story.md` na raiz da pasta
- Sempre ter uma pasta `subtask/` (mesmo que vazia inicialmente)
- Nenhum outro arquivo na raiz da pasta da história

## 📝 Formato do Arquivo story.md

### Estrutura Obrigatória

```markdown
# Storie-XX: Título da História

## Status: Em Andamento

## Descrição
Como [papel], quero [ação], para [benefício].

## Objetivo
[Descrição clara e objetiva do que será entregue]

## Escopo Técnico
- Tecnologias: [Lista de tecnologias]
- Arquivos afetados: [Lista de arquivos/caminhos]
- Recursos AWS: [Lista de recursos AWS que serão criados]

## Subtasks

- [ ] [Subtask 01: Nome da subtask](./subtask/Subtask-01-Nome_da_subtask.md)
- [ ] [Subtask 02: Nome da subtask](./subtask/Subtask-02-Nome_da_subtask.md)
- [x] [Subtask 03: Nome da subtask](./subtask/Subtask-03-Nome_da_subtask.md)

## Critérios de Aceite da História

- [ ] Critério 1 (específico e mensurável)
- [ ] Critério 2 (específico e mensurável)
- [ ] Critério 3 (específico e mensurável)
```

### Regras de Escrita do story.md

1. **Título:**
   - Formato: `# Storie-XX: Título da História`
   - Título deve ser claro e descritivo
   - Máximo 10 palavras

2. **Descrição:**
   - Usar formato "Como... quero... para..."
   - Ser específico sobre o papel (ex: "engenheiro de plataforma")
   - Ação deve ser clara e objetiva
   - Benefício deve ser mensurável ou observável

3. **Objetivo:**
   - Uma única frase ou parágrafo curto
   - Deve explicar O QUE será entregue, não COMO
   - Focar no resultado final

4. **Escopo Técnico:**
   - Listar tecnologias específicas (ex: "Terraform, AWS EKS")
   - Listar arquivos/caminhos que serão criados/modificados
   - Listar recursos AWS que serão provisionados
   - Ser específico, não genérico

5. **Subtasks:**
   - Mínimo 3 subtasks, máximo 8 subtasks
   - Cada subtask deve ser uma ação específica e testável
   - Ordem deve fazer sentido lógico (dependências)
   - Links devem usar caminho relativo: `./subtask/Subtask-XX-Nome.md`
   - Usar formato de checklist: `- [ ]` para marcar subtasks como concluídas
   - Marcar como concluída: `- [x]` quando a subtask estiver finalizada
   - Uma história só é considerada concluída quando todas as subtasks estiverem marcadas como concluídas

6. **Critérios de Aceite:**
   - Mínimo 5 critérios, ideal 7-10
   - Cada critério deve ser:
     - Específico (não genérico)
     - Mensurável (pode ser validado)
     - Testável (pode ser verificado)
   - Usar formato de checklist: `- [ ]`
   - Incluir critérios de:
     - Funcionalidade (o que foi criado)
     - Qualidade (validações, testes)
     - Documentação (se aplicável)

7. **Status da História:**
   - Adicionar campo de status no início do arquivo (após o título)
   - Formato: `## Status: [Em Andamento | Concluída]`
   - Quando todas as subtasks e critérios de aceite estiverem concluídos, marcar como `## Status: Concluída`
   - Marcar subtasks como concluídas usando checkbox: `- [x]` na lista de Subtasks

## 📋 Formato do Arquivo de Subtask

### Nomenclatura da Subtask

**Formato obrigatório:**
```
Subtask-XX-Nome_Da_Subtask.md
```

**Regras:**
- Sempre começar com `Subtask-` seguido do número com 2 dígitos (01, 02, 03...)
- Número deve ser sequencial dentro da história
- Nome deve ser curto e descritivo (máximo 4-5 palavras)
- Usar underscore (`_`) para separar palavras
- Primeira letra de cada palavra em maiúscula

**Exemplos corretos:**
- `Subtask-01-Criar_modulo_Terraform_backend_S3.md`
- `Subtask-02-Configurar_backend_tf_principal.md`
- `Subtask-03-Adicionar_IAM_policies_minimas_backend.md`

### Estrutura Obrigatória da Subtask

```markdown
# Subtask XX: Nome da Subtask

## Status: [Em Andamento | Concluída]

## Descrição
[Uma ou duas frases explicando exatamente o que será feito nesta subtask]

## Passos de implementação
- [ ] [Passo 1 específico e acionável]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diegoknsk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
