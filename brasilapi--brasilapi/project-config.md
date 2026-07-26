---
trigger: always_on
description: A BrasilAPI é uma iniciativa **100% gratuita e sem custos** que centraliza APIs públicas brasileiras, oferecendo acesso rápido e moderno a dados como CEP, CNPJ, bancos, feriados, IBGE, entre outros. O projeto é hospedado na Vercel com CDN distribuída globalmente.
---

# Instruções para Desenvolvimento - BrasilAPI

## 🎯 Visão Geral do Projeto

A BrasilAPI é uma iniciativa **100% gratuita e sem custos** que centraliza APIs públicas brasileiras, oferecendo acesso rápido e moderno a dados como CEP, CNPJ, bancos, feriados, IBGE, entre outros. O projeto é hospedado na Vercel com CDN distribuída globalmente.

### Missão Crítica
- **Projeto sem custos**: Todas as soluções devem ser gratuitas e sustentáveis
- **Clientes importantes**: Empresas de grande porte dependem desta API em produção
- **Zero downtime**: Não podemos ter quedas ou indisponibilidades
- **Alta disponibilidade**: Requisito essencial para manter a confiança dos usuários

## ⚠️ Princípios Fundamentais

### 1. Nunca Quebre Contratos de API
- **SEMPRE mantenha compatibilidade retroativa** em endpoints existentes
- Adicione novos campos, mas NUNCA remova ou renomeie campos existentes
- Se precisar mudar a estrutura, crie uma nova versão (v2, v3, etc.)
- Valide com testes E2E antes de fazer merge

### 2. Documentação Sempre Atualizada
- **SEMPRE** atualize a documentação OpenAPI ao modificar endpoints
- Documentação está em `pages/docs/doc/` (formato JSON, OpenAPI 3.0)
- Todo novo endpoint DEVE ter documentação antes do merge
- Mantenha exemplos de requisição/resposta atualizados

### 3. Performance e Custos
- Minimize chamadas a APIs externas (use cache quando possível)
- Evite processamento pesado que aumente custos da Vercel
- Reutilize código e bibliotecas existentes
- Não adicione dependências pesadas desnecessárias

### 4. Qualidade e Confiabilidade
- **SEMPRE** escreva testes E2E para novos endpoints
- Execute `npm test` localmente antes de submeter PR
- Use `npm run fix` para garantir código limpo (ESLint + Prettier)
- Siga os padrões de código existentes no projeto

## 🏗️ Arquitetura do Projeto

### Estrutura de Diretórios
```
/pages/api/        # Endpoints da API (Next.js API Routes)
/services/         # Lógica de negócio e integrações com APIs externas
/tests/            # Testes E2E (Vitest)
/pages/docs/       # Documentação OpenAPI
/components/       # Componentes React para o site
/util/             # Utilitários compartilhados
```

### Padrão de Endpoints
- Endpoints em `/pages/api/[recurso]/[versao]/[...params].js`
- Exemplo: `/pages/api/cep/v1/[cep].js`
- Sempre use versionamento (v1, v2, etc.)
- Use Next.js API Routes com `next-connect` para middleware

### Padrão de Serviços
- Serviços em `/services/[recurso].js`
- Exportam funções assíncronas que fazem integrações
- Tratam erros e retornam dados padronizados
- Não devem ter lógica de HTTP (isso fica nos endpoints)

## 🧪 Testes

### Estrutura de Testes
- Testes E2E usando Vitest em `/tests/`
- Nomeação: `[recurso]-v[versao].test.js`
- Sempre teste CORS (deve retornar `*`)
- Teste casos de sucesso E erro
- Teste validações de entrada

### Comandos
```bash
npm test              # Roda todos os testes
npm run test:watch    # Modo watch
npm run dev           # Servidor local (porta 3000)
```

### Exemplo de Teste E2E
```javascript
describe('/api/recurso/v1 (E2E)', () => {
  test('Verifica CORS', async () => {
    const response = await axios.get(`${global.SERVER_URL}/api/recurso/v1/...`);
    expect(response.headers['access-control-allow-origin']).toBe('*');
  });

  test('Caso de sucesso', async () => {
    const response = await axios.get(`${global.SERVER_URL}/api/recurso/v1/...`);
    expect(response.data).toEqual({
      // estrutura esperada
    });
  });

  test('Caso de erro - recurso não encontrado', async () => {
    try {
      await axios.get(`${global.SERVER_URL}/api/recurso/v1/invalido`);
    } catch (error) {
      expect(error.response.status).toBe(404);
    }
  });
});
```

## 📝 Padrões de Código

### JavaScript
- Use ES6+ (async/await, destructuring, arrow functions)
- Use ESLint + Prettier (configuração já definida)
- Siga o Airbnb Style Guide (base do projeto)
- Evite comentários óbvios, prefira código auto-explicativo

### Commits
- Use Conventional Commits (execute `npm run commit`)
- Formato: `tipo(escopo): mensagem`
- Tipos: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
- Exemplo: `feat(cep): adiciona endpoint v2 com mais detalhes`

### Tratamento de Erros
- Use o schema `ErrorMessage` da documentação OpenAPI
- Retorne status HTTP apropriados (400, 404, 500, 503)
- Mensagens de erro em português, claras e úteis
- Exemplo:
```javascript
return res.status(404).json({
  name: 'NotFoundError',
  message: 'CEP não encontrado',
  type: 'not_found'
});
```

## 🔒 Segurança

- **NUNCA** commite credenciais, tokens ou secrets
- Use variáveis de ambiente para configurações sensíveis
- Valide TODOS os inputs do usuário
- Sanitize dados antes de processar
- Proteja contra injeção e XSS

## 🚀 Workflow de Desenvolvimento

### Criando um Novo Endpoint

1. **Crie o serviço** em `/services/[recurso].js`
   ```javascript
   async function buscarRecurso(id) {
     // lógica de integração
     return dados;
   }
   ```

2. **Crie o endpoint** em `/pages/api/[recurso]/v1/[param].js`
   ```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrasilAPI/BrasilAPI](https://github.com/BrasilAPI/BrasilAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
