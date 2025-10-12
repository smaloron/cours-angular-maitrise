# Bonus 1 : L'Art de l'Animation avec @angular/animations - Pour aller plus loin

### Objectifs pédagogiques

À la fin de cette partie, vous serez capable de :

* **Orchestrer** des animations multiples en utilisant `group()` (parallèle) et `sequence()` (séquentiel).
* **Créer** des animations complexes en cascade sur des listes avec `query()` et `stagger()`.
* **Implémenter** des transitions de route animées pour rendre la navigation entre les pages plus fluide.
* **Interagir** avec le cycle de vie des animations grâce aux callbacks `(@trigger.done)`.

### Introduction

Dans la partie "L'essentiel", nous avons appris à faire danser un soliste. C'était élégant, mais une vraie production
nécessite un corps de ballet. Comment faire danser plusieurs éléments ensemble ? Comment créer un effet de domino où les
éléments d'une liste apparaissent les uns après les autres avec un léger décalage ?

C'est là que les fonctions de chorégraphie d'Angular entrent en scène. Elles nous permettent de passer du statut de
simple chorégraphe à celui de metteur en scène, capable de diriger des séquences d'animation complexes. Nous allons
apprendre à grouper des animations, à les séquencer, et à cibler des éléments enfants à l'intérieur d'un conteneur
animé.

Pour couronner le tout, nous nous attaquerons à l'une des animations les plus impressionnantes : l'animation des
transitions de route. Fini les changements de page brusques ; nous allons créer des fondus enchaînés ou des glissements
qui donneront à notre Single Page Application l'aspect d'une application native parfaitement polie.

### 1. Animations Complexes et Chorégraphies

Lorsque vous placez plusieurs appels à `animate()` dans une transition, ils s'exécutent par défaut en séquence. Mais
pour des contrôles plus fins, nous avons des outils dédiés.

<tabs>

<tab title="`group()` : Tous ensemble !">

La fonction `group()` permet d'exécuter plusieurs animations **en parallèle**. C'est utile si vous voulez animer plusieurs propriétés en même temps, potentiellement avec des durées ou des timings différents.

```typescript
// Fait un fondu et un changement de taille en même temps
transition(':enter', [
    style({opacity: 0, height: '0px'}),
    group([
        animate('500ms ease-out', style({height: '*'})), // '*' = valeur calculée
        animate('800ms ease-out', style({opacity: 1}))
    ])
])
```

</tab>

<tab title="`query()` et `stagger()` : L'Effet Domino">

Ce duo est magique pour animer les éléments d'une liste.
*   **`query(':enter', ...)` :** Permet de trouver des éléments à l'intérieur de l'élément qui porte le `trigger`. On peut cibler des éléments qui entrent (`:enter`), qui sortent (`:leave`), ou n'importe quel sélecteur CSS.
*   **`stagger(temps, ...)` :** Applique un délai entre les animations de chaque élément trouvé par `query()`.

```typescript
// Animation attachée au conteneur <ul> ou <div>
trigger('listStagger', [
    transition('* => *', [ // S'exécute à chaque changement dans la liste
        // On cible les nouveaux éléments qui entrent dans le DOM
        query(':enter', [
            // On les rend invisibles et décalés au départ
            style({opacity: 0, transform: 'translateY(-50px)'}),
            // On applique un délai de 100ms entre l'animation de chaque li
            stagger('100ms', [
                animate('500ms ease-out', style({
                    opacity: 1,
                    transform: 'none'
                }))
            ])
        ], {optional: true}) // optional: true évite les erreurs si aucun élément n'est trouvé
    ])
])
```

</tab>
</tabs>

### 2. Animer les Transitions de Route

C'est une technique avancée qui donne un cachet très professionnel à une application. Le principe est d'attacher un
`trigger` d'animation au conteneur du `<router-outlet>`.

**Le processus :**

1. **Préparer le routeur :** Chaque route doit avoir une donnée (`data`) qui servira à identifier l'état d'animation.
2. **Préparer le template :** Le `<router-outlet>` doit être dans un conteneur qui portera le `trigger`. On passe l'état
   de la route au `trigger`.
3. **Créer l'animation :** On utilise `query(':enter')` et `query(':leave')` pour cibler la page qui arrive et celle qui
   part. On utilise `group()` pour les animer en même temps.

**1. `app.routes.ts`**

```typescript
// ...
export const routes: Routes = [
    {path: 'home', component: HomeComponent, data: {animation: 'HomePage'}},
    {path: 'about', component: AboutComponent, data: {animation: 'AboutPage'}}
];
```

**2. `app.component.html`**

```html

<div [@routeAnimations]="getRouteAnimationData()">
    <router-outlet></router-outlet>
</div>
```

**3. `app.component.ts`**

```typescript
// ...
@Component({
    // ...
    animations: [slideInAnimation] // On importe notre animation
})
export class AppComponent {
    private contexts = inject(ChildrenOutletContexts);

    getRouteAnimationData() {
        // Récupère la donnée 'animation' de la route active
        return this.contexts.getContext('primary')?.route?.snapshot?.data?.['animation'];
    }
}
```

**4. L'animation `slideInAnimation`**

```typescript
// animations.ts
trigger('routeAnimations', [
    // Transition entre n'importe quels états
    transition('* <=> *', [
        // La page doit être positionnée en absolu pour que
        // l'ancienne et la nouvelle puissent coexister pendant l'animation
        style({position: 'relative'}),

        // On cible la nouvelle page qui arrive (:enter) et l'ancienne qui part (:leave)
        query(':enter, :leave', [
            style({
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%'
            })
        ], {optional: true}),

        query(':enter', [
            // On fait entrer la nouvelle page depuis la droite
            style({left: '100%'})
        ], {optional: true}),

        group([
            // On fait sortir l'ancienne page vers la gauche
            query(':leave', [
                animate('300ms ease-out', style({left: '-100%'}))
            ], {optional: true}),

            // On fait entrer la nouvelle page au centre
            query(':enter', [
                animate('300ms ease-out', style({left: '0%'}))
            ], {optional: true})
        ])
    ])
]);
```

### 3. Callbacks d'Animation

Parfois, on a besoin de savoir quand une animation commence ou se termine pour exécuter une action. Le `trigger`
d'animation émet des événements.

* `(@nomDuTrigger.start)`: Émis au début de l'animation.
* `(@nomDuTrigger.done)`: Émis à la fin de l'animation.

```html

<div @myAnimation (@myAnimation.done)="onAnimationFinished($event)">
    ...
</div>
```

```typescript
// Dans le composant
onAnimationFinished(event
:
AnimationEvent
)
{
    // L'objet event contient des infos utiles :
    // event.fromState, event.toState, event.totalTime...
    console.log('Animation terminée !', event);

    // Cas d'usage : après une animation de sortie (:leave),
    // on peut maintenant supprimer l'objet du tableau de données.
    if (event.toState === 'void') {
        this.removeItemFromBackend(this.itemToRemove);
    }
}
```

### Exercice 9.2 : Créer un effet "Stagger" sur une galerie d'images

**Objectif :** Créer une galerie d'images où les cartes apparaissent en cascade les unes après les autres.

**Instructions :**

1. Créez un composant `ImageGalleryComponent`.
2. Dans ce composant, ayez un tableau d'URLs d'images.
3. Dans le template, utilisez `@for` pour afficher ces images dans des cartes (des `div` stylisées).
4. Créez une animation `galleryStagger` que vous attacherez au **conteneur** de la galerie.
5. Cette animation doit se déclencher lorsque la page se charge (`:enter`).
6. À l'intérieur, utilisez `query()` pour cibler chaque carte d'image qui entre (`query(':enter', ...)`).
7. Utilisez `stagger()` pour animer l'apparition de chaque carte avec un décalage de 150ms. L'animation de chaque carte
   sera un simple fondu (`opacity`) et un léger zoom (`transform: scale(0.9)` vers `scale(1)`).

#### Correction exercice 9.2 {collapsible='true'}

<procedure>
<p>Voici l'implémentation complète de la galerie avec effet "stagger".</p>

**1. `src/app/animations.ts` (ajout de la nouvelle animation)**

```typescript
// ...
export const galleryStagger = trigger('galleryStagger', [
    transition(':enter', [
        // On cible les éléments enfants avec la classe .gallery-item
        // qui entrent dans le DOM.
        query('.gallery-item', [
            style({opacity: 0, transform: 'scale(0.8)'}),
            stagger('150ms', [
                animate('400ms ease-out', style({
                    opacity: 1,
                    transform: 'scale(1)'
                }))
            ])
        ], {optional: true})
    ])
]);
```

**2. `src/app/image-gallery/image-gallery.component.ts`**

```typescript
import {Component} from '@angular/core';
import {CommonModule} from '@angular/common';
import {galleryStagger} from '../animations';

@Component({
    selector: 'app-image-gallery',
    standalone: true,
    imports: [CommonModule],
    template: `
    <h2>Galerie Animée</h2>
    <!-- On attache l'animation au conteneur -->
    <div class="gallery-container" @galleryStagger>
      @for (image of images; track image) {
        <div class="gallery-item">
          <img [src]="image" alt="Image de galerie">
        </div>
      }
    </div>
  `,
    styleUrls: ['./image-gallery.component.css'],
    animations: [galleryStagger] // On déclare l'animation
})
export class ImageGalleryComponent {
    images = [
        'https://via.placeholder.com/150/92c952',
        'https://via.placeholder.com/150/771796',
        'https://via.placeholder.com/150/24f355',
        'https://via.placeholder.com/150/d32776',
        'https://via.placeholder.com/150/f66b97',
        'https://via.placeholder.com/150/56a8c2',
    ];
}
```

**3. `src/app/image-gallery/image-gallery.component.css` (pour le visuel)**

```css
.gallery-container {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.gallery-item {
    width: 150px;
    height: 150px;
    border: 1px solid #ccc;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.gallery-item img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
```

</procedure>

### Auto-évaluation

1. **Quelle fonction de l'API d'animation utiliseriez-vous pour animer un changement d'opacité ET un mouvement en même
   temps ?**<br/>
   a. `sequence()`<br/>
   b. `query()`<br/>
   c. `group()`<br/>
   d. `stagger()`<br/>

2. **Pour créer un effet où les éléments d'une liste apparaissent les uns après les autres, quelle fonction est
   spécifiquement conçue pour introduire un délai entre chaque animation ?**

3. **Lors de l'animation d'une transition de route, quel est le rôle de la propriété `data: { animation: '...' }` dans
   la configuration d'une route ?**<br/>
   a. Elle définit la durée de l'animation.<br/>
   b. Elle fournit un nom d'état unique pour la route, que le `trigger` d'animation peut utiliser pour détecter un
   changement.<br/>
   c. Elle charge dynamiquement le code de l'animation.<br/>
   d. Elle n'est utilisée que pour le débogage.<br/>

4. **Si vous voulez exécuter du code TypeScript une fois qu'une animation est terminée, quel mécanisme utilisez-vous ?**

5. **Que fait la fonction `query(':leave', ...)` dans une transition d'animation ?**

### Conclusion

Vous êtes maintenant un metteur en scène d'interfaces. Vous ne vous contentez plus d'animer des éléments isolés ; vous
orchestrez de véritables ballets visuels. Vous savez comment créer des animations parallèles et séquentielles, comment
produire des effets en cascade impressionnants, et comment rendre l'expérience de navigation de votre application
exceptionnellement fluide avec des transitions de route.

Cette maîtrise des animations est souvent ce qui distingue une application "correcte" d'une application "wow". C'est la
touche finale qui démontre un grand souci du détail et de l'expérience utilisateur.

Ce module conclut notre exploration approfondie du framework Angular. Avec toutes les compétences que vous avez
acquises, de l'architecture à l'animation, vous êtes maintenant parfaitement équipé pour concevoir et développer des
applications web modernes, complexes et de haute qualité.