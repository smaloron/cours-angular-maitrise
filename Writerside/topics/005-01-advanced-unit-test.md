# Module 5 : Pour aller plus loin - Mocking, Spies et Tests Asynchrones

### Objectifs pédagogiques

À la fin de cette partie, vous serez capable de :

* **Créer** des "doublures" (mocks) de vos services en utilisant `jasmine.createSpyObj`.
* **Espionner** (`spyOn`) des méthodes existantes pour vérifier si elles ont été appelées, et avec quels arguments.
* **Simuler** des interactions utilisateur comme des clics sur des boutons.
* **Maîtriser** les différentes stratégies pour tester du code asynchrone (RxJS) avec `async`, `fakeAsync`, et `tick`.

### Introduction

Dans la partie "L'essentiel", nous avons appris à tester des unités de code relativement isolées. Nous avons testé un
service qui dépendait de `HttpClient`, une dépendance "externe" pour laquelle Angular nous fournit un module de test
tout prêt.

Mais dans une vraie application, nos composants dépendent de **nos propres services**. Un `LoginComponent` dépend d'un
`AuthService`, qui lui-même dépend peut-être d'un `NotificationService`. Comment tester le `LoginComponent` **sans
dépendre du vrai `AuthService`** ? Si le test du composant échoue, est-ce à cause du composant ou à cause d'un bug dans
le service ?

Pour des tests unitaires purs, nous devons **isoler l'unité testée**. C'est là qu'interviennent les techniques de *
*mocking** et les **espions (spies)**. Nous allons remplacer les dépendances réelles par des doublures que nous
contrôlons entièrement. Nous deviendrons des maîtres de l'illusion, capables de dicter le comportement des dépendances
pour tester chaque scénario de notre composant. Enfin, nous nous attaquerons au défi de l'asynchronisme, en apprenant à
contrôler le temps pour tester nos `Observables` de manière fiable et rapide.

### Le Mocking et les Spies

Un "mock" est un faux objet qui simule le comportement d'un objet réel. Un "spy" est une fonction qui enrobe une
fonction réelle pour observer comment elle est utilisée. Jasmine nous fournit d'excellents outils pour cela.

<tabs>
<tab title="jasmine.createSpyObj : Le Faussaire">
`createSpyObj` est un moyen rapide de créer un objet factice qui possède certaines méthodes (les "spies"). C'est idéal pour mocker un service dans un test de composant.

Imaginez que notre `LoginComponent` dépend de `AuthService`.

```typescript
// login.component.ts
@Component({...})
export class LoginComponent {
    constructor(private authService: AuthService) {
    }

    login() {
        this.authService.login({user: 'test', pass: 'pwd'});
    }
}
```

Dans notre test, nous ne voulons pas utiliser le vrai `AuthService`.

```typescript
// login.component.spec.ts
describe('LoginComponent', () => {
    let component: LoginComponent;
    let fixture: ComponentFixture<LoginComponent>;
    // On déclare une variable pour notre mock
    let authServiceSpy: jasmine.SpyObj<AuthService>;

    beforeEach(async () => {
        // On crée le mock avec les méthodes que l'on veut espionner
        authServiceSpy = jasmine.createSpyObj('AuthService', ['login']);

        await TestBed.configureTestingModule({
            imports: [LoginComponent],
            providers: [
                // On dit à Angular : quand quelqu'un demande AuthService,
                // fournis mon mock à la place !
                {provide: AuthService, useValue: authServiceSpy}
            ]
        }).compileComponents();

        fixture = TestBed.createComponent(LoginComponent);
        component = fixture.componentInstance;
    });

    it('devrait appeler authService.login() lors du clic', () => {
        // On simule un clic sur un bouton (plus de détails ci-dessous)
        // ...
        component.login(); // Appelons la méthode directement pour l'instant

        // On vérifie que la méthode 'login' sur notre mock a bien été appelée.
        expect(authServiceSpy.login).toHaveBeenCalled();
        // On peut même vérifier avec quels arguments !
        expect(authServiceSpy.login).toHaveBeenCalledWith({user: 'test', pass: 'pwd'});
    });
});
```

</tab>
<tab title="spyOn : L'Espion Discret">
Parfois, on ne veut pas remplacer un objet entier, mais juste "espionner" une méthode sur un objet réel, pour voir si elle est appelée, tout en laissant l'appel original se produire. C'est le rôle de `spyOn`.

```typescript
it('devrait appeler console.log', () => {
    // On crée un espion sur la méthode `log` de l'objet `console`.
    const logSpy = spyOn(console, 'log');

    myFunctionThatLogs(); // Une fonction qui contient un console.log

    expect(logSpy).toHaveBeenCalled();
});
```

On peut aussi contrôler la valeur de retour d'un espion :

```typescript
// login.component.spec.ts
//...
it('devrait afficher un message si le login réussit', () => {
    // On dit à notre spy : "Quand on t'appellera,
    // retourne un Observable qui émet `true`"
    authServiceSpy.login.and.returnValue(of(true));

    //... simuler l'action et vérifier l'affichage du message ...
});
```

</tab>
</tabs>

### Simuler les Interactions Utilisateur

Pour tester un composant de manière réaliste, nous devons simuler les actions d'un utilisateur.

1. **Trouver l'élément :** On utilise `fixture.nativeElement.querySelector(...)` pour récupérer un élément du DOM.
2. **Déclencher l'événement :** Pour un clic, on peut simplement appeler la méthode `.click()` sur l'élément. Pour des
   événements plus complexes, on peut utiliser `dispatchEvent(new Event('...'))`.

```typescript
it('devrait appeler la méthode login() quand on clique sur le bouton', () => {
    // On espionne la méthode login() sur l'instance réelle du composant
    spyOn(component, 'login');

    // On trouve le bouton dans le DOM
    const loginButton = fixture.nativeElement.querySelector('button#login-btn');

    // On simule le clic
    loginButton.click();

    // On vérifie que la méthode du composant a été appelée
    expect(component.login).toHaveBeenCalled();
});
```

### Tester le Code Asynchrone (RxJS)

C'est souvent le point qui pose le plus de problèmes. Comment tester un `Observable` qui émet une valeur après un
délai ? On ne veut pas que nos tests durent des secondes. Jasmine propose plusieurs stratégies.

<tabs>
<tab title="Avec `async` et `whenStable`">
`async` est un utilitaire de `TestBed` qui crée une zone de test spéciale. À l'intérieur, on peut utiliser `fixture.whenStable()`. Cette méthode retourne une `Promise` qui se résout une fois que toutes les tâches asynchrones (comme les `Promises`, `setTimeout`) sont terminées.

```typescript
it('devrait afficher les données après un appel asynchrone', async(() => {
    // On configure le mock pour qu'il retourne un Observable asynchrone
    userServiceSpy.getUsers.and.returnValue(of(mockUsers).pipe(delay(10)));

    // On déclenche la détection de changement initiale
    fixture.detectChanges();

    // On attend que toutes les tâches asynchrones soient terminées
    fixture.whenStable().then(() => {
        // Une fois stable, on redéclenche la détection de changement
        // pour afficher les données reçues.
        fixture.detectChanges();

        const userList = fixture.nativeElement.querySelector('ul');
        expect(userList.children.length).toBe(mockUsers.length);
    });
}));
```

Cette approche est réaliste, mais peut être un peu verbeuse.
</tab>
<tab title="Avec `fakeAsync` et `tick`">
`fakeAsync` est encore plus puissant. Il crée une zone de test où nous devenons les **maîtres du temps**. Toutes les fonctions asynchrones (comme `setTimeout`, `setInterval`, `delay`) sont patchées.

À l'intérieur de cette zone, nous avons accès à la fonction `tick(milliseconds)`. Appeler `tick(1000)` simule
l'avancement du temps de 1000 millisecondes, **instantanément**.

```typescript
it('devrait afficher les données en contrôlant le temps', fakeAsync(() => {
    userServiceSpy.getUsers.and.returnValue(of(mockUsers).pipe(delay(500)));

    fixture.detectChanges(); // Déclenche l'appel à getUsers

    // À ce stade, le DOM est vide, le délai de 500ms n'est pas passé
    let list = fixture.nativeElement.querySelector('ul');
    expect(list.children.length).toBe(0);

    // On fait avancer l'horloge virtuelle de 500ms
    tick(500);

    // L'Observable a maintenant émis sa valeur.
    // Mettons à jour la vue.
    fixture.detectChanges();

    list = fixture.nativeElement.querySelector('ul');
    expect(list.children.length).toBe(mockUsers.length);
}));
```

`fakeAsync` est souvent la méthode préférée car elle rend les tests asynchrones synchrones, clairs et très rapides.
</tab>
</tabs>

### Exercice 5.2 : Tester un composant de recherche

**Objectif :** Tester le `SearchComponent` que nous avons créé dans le module RxJS. Nous allons mocker le
`SearchService` et tester la logique asynchrone avec `fakeAsync`.

**Rappel du composant :** Il a un champ de saisie. Quand l'utilisateur tape, il attend 300ms (`debounceTime`), puis
appelle `searchService.search()`.

**Instructions :**

1. Mettez en place le `TestBed` pour le `SearchComponent`.
2. Créez un mock du `SearchService` avec `jasmine.createSpyObj` et fournissez-le au `TestBed`.
3. Écrivez un test avec `fakeAsync` qui :
    * Simule la saisie d'un texte dans le champ de recherche.
    * Vérifie que `searchService.search` N'EST PAS appelé immédiatement.
    * Fait avancer le temps de 300ms avec `tick(300)`.
    * Vérifie que `searchService.search` A ÉTÉ appelé avec le bon terme.

#### Correction exercice 5.2 {collapsible='true'}

<procedure>
<p>Ce test combine le mocking, la simulation d'événement et le test asynchrone avec `fakeAsync`.</p>

```typescript
// src/app/search/search.component.spec.ts
import {ComponentFixture, TestBed, fakeAsync, tick}
    from '@angular/core/testing';
import {ReactiveFormsModule} from '@angular/forms';
import {of} from 'rxjs';

import {SearchComponent} from './search.component';
import {SearchService} from '../search.service';

describe('SearchComponent', () => {
    let component: SearchComponent;
    let fixture: ComponentFixture<SearchComponent>;
    let searchServiceSpy: jasmine.SpyObj<SearchService>;

    beforeEach(fakeAsync(() => {
        // Création du mock pour SearchService
        searchServiceSpy = jasmine.createSpyObj('SearchService', ['search']);

        TestBed.configureTestingModule({
            imports: [SearchComponent, ReactiveFormsModule],
            providers: [
                {provide: SearchService, useValue: searchServiceSpy}
            ]
        }).compileComponents();

        fixture = TestBed.createComponent(SearchComponent);
        component = fixture.componentInstance;
    }));

    it('devrait appeler le service après un debounceTime de 300ms', fakeAsync(() => {
        // Le spy retourne un Observable vide pour que le subscribe fonctionne
        searchServiceSpy.search.and.returnValue(of([]));

        fixture.detectChanges(); // Initialise ngOnInit et le valueChanges

        // On récupère l'input et on simule la saisie
        const inputElement = fixture.nativeElement.querySelector('input');
        inputElement.value = 'Angular';
        inputElement.dispatchEvent(new Event('input'));

        fixture.detectChanges();

        // IMMÉDIATEMENT après la saisie, le service ne doit pas avoir été appelé
        expect(searchServiceSpy.search).not.toHaveBeenCalled();

        // On avance l'horloge de 300ms pour passer le debounceTime
        tick(300);

        // MAINTENANT, le service doit avoir été appelé
        expect(searchServiceSpy.search).toHaveBeenCalledTimes(1);
        expect(searchServiceSpy.search).toHaveBeenCalledWith('Angular');
    }));
});
```

</procedure>

### Auto-évaluation

1. **Dans un test de composant, si vous voulez fournir une fausse version d'un service, comment le déclarez-vous dans
   le `TestBed` ?**
   a. `imports: [MyFakeService]`
   b. `declarations: [MyFakeService]`
   c. `providers: [{ provide: RealService, useValue: myFakeService }]`
   d. `bootstrap: [MyFakeService]`

2. **Quelle est la principale différence entre `jasmine.createSpyObj` et `spyOn` ?**

3. **Vous testez un `Observable` qui utilise l'opérateur `debounceTime(500)`. Quelle est la meilleure stratégie de test
   pour éviter d'attendre 500ms réelles ?**
   a. La zone `async` avec `fixture.whenStable()`.
   b. La zone `fakeAsync` avec un appel à `tick(500)`.
   c. Mettre un `setTimeout(..., 500)` dans le test.
   d. Utiliser un callback `done`.

4. **Dans un test utilisant `fakeAsync`, à quoi sert la fonction `tick()` ?**

5. **Vous avez un espion `mySpy`. Comment pouvez-vous vérifier qu'il a été appelé exactement une fois avec l'
   argument `'hello'` ?**
   a. `expect(mySpy).toHaveBeenCalledWith('hello', 1)`
   b. `expect(mySpy.calls.count()).toBe(1); expect(mySpy.calls.first().args).toContain('hello');`
   c. `expect(mySpy).toHaveBeenCalledTimes(1); expect(mySpy).toHaveBeenCalledWith('hello');`
   d. `expect(mySpy('hello')).toBe(1);`

### Conclusion

Vous avez maintenant débloqué les techniques de test les plus puissantes. Vous savez comment **isoler complètement**
l'unité que vous testez en **mockant** ses dépendances. Vous êtes capable de vérifier les interactions entre les
différentes parties de votre code en utilisant des **espions**. Et surtout, vous avez démystifié le test asynchrone en
apprenant à **maîtriser le temps** avec `fakeAsync` et `tick`.

L'écriture de tests n'est plus une boîte noire. C'est une compétence d'ingénierie qui vous apporte confiance et
robustesse. En pratiquant ces techniques, vous constaterez que vous concevez même votre code différemment, en le rendant
plus modulaire et plus testable dès le départ.

Nous avons couvert l'essentiel de l'outillage Angular moderne. Dans les prochains modules, nous nous intéresserons à des
sujets plus spécifiques mais tout aussi importants pour des applications professionnelles : comment intégrer des
librairies externes comme un Design System, comment créer des composants dynamiques, ou encore comment rendre nos
applications accessibles à tous.