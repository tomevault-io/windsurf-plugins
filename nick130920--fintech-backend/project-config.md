---
trigger: always_on
description: Reglas de diseño y desarrollo para Money Flow Flutter App
---


# Money Flow - Reglas de Desarrollo Flutter

## 🚫 REGLAS ESTRICTAS - PROHIBIDO

### ❌ COLORES HARDCODEADOS
```dart
// ❌ NUNCA HACER ESTO
color: Colors.red
color: Color(0xFF123456)
backgroundColor: Colors.grey[50]
Colors.blue[600]

// ✅ SIEMPRE HACER ESTO
color: Theme.of(context).colorScheme.error
color: Theme.of(context).colorScheme.primary
backgroundColor: Theme.of(context).colorScheme.surface
```

### ❌ WIDGETS SIN CONTEXTO DE TEMA
```dart
// ❌ NUNCA HACER ESTO
const Text('Título', style: TextStyle(color: Colors.black))
Container(color: Colors.white)

// ✅ SIEMPRE HACER ESTO
Text('Título', style: TextStyle(color: Theme.of(context).colorScheme.onSurface))
Container(color: Theme.of(context).colorScheme.surfaceContainerHighest)
```

### ❌ ESPACIADO INCONSISTENTE
```dart
// ❌ NUNCA HACER ESTO
const SizedBox(height: 15)
const EdgeInsets.all(13)
const EdgeInsets.only(top: 18, left: 22)

// ✅ SIEMPRE HACER ESTO
const SizedBox(height: 16)  // 8, 16, 24, 32
const EdgeInsets.all(16)    // 8, 16, 24
```

---

## ✅ OBLIGATORIO - SIEMPRE USAR

### 📱 Estructura de Pantalla Estándar
```dart
Scaffold(
  backgroundColor: Theme.of(context).colorScheme.surface,
  appBar: AppBar(
    title: const Text('Título'),
    backgroundColor: Colors.transparent,
    elevation: 0,
    actions: [
      TextButton(
        onPressed: onSave,
        child: const Text(
          'Guardar',
          style: TextStyle(
            fontSize: 16,
            fontWeight: FontWeight.w600,
          ),
        ),
      ),
    ],
  ),
  body: SingleChildScrollView(
    padding: const EdgeInsets.all(24),
    child: Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _buildHeader(),
          const SizedBox(height: 32),
          // Contenido...
        ],
      ),
    ),
  ),
)
```

### 🎨 Sistema de Colores
```dart
// Colores principales
Theme.of(context).colorScheme.primary          // AppColors.primary
Theme.of(context).colorScheme.onPrimary        // AppColors.white
Theme.of(context).colorScheme.primaryContainer // Para badges/chips

// Colores de superficie
Theme.of(context).colorScheme.surface                    // Fondo principal
Theme.of(context).colorScheme.surfaceContainerHighest   // Cards/inputs
Theme.of(context).colorScheme.surfaceContainerHigh      // Contenedores
Theme.of(context).colorScheme.surfaceContainerLow       // Secciones

// Colores de texto
Theme.of(context).colorScheme.onSurface                        // Texto principal
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.8) // Texto secundario
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6) // Texto terciario
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.4) // Placeholders

// Colores semánticos
Theme.of(context).colorScheme.error           // Errores
Theme.of(context).colorScheme.errorContainer  // Fondo error
Theme.of(context).colorScheme.outline         // Bordes
```

### 📏 Espaciado Estándar
```dart
// Espaciado vertical
const SizedBox(height: 8)     // Pequeño
const SizedBox(height: 16)    // Mediano
const SizedBox(height: 24)    // Grande
const SizedBox(height: 32)    // Extra grande

// Padding de contenedores
const EdgeInsets.all(8)       // Pequeño
const EdgeInsets.all(16)      // Estándar
const EdgeInsets.all(24)      // Screen padding

// Border radius
BorderRadius.circular(8)      // Pequeño
BorderRadius.circular(12)     // Estándar
BorderRadius.circular(16)     // Grande
BorderRadius.circular(20)     // Modal tops
```

---

## 🧩 COMPONENTES OBLIGATORIOS

### 1. Header con Ícono
```dart
Widget _buildHeader() {
  return Row(
    children: [
      Container(
        width: 48,
        height: 48,
        decoration: BoxDecoration(
          color: Theme.of(context).colorScheme.primaryContainer, // Para ingresos
          // color: Theme.of(context).colorScheme.errorContainer, // Para gastos
          borderRadius: BorderRadius.circular(12),
        ),
        child: Icon(
          Icons.trending_up, // trending_up para ingresos, receipt para gastos
          color: Theme.of(context).colorScheme.onPrimaryContainer,
          size: 24,
        ),
      ),
      const SizedBox(width: 16),
      Expanded(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Título Principal',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
                color: Theme.of(context).colorScheme.onSurface,
              ),
            ),
            Text(
              'Subtítulo descriptivo',
              style: TextStyle(
                fontSize: 14,
                color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6),
              ),
            ),
          ],
        ),
      ),
    ],
  );
}
```

### 2. Campo de Formulario Estándar
```dart
Widget _buildFormField(String label, String hint, TextEditingController controller) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(
        label,
        style: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.w600,
          color: Theme.of(context).colorScheme.onSurface,
        ),
      ),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
