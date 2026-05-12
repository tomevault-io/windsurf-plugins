---
trigger: always_on
description: Este documento define los subagentes especializados para el MCP Godot Python, un servidor que permite a LLMs interagir con proyectos Godot Engine.
---

# AGENTS.md - Subagentes del MCP Godot Python

Este documento define los subagentes especializados para el MCP Godot Python, un servidor que permite a LLMs interagir con proyectos Godot Engine.

---

## 📋 Subagentes Disponibles

| Agente | Descripción | Propósito Principal |
|--------|-------------|---------------------|
| `@Parser` | Especialista en parsing TSCN y formatos Godot | Analizar y manipular archivos .tscn, .gd, .tres |
| `@ArrayOps` | Operaciones quirúrgicas sobre arrays | Añadir/quitar/modificar elementos sin reescribir archivos |
| `@LSPClient` | Language Server Protocol de Godot | Autocompletado, hover, diagnósticos de GDScript |
| `@DAPClient` | Debug Adapter Protocol de Godot | Debugging, breakpoints, stack traces |
| `@CacheMaster` | Gestión de cache LRU e invalidación | Optimizar operaciones repetitivas |
| `@TemplateEngine` | Templates Jinja2 para nodos/scripts | Generar código GDScript automáticamente |
| `@ToolSmith` | Creación de herramientas FastMCP | Extender funcionalidades del servidor |
| `@SessionKeeper` | Gestión de sesiones ligeras | Manejar conexiones temporales |
| `@GodotSage` | Conocimiento profundo de Godot Engine | Consulta de documentación y mejores prácticas |

---

## 🎯 @Parser - Especialista en Parsing TSCN

### Responsabilidades
- Parsear archivos `.tscn` (formato de escena Godot)
- Parsear archivos `.gd` (scripts GDScript)
- Parsear archivos `.tres` (recursos Godot)
- Analizar estructura de nodos
- Validar sintaxis y referencias
- Extraer metadatos de recursos

### Cuándo Invocarlo
- Cuando necesitas analizar una escena existente
- Al crear o modificar nodos en tiempo real
- Para obtener información de recursos
- Al validar referencias entre escenas

### Herramientas MCP Usadas
```python
# Herramientas principales del Parser
- mcp__godot__get_scene_tree    # Obtener jerarquía de nodos
- mcp__godot__list_scenes     # Listar todas las escenas
- mcp__godot__find_nodes     # Buscar nodos por tipo
- mcp__godot__get_node_properties  # Propiedades de un nodo
```

### Flujo de Trabajo
```
1. Identificar tipo de archivo (.tscn, .gd, .tres)
2. Seleccionar parser apropiado
3. Parsear contenido estructura
4. Aplicar transformaciones necesarias
5. Serializar de vuelta al formato original
```

### Ejemplo de Uso
```python
# Parser TSCN básico
from godot_mcp.core.parser import TSCNParser

parser = TSCNParser()
scene_data = parser.parse("res://scenes/Player.tscn")
print(scene_data.nodes)  # {'root': 'CharacterBody2D', 'children': [...]}

# Encontrar nodos específicos
parser = TSCNParser()
nodes = parser.find_nodes("res://scenes/Player.tscn", "Area2D")
print(f"Encontrados {len(nodes)} nodos Area2D")
```

---

## ⚡ @CacheMaster - Gestión de Cache

### Responsabilidades
- Implementar cache LRU para operaciones frecuentes
- Invalidar cache cuando cambia el proyecto
- Gestionar TTL (Time-To-Live) de entradas
- Sincronizar cache entre sesiones
- Optimizar consultas repetitivas

### Cuándo Invocarlo
- Al acceder frecuentemente a los mismos recursos
- Para acelerar listados de escenas
- Cuando el proyecto tiene muchas escenas
- Para caching de metadata

### Herramientas MCP Usadas
```python
# Gestión de cache
- mcp__godot__get_scene_tree   # Cacheado
- mcp__godot__list_scenes    # Cacheado
- mcp__godot__find_nodes    # Cacheado
- mcp__godot__get_node_properties  # Cacheado
```

### Flujo de Trabajo
```
1. Verificar si entrada existe en cache
2. Si existe y es válida → retornar dato cacheado
3. Si no existe → ejecutar operación
4. Almacenar resultado en cache
5. Programar invalidación (TTL o手动)
```

### Ejemplo de Uso
```python
from godot_mcp.core.cache import CacheManager

cache = CacheManager(max_size=100, ttl=300)

# Primera llamada - cache miss
result = cache.get("scenes:/Player.tscn", fetch_scene_tree)

# Segunda llamada - cache hit (rápido)
result = cache.get("scenes:/Player.tscn", fetch_scene_tree)

# Invalidar cache manualmente
cache.invalidate("scenes:/Player.tscn")

# Limpiar todo el cache
cache.clear()
```

---

## 📝 @TemplateEngine - Motor de Templates

### Responsabilidades
- Generar código GDScript desde templates
- Crear estructuras de nodos prediseñadas
- Personalizar templates con variables
- Proveer librerías de templates comunes
- Versionar templates por tipo de nodo

### Cuándo Invocarlo
- Al crear nuevos scripts GDScript
- Para generar escenas con estructura común
- Al implementar patterns recurrentes
- Para crear boilerplate de nodos

### Herramientas MCP Usadas
```python
# Generación de código
- mcp__godot__create_scene   # Crear escena desde template
- mcp__godot__create_script  # Crear script desde template
- mcp__godot__add_node      # Añadir nodo generado
- mcp__godot__set_script    # Adjuntar script
```

### Flujo de Trabajo
```
1. Seleccionar template apropiado
2. Proveer variables de personalización
3. Renderizar template con Jinja2
4. Escribir archivo resultante
5. Registrar en cache de templates
```

### Ejemplo de Uso
```python
from godot_mcp.templates import TemplateEngine

engine = TemplateEngine()

# Template para Player Controller
player_script = engine.render("player_controller.gd.j2", {
    "class_name": "Player",
    "extends": "CharacterBody2D",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CerebroCanibalus/Ultra-Godot-MCP-ASL](https://github.com/CerebroCanibalus/Ultra-Godot-MCP-ASL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
