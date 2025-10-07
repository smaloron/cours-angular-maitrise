# Module 1 : Pour aller plus loin - L'Art de la Composition et de la Résilience

### Objectifs pédagogiques

À la fin de cette partie, vous aurez ajouté de puissants outils RxJS à votre arsenal :

* **Implémenter** des stratégies de gestion d'erreurs robustes avec `catchError` et `retry`.
* **Exécuter** des actions de nettoyage (comme cacher un loader) de manière fiable avec `finalize`.
* **Combiner** plusieurs flux de données en parallèle avec `forkJoin` pour optimiser les chargements.
* **Synchroniser** des flux réactifs avec `combineLatest` pour construire des interfaces dynamiques.
* **Construire** un dashboard complexe qui charge plusieurs sources de données et gère un état de chargement global.

### Introduction

Dans la partie "L'essentiel", nous avons appris à diriger un musicien solo. Nous avons pris un flux de données et
l'avons transformé avec `switchMap` ou `concatMap`. C'est déjà une grande réussite.

Maintenant, il est temps de devenir le chef d'orchestre. Que se passe-t-il si un musicien joue une fausse note ? (
Gestion des erreurs). Que se passe-t-il lorsque vous devez faire jouer plusieurs sections de l'orchestre en même temps
pour atteindre un crescendo spectaculaire ? (Combinaison de flux).

Cette partie est dédiée à la résilience et à la composition. Une application professionnelle ne doit pas s'effondrer au
premier pépin réseau. Elle doit gérer les erreurs avec grâce. De même, une interface utilisateur riche dépend souvent de
plusieurs sources de données qui doivent être chargées et synchronisées. Apprendre à maîtriser ces techniques est ce qui
sépare une application fonctionnelle d'une application véritablement robuste et impressionnante.

### Gestion Avancée des Erreurs et de la Résilience

Un `Observable` qui rencontre une erreur (via la méthode `error()`) est un `Observable` mort. Il s'arrête net et ne
produira plus jamais de valeurs. Si vous ne gérez pas cette erreur, elle peut se propager et "casser" votre chaîne
d'opérateurs, laissant votre UI dans un état incohérent. Heureusement, RxJS nous fournit les outils pour nous défendre.

<tabs>
<tab title="catchError : L'airbag">
L'opérateur `catchError` est votre "try-catch" pour les `Observables`. Il intercepte une erreur dans le flux et vous donne une chance de la gérer.

**La règle d'or :** `catchError` doit retourner un **nouvel `Observable`**. Vous pouvez retourner :

* Un `Observable` avec une valeur par défaut (ex: `of([])` pour retourner un tableau vide).
* Un autre appel API (ex: réessayer avec un autre serveur).
* L'erreur elle-même, pour la propager après l'avoir traitée (ex: `throwError(() => error)`).

```typescript
import {HttpClient} from '@angular/common/http';
import {inject} from '@angular/core';
import {of, throwError} from 'rxjs';
import {catchError} from 'rxjs/operators';

const http = inject(HttpClient);

http.get('/api/users').pipe(
    catchError(error => {
        // Log l'erreur pour les développeurs
        console.error('Erreur lors de la récupération des utilisateurs', error);

        // Pour l'utilisateur, on retourne un tableau vide pour ne pas casser l'UI
        return of([]);
    })
).subscribe(users => console.log(users)); // L'UI reçoit [] et continue de fonctionner
```

@startuml
!theme vibrant
skinparam linetype ortho

participant "Source (http.get)" as Source
participant "catchError" as Catcher
participant "Abonné (subscribe)" as Sub

Source -> Catcher : Erreur!
activate Catcher

alt Erreur gérée
Catcher -> Sub : next([])
Catcher -> Sub : complete()
else Erreur propagée
Catcher -> Sub : error()
end
deactivate Catcher
@enduml
</tab>
<tab title="retry(n) : La Seconde Chance">
Parfois, une erreur est temporaire (un soubresaut du réseau). `retry` permet de se ré-abonner automatiquement au flux source qui a échoué, un certain nombre de fois.

C'est un outil simple mais puissant pour améliorer la résilience de votre application face à des problèmes de
connectivité intermittents.

<warning>
**Attention :** Utilisez `retry` avec sagesse. Retenter une requête qui a échoué avec un code 404 (Not Found) ou 403 (Forbidden) est inutile, car l'échec est permanent. C'est idéal pour les erreurs 5xx (Server Error) ou les erreurs réseau.
</warning>

```typescript
import {HttpClient} from '@angular/common/http';
import {inject} from '@angular/core';
import {of} from 'rxjs';
import {catchError, retry} from 'rxjs/operators';

const http = inject(HttpClient);

http.get('/api/flaky-endpoint').pipe(
    // Tente la requête jusqu'à 3 fois (1 initiale + 2 retries)
    retry(2),
    catchError(error => {
        console.error('La requête a échoué après 3 tentatives.');
        // Si après toutes les tentatives ça ne marche pas,
        // on retourne une valeur par défaut.
        return of({data: 'indisponible'});
    })
).subscribe();
```

</tab>
<tab title="finalize : Le Nettoyage Final">
L'opérateur `finalize` est le bloc `finally` de votre `Observable`. Il exécute une fonction callback quand le flux se termine, que ce soit par un `complete` (succès) ou un `error` (échec).

C'est l'endroit **parfait** pour faire du nettoyage, et notamment pour cacher un indicateur de chargement (`loader`).

```typescript
import {tap, finalize} from 'rxjs/operators';

this.isLoading = true; // Affiche le loader AVANT de s'abonner

myHttpCall$.pipe(
    tap(data => console.log('Données reçues:', data)),

    // Ce bloc sera TOUJOURS exécuté
    finalize(() => {
        this.isLoading = false; // Cache le loader, que ça réussisse ou non
        console.log('Flux terminé.');
    })
).subscribe({
    next: data => { /* Fait quelque chose avec les données */
    },
    error: err => { /* Gère l'erreur si nécessaire */
    }
});
```

</tab>
</tabs>

### La Combinaison de Flux : Créer des Vues Riches

Rarement une page complexe ne dépend que d'une seule source de données. Un dashboard peut avoir besoin des infos
utilisateur, de la liste des ventes et des notifications récentes. Comment charger tout cela efficacement ?

<tabs>
<tab title="forkJoin : Le Peloton d'Exécution">
`forkJoin` est l'équivalent de `Promise.all` pour les `Observables`. Il prend un tableau (ou un dictionnaire) d'Observables et attend que **tous** se terminent (`complete`). Une fois que c'est le cas, il émet une **seule valeur** : un tableau (ou un dictionnaire) contenant la **dernière valeur** de chaque `Observable` source.

Idéal pour les appels de "setup" : charger toutes les données initiales d'une page en parallèle.

<warning>
**Attention :** Si l'un des `Observables` dans `forkJoin` émet une erreur, l'ensemble du `forkJoin` échoue immédiatement et vous ne recevrez les résultats d'aucun des autres. Vous devez gérer les erreurs sur les flux internes si vous voulez éviter ce comportement.
</warning>

```typescript
import {forkJoin} from 'rxjs';
import {map} from 'rxjs/operators';

const user$ = this.http.get('/api/user/1');
const products$ = this.http.get('/api/products');
const settings$ = this.http.get('/api/settings');

forkJoin({
    user: user$,
    products: products$,
    settings: settings$
}).pipe(
    map(({user, products, settings}) => {
        // Travailler avec les trois résultats en même temps
        return {...user, preferredTheme: settings.theme, topProduct: products[0]};
    })
).subscribe(viewModel => console.log(viewModel));
```

@startuml
!theme vibrant

participant "user$" as U
participant "products$" as P
participant "forkJoin" as FJ
participant "Abonné" as Sub

@0
U is "..."
P is "..."

@100
U is {User Object}
U is {end}

@150
P is {[Product Array]}
P is {end}

@150
FJ -> Sub : next([ {User Object}, {[Product Array]} ])
FJ -> Sub : complete()
@enduml
</tab>
<tab title="combineLatest : Le Panneau de Contrôle">
`combineLatest` est un autre outil de combinaison, mais avec un comportement très différent et très réactif.
Il attend que chaque `Observable` source ait émis au moins une fois. Ensuite, il émet une nouvelle valeur **chaque fois que l'un des `Observables` source émet**. La valeur émise est un tableau contenant la **dernière valeur de chaque source**.

C'est l'opérateur parfait pour les scénarios où la vue dépend de plusieurs contrôles qui peuvent changer
indépendamment (filtres multiples, formulaires réactifs, etc.).

```typescript
import {combineLatest, fromEvent} from 'rxjs';
import {map, startWith} from 'rxjs/operators';

const filterBy$ = categoryFilter.valueChanges.pipe(startWith('all'));
const sortBy$ = sortOrder.valueChanges.pipe(startWith('date'));

combineLatest([
    filterBy$,
    sortBy$
]).pipe(
    map(([category, sortOrder]) => {
        // Dès que la catégorie OU l'ordre de tri change, 
        // on a les deux dernières valeurs pour relancer la recherche.
        return {category, sortOrder};
    })
).subscribe(filters => this.reloadData(filters));
```

</tab>
</tabs>

### Exercice 1.2 : Le Dashboard Multi-Sources

**Votre mission, si vous l'acceptez :** construire un dashboard qui charge trois types de données en parallèle depuis un
service. Le dashboard doit afficher un loader global pendant le chargement et gérer gracieusement les erreurs si l'une
des sources ne répond pas.

**Instructions :**

1. **Créez un `DashboardService`** avec 3 méthodes :
    * `getUserProfile()`: retourne un `Observable` d'un objet utilisateur après 1s.
    * `getSalesStats()`: retourne un `Observable` de statistiques après 1.5s.
    * `getSystemNotifications()`: **cette méthode doit échouer**. Elle retourne un `Observable` qui émet une erreur
      après 0.5s (`throwError`).
2. **Créez un `DashboardComponent`**.
3. Dans le composant, déclarez une propriété `isLoading = false;` et une propriété pour stocker les données du
   dashboard.
4. Dans `ngOnInit`, mettez `isLoading` à `true`.
5. Utilisez `forkJoin` pour appeler les 3 méthodes du service.
6. **Le point crucial :** Chaque `Observable` passé à `forkJoin` doit avoir sa propre gestion d'erreur (`catchError`)
   pour retourner une valeur par défaut (ex: `null` ou un objet d'erreur) afin d'empêcher le `forkJoin` global
   d'échouer.
7. Dans le `subscribe` de `forkJoin`, stockez les données reçues.
8. Utilisez `finalize` sur le `forkJoin` pour vous assurer que `isLoading` passe à `false` à la fin, que tout ait réussi
   ou non.
9. Affichez les données (ou un message d'erreur) et le loader dans le template.

#### Correction exercice 1.2 {collapsible='true'}

<procedure>
<p>Voici une solution robuste pour ce dashboard.</p>

**1. Le `DashboardService`**

```typescript
// src/app/dashboard.service.ts
import {Injectable} from '@angular/core';
import {Observable, of, throwError} from 'rxjs';
import {delay} from 'rxjs/operators';

@Injectable({
    providedIn: 'root'
})
export class DashboardService {

    getUserProfile(): Observable<{ name: string; email: string }> {
        return of({
            name: 'Alice',
            email: 'alice@example.com'
        }).pipe(delay(1000));
    }

    getSalesStats(): Observable<{ revenue: number; newCustomers: number }> {
        return of({
            revenue: 50000,
            newCustomers: 120
        }).pipe(delay(1500));
    }

    getSystemNotifications(): Observable<{ count: number; messages: string[] }> {
        // Simule une erreur de chargement des notifications
        return throwError(() => new Error('Notification service is down'))
            .pipe(delay(500));
    }
}
```

**2. Le `DashboardComponent`**

```typescript
// src/app/dashboard/dashboard.component.ts
import {Component, OnInit, inject} from '@angular/core';
import {CommonModule} from '@angular/common';
import {forkJoin, of} from 'rxjs';
import {catchError, finalize} from 'rxjs/operators';
import {DashboardService} from '../dashboard.service';

@Component({
    selector: 'app-dashboard',
    standalone: true,
    imports: [CommonModule],
    template: `
    <h2>Dashboard</h2>
    <div *ngIf="isLoading; else content">Chargement des données...</div>
    
    <ng-template #content>
      <div class="card">
        <h3>Profil Utilisateur</h3>
        <p *ngIf="dashboardData.userProfile">
          {{ dashboardData.userProfile.name }}
        </p>
        <p class="error" *ngIf="!dashboardData.userProfile">
          Données indisponibles.
        </p>
      </div>
      
      <div class="card">
        <h3>Statistiques de Vente</h3>
        <p *ngIf="dashboardData.salesStats">
          Revenu: {{ dashboardData.salesStats.revenue | currency }}
        </p>
        <p class="error" *ngIf="!dashboardData.salesStats">
          Données indisponibles.
        </p>
      </div>

      <div class="card">
        <h3>Notifications</h3>
        <p *ngIf="dashboardData.notifications">
          {{ dashboardData.notifications.count }} messages
        </p>
        <p class="error" *ngIf="!dashboardData.notifications">
          Impossible de charger les notifications.
        </p>
      </div>
    </ng-template>
  `,
    styles: [`.card { border: 1px solid #ccc; padding: 1rem; margin-bottom: 1rem; }
  .error { color: red; }`]
})
export class DashboardComponent implements OnInit {
    private dashboardService = inject(DashboardService);

    isLoading = false;
    dashboardData: any = {};

    ngOnInit(): void {
        this.loadDashboardData();
    }

    loadDashboardData(): void {
        this.isLoading = true;

        const userProfile$ = this.dashboardService.getUserProfile().pipe(
            catchError(err => of(null)) // En cas d'erreur, retourne null
        );

        const salesStats$ = this.dashboardService.getSalesStats().pipe(
            catchError(err => of(null))
        );

        const notifications$ = this.dashboardService.getSystemNotifications().pipe(
            catchError(err => {
                console.error(err); // On peut logger l'erreur
                return of(null); // Et continuer avec une valeur par défaut
            })
        );

        forkJoin({
            userProfile: userProfile$,
            salesStats: salesStats$,
            notifications: notifications$
        }).pipe(
            finalize(() => this.isLoading = false) // Sera toujours exécuté
        ).subscribe(data => {
            this.dashboardData = data;
        });
    }
}
```

</procedure>

### Auto-évaluation

1. **Dans un opérateur `catchError`, que devez-vous absolument retourner pour que le flux continue sans propager
   l'erreur à l'abonné ?**
   a. `null` ou `undefined`.
   b. Une chaîne de caractères décrivant l'erreur.
   c. Un nouvel `Observable` (par exemple, créé avec `of()` ou `throwError()`).
   d. Rien, le simple fait d'utiliser `catchError` suffit.

2. **Décrivez un scénario d'application concret où vous choisiriez `combineLatest` plutôt que `forkJoin`, et expliquez
   pourquoi.**

3. **Vous avez une propriété `isSaving` dans votre composant, que vous mettez à `true` avant d'appeler un service pour
   sauvegarder un formulaire. Quel opérateur RxJS est le plus approprié pour garantir que `isSaving` repasse à `false`
   que la sauvegarde réussisse ou échoue ?**

4. **Si un des `Observable`s fournis à `forkJoin` ne se termine jamais (ne `complete` jamais), que se passe-t-il ?**
   a. `forkJoin` émet après un certain timeout.
   b. `forkJoin` émet les valeurs de tous les `Observable`s qui se sont terminés.
   c. `forkJoin` n'émettra jamais de valeur.
   d. `forkJoin` émet une erreur.

5. **Quel est le principal danger si vous oubliez de mettre un `catchError` sur les `Observable`s internes que vous
   passez à `forkJoin` ?**

### Conclusion

Félicitations ! Vous avez ajouté à votre boîte à outils RxJS des techniques avancées qui sont le pain quotidien des
développeurs Angular expérimentés. Vous savez maintenant comment construire des chaînes d'opérateurs qui ne craignent
pas les erreurs grâce à `catchError` et `retry`, et comment orchestrer des opérations de nettoyage avec `finalize`. Plus
important encore, vous savez comment composer des flux multiples avec `forkJoin` et `combineLatest` pour créer des
expériences utilisateur riches et réactives.

La prochaine grande question qui se pose est : maintenant que nous savons comment récupérer et manipuler toutes ces
données, où les stockons-nous ? Comment partageons-nous cet état à travers différents composants de manière propre et
efficace ? C'est le défi de la **gestion d'état**, et ce sera le sujet de notre prochain module passionnant.