---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important constraints

### Não comentar o óbvio

**Não** escrever um comentário em cima de cada mudança explicando o que ela faz ou por que foi feita. O código já diz o que faz, e a justificativa da mudança pertence à resposta do chat ou à mensagem de commit — não ao arquivo. Isso vale para todo tipo de comentário: `//`, `/* */`, `<!-- -->`, JSDoc e XML docs.

Comentário só se justifica quando o código sozinho engana: uma decisão contra-intuitiva que alguém tentaria "consertar", um workaround de bug de terceiro, um invariante que não dá pra ler ali. Nesses casos, uma ou duas linhas, explicando o **porquê** — nunca o quê.

**Errado:**
```vue
<!-- Estado e cidade dividem a linha só a partir do `sm` — o mesmo ponto em que
     o `useIsMobile` para de valer e o modal deixa de ser fullscreen. Abaixo
     disso o select e o input ficam estreitos demais lado a lado. -->
<div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
```

```csharp
// Busca o curso pelo id
var course = await ctx.Courses.FirstOrDefaultAsync(x => x.Id == id);
```

**Correto:**
```vue
<div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
```

```csharp
// O Google devolve `email_verified` como string em alguns fluxos legados.
var verified = claim.Value is "true" or "True";
```

Na dúvida, não comentar.

## Backend conventions

### Checagem de strings — usar `HasValue()` / `IsEmpty()`

Em `if`s que checam strings, **sempre** usar as extensions `HasValue()` e `IsEmpty()` (definidas em `Back/Shared/Extensions/StringExtensions.cs`).
**Nunca** usar `string.IsNullOrEmpty`, `string.IsNullOrWhiteSpace` nem suas negações.

**Correto:**
```csharp
if (name.HasValue()) { ... }
if (name.IsEmpty()) { ... }
```

**Errado:**
```csharp
if (!string.IsNullOrEmpty(name)) { ... }
if (string.IsNullOrWhiteSpace(name)) { ... }
```

### Comentários XML (`<summary>` / `<remarks>`) — sempre multi-linha

Nos comentários XML dos controllers, **sempre** colocar as tags de abertura e fechamento em linhas próprias, com o conteúdo numa linha separada. **Nunca** colocar o conteúdo na mesma linha da tag.

**Correto:**
```csharp
/// <summary>
/// Matricular aluno em oferta de curso
/// </summary>
/// <remarks>
/// Vincula um aluno a uma oferta de curso, criando uma matrícula.
/// </remarks>
```

**Errado:**
```csharp
/// <summary>Matricular aluno em oferta de curso</summary>
/// <remarks>Vincula um aluno a uma oferta de curso, criando uma matrícula.</remarks>
```

### Enums — valor inteiro sempre explícito

Todo membro de enum **sempre** declara explicitamente seu valor inteiro. **Nunca** depender da numeração implícita do compilador: os valores são persistidos no banco e expostos na API, então reordenar ou inserir um membro no meio mudaria o significado dos dados já gravados.

Ao adicionar um membro novo, usar o próximo valor livre (nunca reaproveitar nem renumerar os existentes).

**Correto:**
```csharp
public enum ClassLessonStatus
{
    [Description("Pendente")]
    Pending = 0,

    [Description("Concluída")]
    Finalized = 1,
}
```

**Errado:**
```csharp
public enum ClassLessonStatus
{
    [Description("Pendente")]
    Pending,

    [Description("Concluída")]
    Finalized,
}
```

### LINQ — sempre method syntax, nunca query syntax

**Nunca** usar a query syntax do LINQ (`from ... join ... where ... select ...`). **Sempre** usar method syntax com as navigation properties do EF Core, deixando o EF montar os joins.

**Errado:**
```csharp
var schedules = await (
    from s in ctx.Schedules.AsNoTracking()
    join c in ctx.Classes.AsNoTracking() on s.ClassId equals (int?)c.Id
    where s.ClassroomId != null && classroomIds.Contains(s.ClassroomId.Value)
        && c.Id != id && c.Status != ClassStatus.Finalized
    select s
).ToListAsync();
```

**Correto:**
```csharp
var schedules = await ctx.Schedules.AsNoTracking()
    .Where(s => s.ClassroomId != null && classroomIds.Contains(s.ClassroomId.Value)
        && s.Class!.Id != id && s.Class.Status != ClassStatus.Finalized)
    .ToListAsync();
```

Se não existir navigation property para o relacionamento, criar uma no entity/config em vez de recorrer a `join`.

### Raw SQL — palavras-chave em maiúsculo

Em SQL escrito à mão (`FromSql`, `SqlQueryRaw`, Dapper, migrations), as palavras-chave do SQL **sempre** vão em
maiúsculo: `SELECT`, `FROM`, `WHERE`, `JOIN`, `ON`, `AND`, `OR`, `NOT`, `IS NULL`, `AS`, `GROUP BY`, `ORDER BY`,
`FILTER`... Nomes de tabelas, colunas e funções (`count`, `to_tsvector`, `unaccent`) continuam em minúsculo.

**Correto:**
```sql
SELECT * FROM estud.class_lessons
WHERE planned_content IS NOT NULL
  AND to_tsvector('portuguese', unaccent(planned_content)) @@ to_tsquery('simple', ...)
```

**Errado:**
```sql
select * from estud.class_lessons
where planned_content is not null
  and to_tsvector('portuguese', unaccent(planned_content)) @@ to_tsquery('simple', ...)
```

## Frontend conventions

### Zod validation — campos opcionais/undefined


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZaqueuCavalcante/estud](https://github.com/ZaqueuCavalcante/estud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
