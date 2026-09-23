---
trigger: always_on
description: Ce document sert de référence de développement et de règles pour la conception visuelle et l'interaction de l'application. Toutes les modifications ou ajouts d'interface doivent impérativement respecter les règles énoncées ci-dessous.
---

# Consignes de Design et d'Ergonomie Android

Ce document sert de référence de développement et de règles pour la conception visuelle et l'interaction de l'application. Toutes les modifications ou ajouts d'interface doivent impérativement respecter les règles énoncées ci-dessous.

---

## 1. La règle fondamentale : les zones tactiles

La pierre angulaire de toute interface Android est la taille de la zone tactile (touch target). 
* **Taille minimale :** La spécification officielle de Material Design impose un minimum de **48 dp x 48 dp**, ce qui correspond physiquement à environ 9 millimètres sur un écran standard. Cette dimension est calculée pour que le pouce d'un adulte puisse appuyer sans hésitation.
* **Espacement minimal :** L'espacement minimum absolu entre deux zones tactiles est de **8 dp**, afin d'éviter les clics intempestifs.
* **Zone visuelle vs Zone tactile :** La zone visuelle d'un élément (par exemple, une petite icône) peut être bien inférieure à 48 dp. Pour étendre la surface réactive autour de l'élément graphique sans déformer le rendu visuel, il faut utiliser :
  * Un **padding** (remplissage interne) généreux.
  * Dans Jetpack Compose, utiliser des modificateurs comme `.defaultMinSize(minWidth = 48.dp, minHeight = 48.dp)` ou des dimensions de conteneur appropriées pour s'assurer que la cible fait au moins 48 dp.
* **Normes d'accessibilité (WCAG 2.2) :** Le seuil absolu est de 44 x 44 pixels CSS, mais Android reste plus exigeant avec ses 48 dp.

---

## 2. Les icônes et leurs formats

* **Taille d'affichage standard :** **24 dp x 24 dp** pour presque tous les éléments de navigation (barre d'action, onglets, boutons d'action flottants).
* **Icônes d'application (App Icon) :** Se dessinent sur une base de **108 x 108 dp** pour fournir un rendu net sur toutes les densités d'écran.
* **Icônes de notification :** Plus petites, s'établissent à **22 x 22 dp**.
* **Résolutions et Densités (Production) :** Les icônes doivent être déclinées dans plusieurs résolutions pour correspondre aux différentes densités d'écran. Pour une icône standard de 24 dp, voici les équivalences en pixels :
  * **MDPI :** 48 px (1x)
  * **HDPI :** 72 px (1.5x)
  * **XHDPI :** 96 px (2x)
  * **XXHDPI :** 144 px (3x)
  * **XXXHDPI :** 192 px (4x)
* **Note importante :** Même si l'icône visuelle fait 24 dp, sa zone tactile réactive doit être étendue à 48 dp minimum.

---

## 3. La typographie en profondeur

Les textes se mesurent en **sp** (scaleable pixels) pour s'adapter aux préférences de taille de police définies par l'utilisateur.
* **Taille minimale absolue :** **12 sp**. Ne jamais descendre en dessous sous peine de rendre le texte illisible et de pénaliser gravement l'accessibilité.
* **Corps de texte (Body) :** Recommandé à **14 sp** pour le contenu courant, les descriptions ou articles.
* **Champs de saisie & listes principales :** Recommandé à **16 sp** pour garantir un confort de lecture optimal.
* **Boutons et Onglets :** Généralement **14 sp** en gras (Medium / Bold).
* **Hiérarchie Typographique (Material Design 3) :** 5 rôles clés : *Display, Headline, Title, Label* et *Body* (chacun décliné en grand, moyen, petit).
* **Mise en page résiliente :** Tout texte doit pouvoir s'agrandir jusqu'à **200 %** de sa taille initiale sans briser la mise en page ni masquer de contenu.
* **Contraste de couleur (WCAG) :**
  * **Texte normal (moins de 18 sp ou 14 sp en gras) :** Rapport de contraste d'au moins **4,5:1** avec son arrière-plan.
  * **Texte large (à partir de 18 sp) :** Rapport de contraste minimal de **3:1**.

---

## 4. Les dimensions des composants d'interface

Chaque composant graphique possède ses propres spécifications physiques standard :
* **Boutons standards :** Hauteur de zone tactile de **48 dp** minimum.
* **Boutons d'Action Flottants (FAB) :**
  * Taille standard : **56 x 56 dp**
  * Version mini : **40 x 40 dp**
* **Champs de saisie (TextField / TextInputLayout) :** Hauteur comprise entre **48 dp et 56 dp**, avec texte de saisie en **16 sp**.
* **Interrupteurs, cases à cocher, boutons radio :** Bénéficient de zones tactiles de **48 dp** (gérées généralement par le framework).
* **Curseurs (Sliders) :** La zone de préhension (le pouce ou thumb) doit faire au moins **48 dp**.
* **Barre de navigation inférieure (Bottom Navigation) :** Hauteur comprise entre **48 dp et 56 dp**.
* **Barre d'application principale (App Bar) :** **56 dp** en mode portrait, **48 dp** en mode paysage.
* **Éléments de liste (List items) :** Hauteur minimale de **48 dp** (recommandé **56 dp** pour plus de confort tactile).
* **Onglets (Tabs) :** Hauteur de **48 dp**.
* **Fenêtres modales (Dialogs) :** Conserver des marges de sécurité latérales de **16 à 24 dp**.
* **Cartes (Cards) :** Angles arrondis compris entre **4 dp et 12 dp** selon le style recherché.
* **Badges de notification :** Diamètre compris entre **16 dp et 24 dp**.

---

## 5. Les règles de mise en page et d'espacement

L'architecture visuelle repose sur une **grille incrémentale de 8 dp** :

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathieuandrianarivelo10-cell/Fluxio](https://github.com/mathieuandrianarivelo10-cell/Fluxio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
