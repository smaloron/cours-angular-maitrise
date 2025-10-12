# Module 7 : Pour aller plus loin - Bâtir pour Tous : L'Accessibilité (a11y)

### Objectifs pédagogiques

À la fin de cette partie, vous serez capable de :

* **Comprendre** l'importance de l'accessibilité (a11y) et son impact sur les utilisateurs.
* **Utiliser** le HTML sémantique comme fondation pour une application accessible.
* **Enrichir** la sémantique de vos composants avec les attributs ARIA (`role`, `aria-*`).
* **Assurer** une navigation au clavier logique et fonctionnelle.
* **Identifier** et **corriger** des problèmes d'accessibilité de base, comme le contraste des couleurs, en utilisant
  des outils d'audit.

### Introduction

Votre application est maintenant internationale. Elle parle plusieurs langues. Mais est-ce qu'elle parle la langue de
tout le monde, y compris des 15% à 20% de la population qui vivent avec une forme de handicap ? Un utilisateur malvoyant
peut-il la naviguer avec un lecteur d'écran ? Une personne ayant des troubles moteurs peut-elle l'utiliser sans souris ?

C'est l'enjeu de l'**accessibilité**, abrégée **a11y** (car il y a 11 lettres entre le 'a' et le 'y'). L'accessibilité
n'est pas une fonctionnalité optionnelle ou une "case à cocher" à la fin d'un projet. C'est un aspect fondamental de la
qualité logicielle, au même titre que la performance ou la sécurité. Bâtir des applications accessibles, ce n'est pas
seulement une question de conformité légale ou d'éthique, c'est aussi une excellente pratique business : une application
plus accessible est souvent une application mieux conçue, plus facile à utiliser pour **tous**, et avec un meilleur
référencement.

Heureusement, Angular et les plateformes web modernes nous fournissent tous les outils pour construire des expériences
accessibles dès le départ.

### 1. La Fondation : Le HTML Sémantique

La première règle de l'accessibilité est simple : **utilisez les éléments HTML pour ce pour quoi ils ont été conçus**.
C'est la base sur laquelle les technologies d'assistance (comme les lecteurs d'écran) s'appuient pour comprendre la
structure de votre page.

**MAUVAIS : La "div-ite"**

```html

<div class="header">Mon Site</div>
<div class="nav">
    <div class="nav-item">Accueil</div>
</div>
<div class="main-content">
    <div class="title">Titre de l'article</div>
    <div class="button" (click)="doSomething()">Cliquez ici</div>
</div>
```

Un lecteur d'écran voit ici un "groupe", un "groupe", un "texte", un "texte", un "groupe cliquable"... C'est vague et
peu informatif.

**BON : Le HTML Sémantique**

```html

<header>Mon Site</header>
<nav>
    <ul>
        <li><a href="/">Accueil</a></li>
    </ul>
</nav>
<main>
    <article>
        <h1>Titre de l'article</h1>
        <button (click)="doSomething()">Cliquez ici</button>
    </article>
</main>
```

Ici, le lecteur d'écran annonce : "en-tête", "navigation", "liste", "lien", "contenu principal", "article", "titre de
niveau 1", "bouton". L'utilisateur comprend immédiatement la structure et le rôle de chaque élément.

### 2. Enrichir la Sémantique avec ARIA

Parfois, le HTML sémantique ne suffit pas, notamment lorsqu'on crée des composants complexes qui n'ont pas d'équivalent
natif (un carrousel, un onglet, un menu déroulant...). C'est là qu'intervient **ARIA (Accessible Rich Internet
Applications)**.

ARIA est un ensemble d'attributs HTML qui vous permettent d'ajouter des informations sémantiques supplémentaires. Les
trois plus importants sont :

<tabs>
<tab title="`role` : Quel est ton métier ?">
L'attribut `role` définit le "métier" d'un élément. Par exemple, si vous construisez un système d'onglets avec des `div`, vous devez le dire au lecteur d'écran.

```html

<div role="tablist">
    <div role="tab" [attr.aria-selected]="isSelected(0)" (click)="select(0)">
        Onglet 1
    </div>
    <div role="tab" [attr.aria-selected]="isSelected(1)" (click)="select(1)">
        Onglet 2
    </div>
</div>
<div role="tabpanel">
    Contenu de l'onglet sélectionné
</div>
```

</tab>
<tab title="Propriétés `aria-*` : Quel est ton état ?">

Les attributs `aria-*` décrivent l'état actuel d'un composant.
*   `aria-selected`: L'onglet est-il sélectionné ? (`true`/`false`)
*   `aria-expanded`: Le menu est-il déplié ? (`true`/`false`)
*   `aria-label`: Pour donner une étiquette explicite à un élément qui n'a pas de texte visible (ex: un bouton avec juste une icône).
*   `aria-labelledby`: Pour lier un élément à un autre qui lui sert de titre.

```html
<!-- Un bouton accordéon -->
<button [attr.aria-expanded]="isExpanded" (click)="toggle()">
    Détails de la commande
</button>
<div [hidden]="!isExpanded">
    <!-- contenu de l'accordéon -->
</div>

<!-- Un bouton avec une icône seulement -->
<button aria-label="Fermer la fenêtre" (click)="close()">
    <mat-icon>close</mat-icon>
</button>
```

</tab>
<tab title="Relations `aria-*` : Qui sont tes collègues ?">
Certains attributs `aria-*` décrivent les relations entre les éléments.
*   `aria-controls`: Indique quel élément est contrôlé par le bouton actuel.

```html

<button [attr.aria-controls]="'panel-1'" (click)="toggle()">Toggle</button>
<div id="panel-1">...</div>
```

</tab>
</tabs>

### 3. Assurer la Navigabilité au Clavier

Tous les éléments interactifs (`a`, `button`, `input`, `select`...) doivent être accessibles et utilisables **uniquement
au clavier**.

* **Focus visible :** Ne supprimez **jamais** l'indicateur de focus du navigateur (`outline: none;`) sans fournir une
  alternative claire (ex: `outline: 2px solid blue;`). L'utilisateur doit savoir où il se trouve sur la page.
* **Ordre logique :** L'ordre de tabulation (la séquence dans laquelle on passe d'un élément à l'autre avec la touche
  `Tab`) doit suivre l'ordre visuel de la page. En général, si votre structure DOM est logique, c'est automatique.
* **Éléments cliquables non natifs :** Si vous rendez une `div` ou un `span` cliquable avec `(click)`, il n'est pas
  focusable par défaut. Vous devez lui ajouter `tabindex="0"` pour l'insérer dans l'ordre de tabulation et gérer les
  événements clavier `(keydown.enter)` et `(keydown.space)`. (Mais la meilleure solution reste d'utiliser un
  `<button>`).

### 4. Contraste des Couleurs et Outils d'Audit

Un texte est illisible si son contraste avec l'arrière-plan est trop faible. Les standards du WCAG (Web Content
Accessibility Guidelines) recommandent un ratio de contraste d'au moins **4.5:1** pour le texte normal.

Comment vérifier tout cela ? Heureusement, il existe d'excellents outils.

* **Lighthouse :** Intégré directement dans les DevTools de Chrome/Edge. Lancez un audit et il vous donnera un score
  d'accessibilité avec des pistes d'amélioration concrètes.
* **axe DevTools :** Une extension de navigateur qui analyse la page actuelle et liste les problèmes d'accessibilité
  avec des instructions claires pour les corriger.

### Exercice 7.2 : Rendre un composant "toggle" accessible

**Objectif :** Prendre un composant de "bouton à bascule" simple et le rendre entièrement accessible.

**Composant de départ (non accessible) :**

```typescript
// src/app/toggle-switch/toggle-switch.component.ts
@Component({
    selector: 'app-toggle-switch',
    standalone: true,
    template: `
    <div class="switch-container" (click)="toggle()">
      <div class="switch-label">Notifications</div>
      <div class="switch">
        <div class="switch-thumb" [class.on]="isOn"></div>
      </div>
    </div>
  `,
    // ... styles ...
})
export class ToggleSwitchComponent {
    isOn = false;

    toggle() {
        this.isOn = !this.isOn;
    }
}
```

**Problèmes :**

* C'est une `div` cliquable, donc pas accessible au clavier.
* Il n'a aucune sémantique. Un lecteur d'écran ne sait pas que c'est un interrupteur.

**Instructions :**

1. **Changez la sémantique :** Remplacez la `div` conteneur par un `<button>`.
2. **Ajoutez le `role` ARIA** approprié pour un interrupteur (`role="switch"`).
3. **Ajoutez l'état ARIA** qui indique si l'interrupteur est activé ou non (`aria-checked`).
4. **Assurez-vous** que le texte "Notifications" sert bien d'étiquette accessible pour le bouton.

#### Correction exercice 7.2 {collapsible='true'}

<procedure>
<p>Voici la version accessible du composant. Les changements sont subtils dans le template mais fondamentaux pour l'accessibilité.</p>

```typescript
// src/app/toggle-switch/toggle-switch.component.ts
import {Component} from '@angular/core';
import {CommonModule} from '@angular/common';

@Component({
    selector: 'app-toggle-switch',
    standalone: true,
    imports: [CommonModule],
    template: `
    <!-- 
      On utilise un <button> pour la navigabilité clavier et la sémantique native.
      On applique les attributs ARIA pour décrire le rôle et l'état.
    -->
    <button 
      class="switch-container" 
      role="switch"
      [attr.aria-checked]="isOn"
      (click)="toggle()">
      
      <span class="switch-label">Notifications</span>
      
      <span class="switch">
        <span class="switch-thumb" [class.on]="isOn"></span>
      </span>
    </button>
  `,
    styleUrls: ['./toggle-switch.component.css']
})
export class ToggleSwitchComponent {
    isOn = false;

    toggle() {
        this.isOn = !this.isOn;
    }
}
```

**Pourquoi c'est mieux :**

* **`<button>` :** L'élément est nativement focusable avec `Tab` et activable avec `Entrée`/`Espace`.
* **`role="switch"` :** Un lecteur d'écran annonce maintenant "Notifications, interrupteur".
* **`[attr.aria-checked]="isOn"` :** Le lecteur d'écran annonce l'état : "activé" ou "désactivé". L'utilisateur sait
  comment interagir et quel est l'état actuel.

</procedure>

### Auto-évaluation

1. **Pourquoi est-il préférable d'utiliser `<button>` plutôt qu'une `<div (click)="...">` pour créer un bouton ?**
   a. C'est plus court à écrire.
   b. C'est plus facile à styliser avec CSS.
   c. `<button>` est nativement focusable, utilisable au clavier, et a une sémantique correcte pour les technologies
   d'assistance.
   d. Les `div` cliquables sont dépréciées dans le HTML6.

2. **À quoi sert l'attribut ARIA `role` ?**

3. **Vous avez un bouton qui ne contient qu'une icône (une loupe). Comment vous assurez-vous qu'un lecteur d'écran
   annonce "Rechercher" lorsque l'utilisateur arrive sur ce bouton ?**
   a. `title="Rechercher"`
   b. `aria-label="Rechercher"`
   c. `role="Rechercher"`
   d. `i18n="Rechercher"`

4. **Quel est le ratio de contraste de couleurs minimum recommandé par le WCAG pour du texte de taille normale ?**

5. **Quel outil, intégré aux navigateurs basés sur Chromium, permet de lancer un audit d'accessibilité rapide sur votre
   page ?**

### Conclusion

Vous avez ajouté une dimension essentielle à votre expertise de développeur : la capacité de construire des applications
pour tout le monde. L'accessibilité n'est pas un domaine obscur réservé à quelques experts. Elle repose sur des
principes solides et des outils concrets que vous maîtrisez maintenant : **HTML sémantique**, **attributs ARIA**, et *
*audits automatiques**.

En intégrant ces pratiques dans votre flux de travail quotidien, vous ne ferez pas que répondre à des exigences légales
ou éthiques. Vous construirez des produits de meilleure qualité, plus robustes et plus agréables à utiliser pour
l'ensemble de vos utilisateurs.

Nous arrivons au terme de notre parcours principal. La prochaine étape consistera à valider nos applications de bout en
bout avec les **tests End-to-End**, puis à jeter un œil aux sujets qui feront de vous un véritable expert de
l'écosystème Angular.