# Module 2 : L'essentiel - Stratégies de Gestion d'État (State Management)

### Objectifs pédagogiques

À la fin de ce module, vous serez en mesure de :

* **Identifier** les problèmes de communication entre composants qui surviennent dans une application en croissance ("
  prop drilling" et état incohérent).
* **Comprendre** le besoin d'une source de vérité unique pour les données de votre application.
* **Concevoir** et **implémenter** un service "stateful" en utilisant RxJS (`BehaviorSubject`) comme première solution
  de gestion d'état.
* **Distinguer** les avantages et les limites de cette approche "fait-main".

### Introduction

Votre application grandit. Au début, tout était simple : un composant parent parlait à son enfant direct. Mais
maintenant, les familles de composants sont devenues complexes. Le composant "Panier" dans le header doit connaître le
nombre d'articles, mais c'est le composant "DétailProduit", enfoui trois niveaux plus bas, qui contient le bouton "
Ajouter au panier". Comment faire communiquer ces deux composants éloignés sans créer un plat de spaghettis de
dépendances ?

C'est le problème de la **gestion de l'état** (State Management). L'état, c'est l'ensemble des données qui définissent
votre application à un instant T : qui est l'utilisateur connecté ? quel est le contenu du panier ? quelle est la liste
de produits affichée ?

Dans ce module, nous allons explorer pourquoi les méthodes simples de communication ne suffisent plus lorsque
l'application devient complexe. Nous allons ensuite construire notre première solution, élégante et pragmatique, pour
centraliser cet état : le service "stateful". C'est la première étape fondamentale pour construire des applications
fiables et maintenables.

### Le Problème : "Prop Drilling" et État Incohérent

Quand on débute, on utilise deux mécanismes principaux pour faire communiquer les composants :

1. `@Input()` pour passer des données d'un parent à un enfant.
2. `@Output()` et les événements pour remonter une information d'un enfant à un parent.

Mais que se passe-t-il quand le parent et l'enfant ne sont pas directs ?

#### Le "Prop Drilling" (Forage de propriétés)

Imaginez cette hiérarchie de composants : `App > PageProduits > ListeProduits > CarteProduit`.
La `PageProduits` détient l'information sur la devise à utiliser (ex: 'EUR'). La `CarteProduit`, tout en bas, a besoin
de cette information pour afficher le prix.

Pour y arriver, vous allez devoir :

1. Passer la devise de `PageProduits` à `ListeProduits` via un `@Input()`.
2. Passer la devise de `ListeProduits` à `CarteProduit` via un autre `@Input()`.

Le composant `ListeProduits` n'a rien à faire de cette devise ! Il ne fait que la transporter, comme un facteur. C'est
ça, le "prop drilling". C'est fastidieux, et si vous devez ajouter un composant intermédiaire, vous devez penser à
percer un nouveau "trou" pour la propriété.

@startuml
!theme vibrant
skinparam componentStyle rectangle

component "PageProduits \n (détient `devise`)" as Page
component "ListeProduits \n (transporte `devise`)" as List
component "CarteProduit \n (utilise `devise`)" as Card

Page -down-> List : `@Input() devise`
List -down-> Card : `@Input() devise`

note right of List : Je ne me sers pas de `devise`,\nje ne fais que la passer.
@enduml

#### L'État Incohérent

Maintenant, imaginez que deux composants différents et non liés (ex: un `HeaderComponent` et un `ProfileComponent`)
peuvent tous les deux modifier le nom de l'utilisateur. S'ils le font via un service simple qui n'a pas de "mémoire" de
l'état, comment s'assurer que lorsque l'un met à jour le nom, l'autre est immédiatement notifié du changement ?

Sans un point central qui est la **source de vérité unique**, vous risquez de vous retrouver avec deux parties de votre
UI qui affichent des informations contradictoires. C'est l'état incohérent.

### Solution 1 : Le Service "Stateful" avec RxJS (L'approche pragmatique)

La solution à ces deux problèmes est de **centraliser l'état**. Au lieu que l'état soit éparpillé dans les composants,
nous allons le confier à un gardien unique : un service spécial, dit "stateful" (qui a un état).

Le patron de conception est simple et très puissant. Imaginez un service comme un coffre-fort :

1. **L'état est privé :** Le trésor (les données) est à l'intérieur, personne ne peut y toucher directement. On utilise
   pour cela un `BehaviorSubject` **privé**.
2. **On peut consulter l'état :** Le service expose une version "lecture seule" de l'état via un `Observable` public.
   Les composants peuvent s'abonner pour voir l'état, mais pas le modifier. On utilise pour cela la méthode
   `.asObservable()`.
3. **On demande une modification :** Le service expose des **méthodes publiques** (`updateUser`, `loadProducts`...).
   C'est la seule porte d'entrée pour demander un changement. Ces méthodes sont les seules à pouvoir modifier le
   `BehaviorSubject` privé.

**Voici à quoi cela ressemble en pratique :**

```typescript
// user-state.service.ts
import {Injectable} from '@angular/core';
import {BehaviorSubject, Observable} from 'rxjs';

export interface User {
    id: number;
    name: string;
    email: string;
}

// L'état complet géré par ce service
interface UserState {
    currentUser: User | null;
    isLoading: boolean;
    error: string | null;
}

@Injectable({
    providedIn: 'root'
})
export class UserStateService {
    // 1. L'état est stocké dans un BehaviorSubject privé.
    // Il est initialisé avec un état de départ.
    private state$ = new BehaviorSubject<UserState>({
        currentUser: null,
        isLoading: false,
        error: null
    });

    // 2. On expose l'état en lecture seule via un Observable public.
    public currentUser$ = this.state$.asObservable();

    // 3. Méthodes publiques pour modifier l'état.
    public loadUser(id: number): void {
        // On met à jour l'état pour indiquer le chargement
        this.updateState({isLoading: true, error: null});

        // Simule un appel API
        // this.httpClient.get<User>(`/api/users/${id}`).subscribe({ ... })

        // Pour l'exemple, on simule une réponse réussie
        setTimeout(() => {
            const user: User = {id, name: 'Jane Doe', email: 'jane@doe.com'};
            this.updateState({currentUser: user, isLoading: false});
        }, 1000);
    }

    public clearUser(): void {
        this.updateState({currentUser: null});
    }

    // Méthode utilitaire privée pour mettre à jour l'état de manière sûre
    private updateState(partialState: Partial<UserState>): void {
        this.state$.next({
            ...this.state$.getValue(), // On récupère l'état actuel
            ...partialState           // On fusionne avec les nouvelles valeurs
        });
    }
}
```

Avec cette approche, n'importe quel composant de l'application peut injecter `UserStateService` et s'abonner à
`currentUser$` pour réagir aux changements d'état, résolvant à la fois le "prop drilling" et le risque d'incohérence.

@startuml
!theme vibrant

skinparam componentStyle rectangle
skinparam linetype ortho

package "Application" {
component [HeaderComponent] as Header
component [ProfileComponent] as Profile
component [LoginFormComponent] as Login

database "UserStateService" as Service {
rectangle "state$ (BehaviorSubject)" as BS
}
}

Header --up-|> Service : "s'abonne à currentUser$"
Profile --up-|> Service : "s'abonne à currentUser$"
Login --up-|> Service : "appelle loadUser()"

Service -> BS : ".next(newState)"
BS ..> Header : notifie
BS ..> Profile : notifie

@enduml

### Exercice 2.1 : Créer un Store "fait-main" pour une liste de tâches

**Objectif :** Appliquer le patron du service "stateful" pour gérer une liste de tâches (To-Do list).

**Instructions :**

1. **Créez une interface `Task`** dans un fichier `task.model.ts` avec les champs `id` (number), `title` (string), et
   `completed` (boolean).
2. **Créez une interface `TasksState`** qui contiendra un tableau `tasks: Task[]`.
3. **Créez un `TaskStateService`** :
    * Il doit contenir un `BehaviorSubject<TasksState>` privé, initialisé avec une liste de tâches vide.
    * Il doit exposer un `Observable` public `tasks$` qui sélectionne le tableau de tâches de l'état. (Astuce:
      `this.state$.pipe(map(state => state.tasks))`).
    * Il doit avoir une méthode publique `addTask(title: string)` qui crée une nouvelle tâche, l'ajoute à la liste
      actuelle et met à jour l'état.
    * Il doit avoir une méthode publique `toggleTaskCompleted(taskId: number)` qui trouve la tâche correspondante,
      inverse son statut `completed`, et met à jour l'état.
4. Vous pouvez utiliser le composant ci-dessous pour tester votre service.

**Composant de test (à placer dans votre `app.component.ts` par exemple) :**

```typescript
import {Component, inject} from '@angular/core';
import {CommonModule} from '@angular/common';
import {TaskStateService} from './task-state.service'; // Adaptez le chemin

@Component({
    selector: 'app-root',
    standalone: true,
    imports: [CommonModule],
    template: `
    <h1>Ma liste de tâches</h1>
    <button (click)="addTask('Apprendre la gestion d-état')">
      Ajouter Tâche
    </button>
    <ul>
      <li *ngFor="let task of tasks$ | async" 
          (click)="toggleTask(task.id)"
          [style.textDecoration]="task.completed ? 'line-through' : 'none'">
        {{ task.title }}
      </li>
    </ul>
  `
})
export class AppComponent {
    private taskStateService = inject(TaskStateService);

    tasks$ = this.taskStateService.tasks$;

    addTask(title: string): void {
        this.taskStateService.addTask(title);
    }

    toggleTask(id: number): void {
        this.taskStateService.toggleTaskCompleted(id);
    }
}
```

#### Correction exercice 2.1 {collapsible='true'}

<procedure>
<p>Voici l'implémentation du modèle et du service stateful pour la liste de tâches.</p>

**1. Le modèle de données `task.model.ts`**

```typescript
// src/app/task.model.ts
export interface Task {
    id: number;
    title: string;
    completed: boolean;
}

export interface TasksState {
    tasks: Task[];
}
```

**2. Le service d'état `task-state.service.ts`**

```typescript
// src/app/task-state.service.ts
import {Injectable} from '@angular/core';
import {BehaviorSubject, Observable} from 'rxjs';
import {map} from 'rxjs/operators';
import {Task, TasksState} from './task.model';

@Injectable({
    providedIn: 'root'
})
export class TaskStateService {
    // État initial
    private initialState: TasksState = {
        tasks: [
            {id: 1, title: 'Terminer le module RxJS', completed: true}
        ]
    };

    // Le BehaviorSubject privé qui détient l'état
    private state$ = new BehaviorSubject<TasksState>(this.initialState);

    // L'Observable public pour consommer la liste des tâches
    public tasks$: Observable<Task[]> = this.state$.pipe(
        map(state => state.tasks)
    );

    // Méthode publique pour ajouter une tâche
    public addTask(title: string): void {
        const currentState = this.state$.getValue();
        const newTask: Task = {
            id: currentState.tasks.length + 1, // Logique d'ID simpliste
            title,
            completed: false
        };

        // Création du nouvel état
        const newState: TasksState = {
            tasks: [...currentState.tasks, newTask]
        };

        // On pousse le nouvel état dans le BehaviorSubject
        this.state$.next(newState);
    }

    // Méthode publique pour basculer le statut d'une tâche
    public toggleTaskCompleted(taskId: number): void {
        const currentState = this.state$.getValue();

        const updatedTasks = currentState.tasks.map(task =>
            task.id === taskId ? {...task, completed: !task.completed} : task
        );

        const newState: TasksState = {
            tasks: updatedTasks
        };

        this.state$.next(newState);
    }
}
```

</procedure>

### Auto-évaluation

1. **Qu'est-ce que le "prop drilling" ?**
   a. Une technique pour optimiser le rendu des composants.
   b. Le fait de devoir passer des données à travers plusieurs niveaux de composants intermédiaires qui n'en ont pas
   besoin.
   c. Une erreur qui se produit lors de l'injection de dépendances.
   d. Le processus d'extraction des données d'un formulaire.

2. **Dans le patron du service "stateful", pourquoi expose-t-on l'état via `.asObservable()` plutôt que de donner accès
   directement au `BehaviorSubject` ?**

3. **Quel est l'avantage principal d'utiliser un `BehaviorSubject` plutôt qu'un `Subject` simple pour gérer un état ?**
   a. Il est plus performant.
   b. Il fournit la dernière valeur de l'état à tout nouvel abonné.
   c. Il peut gérer plusieurs types de données.
   d. Il gère automatiquement les erreurs.

4. **Comment un composant doit-il demander une modification de l'état lorsqu'on utilise un service "stateful" ?**
   a. En modifiant directement la propriété publique du service.
   b. En émettant un événement global.
   c. En appelant une méthode publique exposée par le service.
   d. En utilisant la fonction `inject()` avec un token spécial.

5. **Citez un inconvénient potentiel de cette approche de "store fait-main" pour une application très grande et
   complexe.**

### Conclusion

Vous venez de mettre en place votre premier système de gestion d'état ! Le patron du service "stateful" avec RxJS est
une solution **extrêmement efficace et pragmatique** pour une grande majorité d'applications Angular. Il résout les
problèmes de "prop drilling" et d'état incohérent en créant une source de vérité unique, centralisée et réactive.

Cependant, à mesure que l'application et l'équipe grandissent, cette approche peut montrer ses limites : la logique de
modification est disséminée dans les méthodes, il n'y a pas d'outils de débogage pour voir l'historique des changements
d'état, et il n'y a pas de cadre strict pour gérer les effets secondaires (comme les appels API).

Pour répondre à ces défis, des bibliothèques plus structurées ont été créées. Dans la partie "Pour aller plus loin",
nous allons découvrir la plus célèbre d'entre elles : **NgRx**, l'implémentation du patron Redux pour Angular.
Préparez-vous à entrer dans un monde encore plus structuré et puissant 