---
trigger: always_on
description: - **Framework CSS** : Bootstrap 5.3+
---


# Règles Frontend

## Stack Frontend

- **Framework CSS** : Bootstrap 5.3+
- **Formulaires** : Crispy Forms avec crispy-bootstrap5
- **Templates** : Django Template Language
- **JavaScript** : Vanilla JS (pas de framework frontend)
- **Icônes** : Font Awesome (CDN)

## Templates Django - Structure

### Template de Base

Tous les templates doivent étendre `base.html` :

```django
{% extends 'base.html' %}
{% load static %}
{% load crispy_forms_tags %}

{% block title %}Mon Titre - Observations Nids{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{% static 'observations/css/custom.css' %}">
{% endblock %}

{% block content %}
<div class="container mt-4">
    <h1>Mon Contenu</h1>
</div>
{% endblock %}

{% block extra_js %}
<script src="{% static 'observations/js/custom.js' %}"></script>
{% endblock %}
```

### Règles de Structure

```django
{# ✅ BON - Espacement clair, commentaires utiles #}
{% extends 'base.html' %}

{# Section formulaire utilisateur #}
<div class="card">
    <div class="card-header">
        <h3>Formulaire</h3>
    </div>
    <div class="card-body">
        {% include 'partials/user_form.html' %}
    </div>
</div>

{# ❌ MAUVAIS - Pas de commentaires, imbrication complexe #}
<div><div><div><form>...</form></div></div></div>
```

## Bootstrap 5 - Conventions

### Grille et Layout

```django
{# Container principal #}
<div class="container">
    <div class="row">
        <div class="col-md-8">
            {# Contenu principal #}
        </div>
        <div class="col-md-4">
            {# Sidebar #}
        </div>
    </div>
</div>

{# Cards pour regrouper le contenu #}
<div class="card mb-3">
    <div class="card-header bg-primary text-white">
        <h4 class="mb-0">Titre</h4>
    </div>
    <div class="card-body">
        <p class="card-text">Contenu</p>
    </div>
    <div class="card-footer">
        <a href="#" class="btn btn-primary">Action</a>
    </div>
</div>
```

### Composants Standards

```django
{# Boutons avec classes sémantiques #}
<a href="{% url 'observations:creer_fiche' %}" class="btn btn-primary">
    <i class="fas fa-plus"></i> Nouvelle Fiche
</a>
<button type="submit" class="btn btn-success">
    <i class="fas fa-save"></i> Enregistrer
</button>
<a href="{% url 'observations:liste' %}" class="btn btn-secondary">
    <i class="fas fa-arrow-left"></i> Retour
</a>
<button type="button" class="btn btn-danger" onclick="confirmerSuppression()">
    <i class="fas fa-trash"></i> Supprimer
</button>

{# Badges pour les statuts #}
<span class="badge bg-success">Validé</span>
<span class="badge bg-warning">En attente</span>
<span class="badge bg-danger">Refusé</span>
<span class="badge bg-info">En cours</span>

{# Alerts pour les messages #}
<div class="alert alert-success alert-dismissible fade show" role="alert">
    <i class="fas fa-check-circle"></i> Opération réussie
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>
```

## Messages Django

### Affichage Automatique

```django
{# Dans base.html ou au début de chaque template #}
{% if messages %}
    <div class="messages-container">
        {% for message in messages %}
            <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                {% if message.tags == 'error' %}
                    <i class="fas fa-exclamation-circle"></i>
                {% elif message.tags == 'success' %}
                    <i class="fas fa-check-circle"></i>
                {% elif message.tags == 'warning' %}
                    <i class="fas fa-exclamation-triangle"></i>
                {% else %}
                    <i class="fas fa-info-circle"></i>
                {% endif %}
                {{ message }}
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        {% endfor %}
    </div>
{% endif %}
```

### Tags Bootstrap Standards

- `success` → `alert-success` (vert)
- `error` → `alert-danger` (rouge)
- `warning` → `alert-warning` (orange)
- `info` → `alert-info` (bleu)

## Crispy Forms - Formulaires

### Usage Standard

```django
{% load crispy_forms_tags %}

{# Formulaire complet avec Crispy #}
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form|crispy }}
    
    <div class="form-actions mt-3">
        <button type="submit" class="btn btn-primary">
            <i class="fas fa-save"></i> Enregistrer
        </button>
        <a href="{% url 'observations:liste' %}" class="btn btn-secondary">
            <i class="fas fa-times"></i> Annuler
        </a>
    </div>
</form>
```

### Personnalisation par Champ

```django
{% load crispy_forms_tags %}

<form method="post">
    {% csrf_token %}
    
    {# Champ individuel avec helper #}
    {% crispy form.username %}
    
    {# Layout personnalisé #}
    <div class="row">
        <div class="col-md-6">
            {% crispy form.first_name %}
        </div>
        <div class="col-md-6">
            {% crispy form.last_name %}
        </div>
    </div>
    
    {# Champ avec aide contextuelle #}
    <div class="mb-3">
        <label for="{{ form.email.id_for_label }}" class="form-label">
            Email <span class="text-danger">*</span>
        </label>
        {{ form.email }}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmFschneider) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
