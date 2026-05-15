---
trigger: always_on
description: Este documento fornece diretrizes claras para uso do GitHub Copilot em projetos DevOps, SRE e desenvolvimento em Go. O objetivo é garantir que o Copilot gere código, commits e documentações seguindo as melhores práticas, facilitando a manutenção, padronização e automação em ambientes corporativos.
---

# Copilot Instructions

## Objetivo
Este documento fornece diretrizes claras para uso do GitHub Copilot em projetos DevOps, SRE e desenvolvimento em Go. O objetivo é garantir que o Copilot gere código, commits e documentações seguindo as melhores práticas, facilitando a manutenção, padronização e automação em ambientes corporativos.

Essas instruções devem ser aplicadas a projetos que utilizam:
- Go (com foco em CLI e DevOps)
- Shell Script
- Terraform / Terragrunt
- Ansible
- GitHub Actions
- Kubernetes / Helm / Helmfile
- Automação e integração de ferramentas de observabilidade e CI/CD

---

## DIRETRIZ PRINCIPAL
	Evite trabalhar em mais de um arquivo por vez.
	Múltiplas edições simultâneas em um arquivo causarão corrupção.
	Fique conversando e ensinando sobre o que você está fazendo enquanto codifica.

## PROTOCOLO DE ARQUIVO GRANDE E ALTERAÇÃO COMPLEXA

### FASE DE PLANEJAMENTO OBRIGATÓRIA
	Ao trabalhar com arquivos grandes (>300 linhas) ou alterações complexas:
		1. SEMPRE comece criando um plano detalhado ANTES de fazer qualquer edição
            2. Seu plano DEVE incluir:
                   - Todas as funções/seções que precisam de modificação
                   - A ordem em que as alterações devem ser aplicadas
                   - Dependências entre as alterações
                   - Número estimado de edições separadas necessárias

            3. Formate seu plano como:
## PLANO DE EDIÇÃO PROPOSTO
	Trabalhando com: [nome do arquivo]
	Total de edições planejadas: [número]

### FAZENDO EDIÇÕES
	- Concentre-se em uma mudança conceitual por vez
	- Mostre trechos claros de "antes" e "depois" ao propor alterações
	- Inclua explicações concisas sobre o que mudou e porquê
	- Sempre verifique se a edição mantém o estilo de codificação do projeto

### Sequência de edição:
	1. [Primeira alteração específica] - Propósito: [porquê]
	2. [Segunda alteração específica] - Propósito: [porquê]
	3. Você aprova este plano? Vou prosseguir com a Edição [número] após sua confirmação.
	4. ESPERE a confirmação explícita do usuário antes de fazer QUALQUER edição quando o usuário ok editar [número]

### FASE DE EXECUÇÃO
	- Após cada edição individual, indique claramente o progresso:
		"✅ Edição [#] de [total] concluída. Pronto para a próxima edição?"
	- Se você descobrir alterações adicionais necessárias durante a edição:
	- PARE e atualize o plano
	- Obtenha aprovação antes de continuar

### ORIENTAÇÃO DE REFATORAÇÃO
	Ao refatorar arquivos grandes:
	- Divida o trabalho em partes lógicas e funcionalmente independentes
	- Certifique-se de que cada estado intermediário mantenha a funcionalidade
	- Considere a duplicação temporária como uma etapa intermediária válida
	- Sempre indique o padrão de refatoração que está sendo aplicado

### EVITAÇÃO DE LIMITE DE TAXA
	- Para arquivos muito grandes, sugira dividir as alterações em várias sessões
	- Priorize as alterações que são unidades logicamente completas
	- Sempre forneça pontos de parada claros

---

## Diretrizes Gerais para o Copilot
- Gerar código **limpo, legível e comentado** quando necessário.
- Seguir padrões de nomenclatura consistentes.
- Utilizar boas práticas de segurança (evitar credenciais hardcoded, uso de variáveis de ambiente, secrets, etc.).
- Sugerir commits **pequenos e atômicos**.
- Incluir mensagens de commit no formato definido abaixo.
- Gerar testes sempre que possível.
- Usar lint e formatação automáticos.

---

## Formato de Mensagens de Commit
Seguir o padrão **Conventional Commits**:

```
<tipo>(escopo opcional): descrição breve

[corpo opcional explicando mudanças, motivos e contexto]
[rodapé opcional para tickets, breaking changes ou referências]
```

**Tipos comuns:**
- `feat`: nova funcionalidade
- `fix`: correção de bug
- `docs`: alterações na documentação
- `style`: formatação (sem mudança de lógica)
- `refactor`: refatoração de código
- `test`: adição ou ajuste de testes
- `chore`: tarefas de manutenção

**Exemplos:**
```
feat(cli): adicionar suporte a múltiplos ambientes no opsmaster
fix(terraform): corrigir variável não definida em módulo VPC
docs(helmfile): atualizar instruções de deploy para cluster de staging
```

---

## Boas Práticas DevOps
- **Infraestrutura como Código**: todo provisionamento e configuração devem estar versionados no Git.
- **Automação**: preferir pipelines automatizados (CI/CD) para build, teste, deploy e validações.
- **Observabilidade**: configurar métricas, logs e tracing sempre que possível.
- **Segurança**: usar ferramentas como `trivy`, `snyk` ou `kubesec` para escaneamento.
- **Controle de Versionamento**: manter tags e changelogs atualizados.
- **Padronização**: seguir templates e módulos reutilizáveis para Terraform, Ansible e Helm.

---

## Boas Práticas para Go
- Usar `gofmt`, `golangci-lint` e `go vet` para manter o código limpo.
- Estruturar projetos seguindo o padrão:
  ```
  cmd/        # Entrypoints da aplicação
  internal/   # Código interno (não exportado)
  pkg/        # Bibliotecas públicas reutilizáveis
  configs/    # Arquivos de configuração

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [estudosdevops/opsmaster](https://github.com/estudosdevops/opsmaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
