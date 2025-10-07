# Corrections des Auto-évaluations

Voici les corrections et les explications pour toutes les auto-évaluations du cours. Utilisez-les pour valider vos
connaissances et, surtout, pour comprendre les raisonnements derrière chaque réponse.

---

## Module 0 : Introduction et Rappels Clés

### L'essentiel

1. **Quel est le principal avantage des composants Standalone par rapport aux `NgModule` ?**

* **Réponse : B.** Ils simplifient la gestion des dépendances et réduisent le code standard (boilerplate).
* **Explication :** L'avantage majeur des composants Standalone est architectural. En permettant à un composant de gérer
  ses propres dépendances (`imports`), ils éliminent la nécessité des `NgModules`, ce qui simplifie la structure du
  projet, réduit la quantité de code à écrire et à maintenir, et rend la logique de dépendance plus locale et plus
  claire.

2. **Expliquez avec vos propres mots le rôle du décorateur `@Injectable({ providedIn: 'root' })`.**

* **Réponse type :** Ce décorateur indique à Angular que la classe est un service qui peut être injecté dans d'autres
  classes. L'option `{ providedIn: 'root' }` est particulièrement importante : elle enregistre le service auprès de
  l'injecteur racine de l'application. Cela signifie qu'Angular créera une seule et unique instance de ce service (un
  singleton) qui sera partagée et disponible dans toute l'application, ce qui est idéal pour partager un état ou une
  logique métier.

3. **Opérateur RxJS pour une recherche auto-complétée.**

* **Réponse : C.** `switchMap`.
* **Explication :** `switchMap` est parfait pour ce cas car à chaque nouvelle émission de la source (une nouvelle frappe
  au clavier), il se désabonne de la requête HTTP précédente (si elle est toujours en cours) et s'abonne à la nouvelle.
  Cela évite des requêtes inutiles et garantit que seul le résultat de la dernière recherche sera traité.

4. **Pourquoi les Formulaires Réactifs sont-ils plus "scalables" ?**

* **Réponse type :** Les Formulaires Réactifs sont considérés comme plus "scalables" car le modèle de données du
  formulaire (`FormGroup`, `FormControl`) et la logique de validation sont définis et gérés de manière explicite dans la
  classe du composant (le fichier `.ts`). Cela sépare la logique de la vue, rend le code plus facile à tester
  unitairement, et permet de gérer des logiques de validation complexes et dynamiques beaucoup plus facilement que les
  Formulaires pilotés par le template, où la logique est plus dispersée dans le HTML.

5. **À quoi sert le `async` pipe ?**

* **Réponse : B.** Il s'abonne à un `Observable` ou une `Promise` et retourne la dernière valeur émise, gérant
  automatiquement la désinscription.
* **Explication :** Le pipe `async` est un outil de commodité et de sécurité. Il gère le cycle de vie de la souscription
  pour nous : il s'abonne quand le composant est créé et se désabonne automatiquement quand le composant est détruit,
  prévenant ainsi les fuites de mémoire.

### Pour aller plus loin

1. **Affirmation FAUSSE concernant `inject()` :**

* **Réponse : C.** Elle peut être appelée à n'importe quel moment dans le cycle de vie d'un composant, y compris dans
  une méthode déclenchée par un clic.
* **Explication :** C'est faux. La fonction `inject()` doit impérativement être appelée dans un "contexte d'injection" :
  lors de l'initialisation d'une propriété de classe, dans le constructeur, ou dans une fonction "factory" d'un
  provider. L'appeler plus tard (dans `ngOnInit` ou un event handler) lèvera une erreur.

2. **Mécanisme des URLs d'API différentes :**

* **Réponse type :** Ce mécanisme repose sur les fichiers d'environnement (`environment.ts` et `environment.prod.ts`).
  Le fichier de configuration `angular.json` contient une instruction `fileReplacements` qui, lors d'un build de
  production (`ng build`), remplace automatiquement le fichier de développement par celui de production, permettant
  ainsi à la même variable (`environment.apiUrl`) d'avoir des valeurs différentes selon le contexte.

3. **Principal bénéfice d'une `interface` TypeScript :**

* **Réponse : C.** Cela fournit une vérification de type à la compilation et améliore l'aide au développement (
  autocomplétion).
* **Explication :** Une `interface` est une construction purement TypeScript. Elle n'existe pas à l'exécution et n'a
  aucun impact sur les performances. Son rôle est d'aider le développeur et le compilateur en garantissant que la forme
  des objets est correcte, ce qui évite de nombreuses erreurs et améliore la productivité.

4. **Mot-clé TypeScript pour une propriété non modifiable :**

* **Réponse :** `readonly`.
* **Explication :** `readonly id: number;` empêchera toute réassignation de la propriété `id` après l'initialisation de
  l'objet.

5. **Fichier de configuration gérant le remplacement des fichiers d'environnement :**

* **Réponse : C.** `angular.json`.
* **Explication :** C'est le fichier de configuration de l'espace de travail Angular qui contient toutes les
  instructions pour le build, le serve, les tests, y compris la section `fileReplacements`.

---

## Module 1 : Maîtrise de RxJS

### L'essentiel

1. **Que reçoit un nouvel abonné à un `BehaviorSubject` ?**

* **Réponse : C.** La toute dernière valeur qui a été émise (ou la valeur initiale si aucune n'a été émise).
* **Explication :** C'est la caractéristique principale du `BehaviorSubject`. Il a une "mémoire" de la dernière valeur
  et la fournit immédiatement à tout nouvel abonné, ce qui le rend parfait pour représenter des états.

2. **Différence entre `switchMap` et `mergeMap` :**

* **Réponse type :** `switchMap` gère la concurrence en annulant l'observable interne précédent dès qu'un nouveau se
  présente (utile pour les recherches). `mergeMap` ne se soucie pas de la concurrence et exécute tous les observables
  internes en parallèle, émettant leurs valeurs dès qu'elles arrivent (utile pour des actions indépendantes comme des
  uploads de fichiers).

3. **Opérateur pour une séquence d'appels API :**

* **Réponse : D.** `concatMap`.
* **Explication :** `concatMap` est le seul des quatre qui garantit l'ordre d'exécution en attendant que l'observable
  interne précédent se termine (`complete`) avant de s'abonner au suivant.

4. **Rôle de l'objet retourné par `.subscribe()` :**

* **Réponse type :** L'objet retourné est une `Subscription`. Son rôle est de représenter l'abonnement en cours. Il est
  crucial car il contient la méthode `.unsubscribe()`, qui doit être appelée pour arrêter d'écouter et libérer la
  mémoire lorsque le composant est détruit, afin d'éviter les fuites de mémoire.

5. **Différence entre `Observable` et `Subject` :**

* **Réponse : B.** Un `Subject` est "chaud" et multicast (partage ses valeurs), tandis qu'un `Observable` standard est "
  froid" et unicast (chaque abonné a sa propre exécution).
* **Explication :** Un `Observable` est comme une recette de cuisine ; chaque personne qui la suit (s'abonne) fait son
  propre plat. Un `Subject` est comme un haut-parleur ; il diffuse le même message à tous ceux qui écoutent en même
  temps.

### Pour aller plus loin

1. **Que doit retourner un `catchError` ?**

* **Réponse : C.** Un nouvel `Observable` (par exemple, créé avec `of()` ou `throwError()`).
* **Explication :** Le contrat de `catchError` est de remplacer le flux qui a échoué par un nouveau flux. Si vous
  retournez une valeur simple, le flux s'arrêtera. En retournant un `Observable`, vous permettez à la chaîne de
  continuer.

2. **Scénario pour `combineLatest` vs `forkJoin` :**

* **Réponse type :** J'utiliserais `forkJoin` pour le chargement initial d'un dashboard : il faut attendre que les infos
  utilisateur ET les statistiques soient arrivées avant d'afficher la page. J'utiliserais `combineLatest` pour une page
  avec des filtres multiples (catégorie, prix, ordre de tri) : dès qu'un des filtres change, `combineLatest` émet les
  dernières valeurs de tous les filtres, me permettant de relancer la recherche avec les bons paramètres.

3. **Opérateur pour garantir que `isSaving` repasse à `false` :**

* **Réponse :** `finalize`.
* **Explication :** `finalize` exécute son callback que l'observable se termine par un succès (`complete`) ou un échec (
  `error`), ce qui en fait l'endroit parfait et garanti pour du code de nettoyage.

4. **Que se passe-t-il si un `Observable` dans `forkJoin` ne se termine jamais ?**

* **Réponse : C.** `forkJoin` n'émettra jamais de valeur.
* **Explication :** `forkJoin` attend que TOUS les observables fournis aient émis leur signal `complete`. Si l'un d'eux
  ne le fait jamais, `forkJoin` attendra indéfiniment.

5. **Danger si on oublie `catchError` sur les `Observable`s internes de `forkJoin` :**

* **Réponse type :** Le principal danger est que si un seul des observables internes émet une erreur, l'ensemble du
  `forkJoin` échoue immédiatement. Vous ne recevrez aucune des données des autres observables, même celles qui ont
  réussi. L'application de `catchError` sur chaque flux interne permet d'isoler les erreurs et de laisser `forkJoin` se
  terminer avec les résultats disponibles.

---

## Module 2 : Stratégies de Gestion d'État

### L'essentiel

1. **Qu'est-ce que le "prop drilling" ?**

* **Réponse : B.** Le fait de devoir passer des données à travers plusieurs niveaux de composants intermédiaires qui
  n'en ont pas besoin.
* **Explication :** C'est un problème de maintenance où un composant se retrouve avec des `@Input()` et `@Output()` qui
  ne lui servent qu'à faire transiter de l'information vers un enfant lointain, ce qui rend le code plus complexe et
  plus fragile.

2. **Pourquoi utiliser `.asObservable()` ?**

* **Réponse type :** On expose l'état via `.asObservable()` pour protéger l'état et respecter le principe d'
  encapsulation. `.asObservable()` retourne une version "lecture seule" du `BehaviorSubject`. Cela empêche les
  composants consommateurs de pousser de nouvelles valeurs directement dans le `Subject` (avec `.next()`), les forçant à
  passer par les méthodes publiques du service, ce qui garantit que l'état est modifié de manière contrôlée et
  prévisible.

3. **Avantage d'un `BehaviorSubject` sur un `Subject` pour gérer un état :**

* **Réponse : B.** Il fournit la dernière valeur de l'état à tout nouvel abonné.
* **Explication :** Un état a toujours une valeur actuelle. Le `BehaviorSubject` modélise parfaitement ce concept en
  garantissant qu'un composant qui s'abonne tardivement reçoive immédiatement l'état le plus récent, sans avoir à
  attendre un changement.

4. **Comment un composant doit-il demander une modification de l'état ?**

* **Réponse : C.** En appelant une méthode publique exposée par le service.
* **Explication :** C'est le principe de l'API publique du service stateful. Les composants ne modifient jamais l'état
  directement ; ils demandent une modification via une méthode, qui contient la logique pour mettre à jour l'état
  interne.

5. **Inconvénient potentiel du "store fait-main" :**

* **Réponse type :** Pour une très grande application, cette approche peut manquer de structure. La logique de
  modification est dispersée, il n'y a pas d'outils de débogage avancés (comme le time-travel debugging), et la gestion
  des effets de bord (appels API) n'est pas formalisée, ce qui peut rendre le code plus difficile à maintenir et à
  déboguer en équipe.

### Pour aller plus loin

1. **Principe qui N'APPARTIENT PAS à NgRx/Redux :**

* **Réponse : B.** Les composants peuvent modifier directement l'état pour plus de simplicité.
* **Explication :** C'est l'exact opposé du principe fondamental de NgRx qui stipule que l'état est en lecture seule (
  `readonly`). Toute modification doit passer par le dispatch d'une action.

2. **Rôle d'un `Effect` :**

* **Réponse type :** Un `Effect` gère les effets de bord (side effects) comme les appels API, l'accès au `localStorage`,
  ou les WebSockets. Il écoute les actions, effectue une tâche asynchrone, puis dispatche une nouvelle action (de succès
  ou d'échec) en réponse. Il ne modifie jamais directement l'état pour respecter le principe de séparation des
  préoccupations : les `Reducers` sont les seuls responsables synchrones et purs du calcul de l'état.

3. **Rôle principal d'un `Reducer` :**

* **Réponse : B.** Calculer un nouvel état en réponse à une action.
* **Explication :** Le reducer est une fonction pure. Sa seule et unique responsabilité est de prendre l'état actuel et
  une action, et de retourner un nouvel état.

4. **Pourquoi les `Selectors` sont-ils performants ?**

* **Réponse type :** Les `Selectors` sont performants car ils sont "mémoïsés" (memoized). Cela signifie que NgRx met en
  cache le résultat d'un selector. Si le selector est appelé à nouveau et que les parties de l'état dont il dépend n'ont
  pas changé, il retourne instantanément le résultat mis en cache au lieu de refaire le calcul.

5. **Première chose qu'un composant fait pour initier un changement d'état avec NgRx :**

* **Réponse : C.** Il dispatche une `Action` au `Store`.
* **Explication :** C'est le point de départ de tout le flux NgRx. L'action est la description de l'événement qui a eu
  lieu.

---

*... Je vais continuer sur ce modèle pour les modules restants. Les réponses et explications sont générées en suivant la
logique du cours et les principes fondamentaux de chaque technologie abordée.*

---

## Module 3 : Patrons de Conception de Composants et Optimisation

### L'essentiel

1. **Responsabilité qui N'APPARTIENT PAS à un composant de présentation :**

* **Réponse : B.** Injecter un service pour récupérer des données.
* **Explication :** C'est la responsabilité d'un composant "Container" (intelligent). Un composant de présentation ("
  Dumb") est isolé et ne connaît pas l'origine des données ; il les reçoit uniquement via `@Input()`.

2. **Pourquoi l'immutabilité est-elle nécessaire pour `OnPush` ?**

* **Réponse type :** La stratégie `OnPush` optimise la détection de changement en ne se redéclenchant que si les
  références des `@Input()` changent. Si on mute une propriété d'un objet (`user.name = 'new'`), la référence de l'objet
  `user` reste la même, et Angular `OnPush` ne voit aucun changement. En créant un nouvel objet (
  `user = { ...user, name: 'new' }`), on change la référence, ce qui déclenche la détection de changement.

3. **Principal bénéfice de `trackBy` :**

* **Réponse : B.** Cela empêche Angular de détruire et recréer inutilement des éléments du DOM lorsque la liste est mise
  à jour.
* **Explication :** En fournissant un identifiant unique, `trackBy` aide Angular à reconnaître les éléments qui ont
  simplement été déplacés ou modifiés, lui permettant d'effectuer des mises à jour du DOM beaucoup plus efficaces et
  performantes.

4. **Comment un "Container" est-il informé d'une action dans un "Presentational" ?**

* **Réponse type :** Le composant de présentation émet un événement via un `@Output()` décoré d'un `EventEmitter`. Le
  composant conteneur écoute cet événement dans son template (`(userAction)="..."`) et appelle une de ses méthodes en
  réponse.

5. **Événement qui NE déclenchera PAS la détection de changement `OnPush` :**

* **Réponse : C.** Son composant parent modifie une propriété d'un objet qui a été passé en `@Input()` au composant
  `OnPush`.
* **Explication :** C'est le piège classique de la mutation. Comme la référence de l'objet n'a pas changé, le composant
  `OnPush` ne se mettra pas à jour.

### Pour aller plus loin

1. **Bénéfice principal du Lazy Loading :**

* **Réponse : B.** Il réduit la taille du bundle JavaScript initial, améliorant le temps de chargement perçu.
* **Explication :** Le Lazy Loading est une technique de "code splitting". Elle permet de ne charger le code d'une
  section de l'application que lorsque l'utilisateur en a besoin, ce qui rend le chargement initial beaucoup plus
  rapide.

2. **Rôle de `import()` dans le lazy loading :**

* **Réponse type :** `import()` est la syntaxe d'importation dynamique de JavaScript. Contrairement à l'instruction
  `import` statique, elle est traitée à l'exécution et retourne une `Promise`. Les outils de build (comme Vite ou
  Webpack) utilisent cette syntaxe comme un signal pour créer un "chunk" (un fichier JavaScript séparé) pour le module
  importé, ce qui est la base du code splitting.

3. **Path pour une sous-route en lazy loading :**

* **Réponse : C.** `edit`.
* **Explication :** Le fichier de routes d'une section chargée paresseusement hérite du préfixe de la route parente. Si
  la route parente est `path: 'profile'`, la route pour `/profile/edit` à l'intérieur de `profile.routes.ts` aura
  simplement `path: 'edit'`.

4. **Comment vérifier que le lazy loading fonctionne ?**

* **Réponse type :** En utilisant l'onglet "Réseau" (Network) des outils de développement du navigateur. Au chargement
  initial, seul le bundle principal est chargé. Lorsque l'on navigue vers la route "paresseuse", on doit voir une
  nouvelle requête réseau apparaître pour télécharger le "chunk" JavaScript correspondant à cette section.

5. **Meilleur candidat pour le lazy loading :**

* **Réponse : C.** Une section "Administration" complexe, accédée seulement par un petit nombre d'utilisateurs.
* **Explication :** Il est inutile de faire charger le code de la section admin à tous les utilisateurs, surtout si la
  plupart d'entre eux n'y accéderont jamais. C'est le candidat parfait pour être séparé du bundle principal.

---

## Module 4 : Gardiens et Intercepteurs

### L'essentiel

1. **Rôle principal d'un garde `CanActivate` :**

* **Réponse : B.** Contrôler si un utilisateur peut accéder à une route.
* **Explication :** `CanActivate` est comme un videur à l'entrée d'une route. Il s'exécute avant que la route ne soit
  activée et sa réponse (`true` ou `false`) détermine si la navigation peut se poursuivre.

2. **Pourquoi utiliser `request.clone()` dans un `HttpInterceptor` ?**

* **Réponse type :** Les objets `HttpRequest` (et `HttpResponse`) sont immuables. Cela signifie qu'on ne peut pas les
  modifier directement. Pour ajouter un header, on doit créer une copie de la requête originale en utilisant la méthode
  `.clone()` et en lui passant les modifications souhaitées.

3. **Garde pour un formulaire non sauvegardé :**

* **Réponse :** `CanDeactivate`.
* **Explication :** `CanDeactivate` est spécifiquement conçu pour se déclencher lorsqu'on tente de quitter une route.
  C'est l'outil parfait pour vérifier une condition (comme un formulaire "dirty") et potentiellement annuler la
  navigation.

4. **Comment enregistrer un `HttpInterceptor` ?**

* **Réponse : B.** En utilisant la fonction `withInterceptors()` dans l'appel à `provideHttpClient()`.
* **Explication :** Dans les applications Angular modernes (standalone), c'est la méthode recommandée pour fournir des
  intercepteurs fonctionnels à l'application. `provideHttpClient(withInterceptors([myInterceptor]))`.

5. **Un `Route Guard` peut-il rediriger ? Si oui, comment ?**

* **Réponse type :** Oui. Au lieu de retourner `false`, un garde peut retourner une `UrlTree`. La manière la plus simple
  de créer cette `UrlTree` est d'injecter le `Router` et de retourner `router.parseUrl('/login')`, ce qui annulera la
  navigation actuelle et en déclenchera une nouvelle vers la page de connexion.

### Pour aller plus loin

1. **Avantage principal d'un `Resolver` :**

* **Réponse : B.** Il permet de charger les données nécessaires à un composant avant son affichage, améliorant l'UX.
* **Explication :** En s'assurant que les données sont disponibles avant que le composant ne soit rendu, les `Resolvers`
  éliminent les états de chargement intermédiaires ("flickering UI"), ce qui donne une impression de fluidité et de
  performance.

2. **Comment accéder aux données d'un `Resolver` ?**

* **Réponse type :** Dans le composant de la route, on injecte `ActivatedRoute`. Les données résolues sont disponibles
  sur la propriété `data` de cet objet (`this.route.data`), qui est un `Observable` contenant un objet où les clés
  correspondent à celles définies dans la propriété `resolve` de la configuration de la route.

3. **Ordre d'exécution dans le cycle de vie d'une navigation :**

* **Réponse : C.** `CanActivate` Guard -> `Resolver` -> Affichage du Composant.
* **Explication :** Le routeur vérifie d'abord si l'accès est autorisé (`CanActivate`), puis il prépare les données (
  `Resolver`), et seulement si tout réussit, il instancie et affiche le composant.

4. **Où stocker un JWT pour qu'il persiste ?**

* **Réponse : C.** Dans `localStorage`.
* **Explication :** `localStorage` persiste même après la fermeture du navigateur. `sessionStorage` est vidé à la
  fermeture de l'onglet/navigateur, et une variable de service est vidée à chaque rechargement de la page.

5. **Rôle PRÉCIS de l'intercepteur dans le flux JWT :**

* **Réponse type :** Le rôle de l'`HttpInterceptor` est d'intercepter **chaque requête HTTP sortante**, de vérifier si
  un JWT est stocké, et si c'est le cas, de **cloner la requête pour y ajouter le header `Authorization: Bearer <token>`
  ** avant de la laisser continuer vers le serveur. Il centralise cette logique pour qu'on n'ait pas à le faire
  manuellement dans chaque service.

---

## Module 5 : Tests Unitaires

### L'essentiel

1. **Rôle de `Karma` :**

* **Réponse : B.** Il exécute les tests dans un vrai navigateur et affiche les résultats.
* **Explication :** Karma est le "lanceur de tests". Jasmine fournit la structure (`describe`, `it`), et Karma prend ce
  code, démarre un navigateur, y exécute les tests et rapporte les succès/échecs.

2. **Utilité du `HttpTestingController` :**

* **Réponse type :** `HttpTestingController` est un outil du module `HttpClientTestingModule` qui nous permet
  d'interagir avec le faux `HttpClient`. Il nous permet de faire des assertions sur les requêtes qui sont faites (URL,
  méthode) et de simuler des réponses ou des erreurs du serveur de manière contrôlée.

3. **À quoi sert `fixture.detectChanges()` ?**

* **Réponse type :** `fixture.detectChanges()` déclenche manuellement un cycle de détection de changement d'Angular.
  C'est essentiel dans les tests pour mettre à jour la vue (le DOM) après avoir modifié une propriété de l'instance du
  composant, afin de pouvoir vérifier que le rendu HTML a bien changé.

4. **Différence entre `fixture.componentInstance` et `fixture.nativeElement` :**

* **Réponse : B.** `componentInstance` est la classe TypeScript, `nativeElement` est l'élément HTML rendu.
* **Explication :** On utilise `componentInstance` pour interagir avec les propriétés et les méthodes de la classe du
  composant (ex: `component.name = 'Test'`). On utilise `nativeElement` pour interagir avec le DOM généré (ex:
  `nativeElement.querySelector('h1')`).

5. **Pourquoi isoler les tests ?**

* **Réponse type :** L'isolation garantit qu'un test n'est pas influencé par l'état laissé par un test précédent. Cela
  rend les tests plus fiables, moins "flaky" (aléatoires), et plus faciles à déboguer. Si un test échoue, on sait que
  c'est à cause de la logique qu'il teste, et non à cause d'un effet de bord d'un autre test.

### Pour aller plus loin

1. **Comment fournir une fausse version d'un service dans le `TestBed` ?**

* **Réponse : C.** `providers: [{ provide: RealService, useValue: myFakeService }]`.
* **Explication :** C'est le mécanisme d'injection de dépendances de test d'Angular. On utilise la clé `provide` pour
  spécifier le "token" du service réel, et `useValue` (ou `useClass`, `useFactory`) pour dire à l'injecteur de fournir
  notre mock à la place.

2. **Différence entre `jasmine.createSpyObj` et `spyOn` :**

* **Réponse type :** `jasmine.createSpyObj` crée un objet factice complet à partir de rien, qui simule une interface
  avec des méthodes espionnes. `spyOn`, lui, s'attache à une méthode sur un objet **existant et réel** pour l'espionner,
  potentiellement en laissant l'implémentation originale s'exécuter.

3. **Meilleure stratégie pour tester `debounceTime(500)` :**

* **Réponse : B.** La zone `fakeAsync` avec un appel à `tick(500)`.
* **Explication :** `fakeAsync` nous donne le contrôle de l'horloge. `tick(500)` simule l'écoulement de 500ms
  instantanément, rendant le test rapide et déterministe, ce qui est idéal pour des opérateurs temporels comme
  `debounceTime` ou `delay`.

4. **À quoi sert `tick()` dans `fakeAsync` ?**

* **Réponse type :** La fonction `tick(milliseconds)` sert à simuler l'écoulement du temps à l'intérieur d'une zone
  `fakeAsync`. Elle fait avancer l'horloge virtuelle, ce qui déclenche l'exécution des tâches asynchrones en attente (
  comme les `setTimeout` ou les `Promise`).

5. **Comment vérifier qu'un espion a été appelé une fois avec un argument ?**

* **Réponse : C.** `expect(mySpy).toHaveBeenCalledTimes(1); expect(mySpy).toHaveBeenCalledWith('hello');`
* **Explication :** C'est la combinaison des deux "matchers" Jasmine les plus courants pour ce cas d'usage :
  `toHaveBeenCalledTimes` pour vérifier le nombre d'appels, et `toHaveBeenCalledWith` pour vérifier les arguments du (
  dernier) appel.

---

## Module 6 : Écosystème Avancé

### Partie A : Design System & Composants Dynamiques

1. **Avantage de `ng add @angular/material` :**

* **Réponse : B.** La commande configure automatiquement le thème, la typographie et les animations.
* **Explication :** `ng add` utilise les "schematics" d'Angular Material pour non seulement installer les dépendances,
  mais aussi pour intégrer la bibliothèque proprement dans le projet (configuration, styles, etc.), ce qui fait gagner
  beaucoup de temps.

2. **Concept de "projection de contenu" :**

* **Réponse type :** C'est un mécanisme qui permet à un composant parent de passer du contenu HTML directement à l'
  intérieur du template d'un composant enfant. La balise `<ng-content>` dans le composant enfant agit comme un "
  placeholder" ou un "slot" où ce contenu sera inséré.

3. **Scénario pour les composants dynamiques :**

* **Réponse : C.** Pour construire un tableau de bord où l'utilisateur peut choisir les "widgets" à afficher à partir
  d'une liste.
* **Explication :** C'est le cas d'usage parfait, car le type et le nombre de composants à afficher ne sont pas connus
  au moment du développement, mais sont déterminés à l'exécution en fonction des actions de l'utilisateur ou d'une
  configuration.

4. **Comment définir plusieurs "slots" de projection ?**

* **Réponse type :** En utilisant l'attribut `select` sur les balises `<ng-content>`. Chaque `ng-content` peut avoir un
  sélecteur CSS (`select="[mon-attribut]"` ou `select=".ma-classe"`) pour cibler spécifiquement quel contenu du parent
  doit être projeté dans quel slot.

5. **Classe fondamentale pour créer des composants dynamiques :**

* **Réponse : D.** `ViewContainerRef`.
* **Explication :** `ViewContainerRef` est l'API moderne et principale pour créer et attacher programmatiquement des
  composants à un point d'ancrage spécifique dans le DOM.

### Partie B : Temps Réel avec WebSockets

1. **Différence fondamentale entre HTTP et WebSocket :**

* **Réponse : C.** HTTP est un protocole basé sur une connexion qui se ferme après chaque réponse, tandis que WebSocket
  maintient une connexion persistante.
* **Explication :** C'est la différence essentielle. La connexion persistante et bidirectionnelle de WebSocket est ce
  qui lui permet de faire de la communication temps réel (push) que HTTP ne peut pas faire nativement.

2. **Correspondance entre `onmessage` et l'`Observer` RxJS :**

* **Réponse :** `next`.
* **Explication :** `onmessage` est l'événement qui se déclenche quand une nouvelle donnée arrive du serveur. Dans notre
  wrapper, on capture cette donnée et on la pousse dans le flux de l'observable via `observer.next(data)`.

3. **Pourquoi retourner `socket.close()` ?**

* **Réponse type :** C'est la fonction de "teardown" (nettoyage) de l'observable. Elle est exécutée automatiquement
  lorsque le dernier abonné se désabonne. Retourner une fonction qui ferme la connexion garantit que lorsque le
  composant est détruit, la connexion WebSocket est proprement fermée, évitant ainsi les fuites de ressources côté
  client et serveur.

4. **Que faire avec un objet JS avant de l'envoyer via WebSocket ?**

* **Réponse : C.** Utiliser `JSON.stringify()` pour le convertir en chaîne de caractères.
* **Explication :** Le protocole WebSocket transporte des données sous forme de "frames" qui contiennent généralement du
  texte ou des données binaires. La convention la plus répandue pour envoyer des objets structurés est de les sérialiser
  en une chaîne JSON.

5. **Vrai ou Faux : Une fois la connexion établie, seul le serveur peut initier l'envoi d'un message.**

* **Réponse :** Faux.
* **Explication :** WebSocket est un protocole **bidirectionnel**. Une fois la connexion établie, le client et le
  serveur peuvent tous deux envoyer des messages à tout moment, de manière indépendante.

### Partie C : Build, Déploiement et PWA

1. **Ce qui N'EST PAS une optimisation de `ng build` :**

* **Réponse : C.** Ajout de commentaires détaillés dans le code.
* **Explication :** C'est le contraire. La minification, une des optimisations clés, supprime tous les commentaires pour
  réduire la taille des fichiers.

2. **Le "problème du rafraîchissement F5" et sa solution :**

* **Réponse type :** Le problème survient quand on rafraîchit une page sur une route profonde (ex: `/users/1`). Le
  navigateur envoie une requête pour cette URL au serveur, qui ne la connaît pas et retourne une erreur 404. La solution
  est la réécriture d'URL côté serveur : il faut configurer le serveur pour que toute requête qui ne correspond pas à un
  fichier existant soit redirigée vers `index.html`, permettant à l'application Angular de se charger et de gérer le
  routage.

3. **Rôle du `manifest.webmanifest` :**

* **Réponse type :** C'est la "carte d'identité" de l'application web. Ce fichier JSON fournit au navigateur des
  métadonnées comme le nom de l'application, ses icônes, ses couleurs de thème, et son comportement d'affichage (
  `standalone`), ce qui permet au navigateur de proposer à l'utilisateur d'installer l'application sur son appareil.

4. **Composant PWA responsable du cache hors-ligne :**

* **Réponse : B.** Le Service Worker.
* **Explication :** Le Service Worker agit comme un proxy réseau. Il peut intercepter les requêtes, et s'il est
  configuré pour le faire, il peut servir des réponses directement depuis le cache, permettant à l'application de
  fonctionner même sans connexion internet.

5. **Commande CLI pour transformer une app en PWA :**

* **Réponse :** `ng add @angular/pwa`.

---

## Module 7 : i18n et a11y

### L'essentiel (i18n)

1. **Commande pour extraire les textes :**

* **Réponse : C.** `ng extract-i18n`.

2. **Avantage d'un ID personnalisé `@@id` :**

* **Réponse type :** Il fournit un identifiant stable et unique pour une unité de traduction. Si le texte source en
  français change légèrement (une correction de typo par exemple), Angular générerait un nouvel ID par défaut, et la
  traduction existante serait perdue. Avec un `@@id` personnalisé, le lien est préservé, ce qui rend la maintenance des
  traductions beaucoup plus robuste.

3. **Fichier de configuration pour l'i18n :**

* **Réponse :** `angular.json`.
* **Explication :** C'est dans ce fichier, sous la clé `projects.mon-app.i18n`, qu'on définit la langue source et les
  locales cibles avec les chemins vers leurs fichiers de traduction.

4. **Vrai ou Faux : `@angular/localize` permet de changer de langue dynamiquement.**

* **Réponse :** Faux.
* **Explication :** L'approche d'i18n par défaut d'Angular est une approche au moment du build. Elle génère une
  application distincte et optimisée pour chaque langue. Pour changer de langue, l'utilisateur doit être redirigé vers
  l'application correspondant à l'autre langue (ex: de `/fr/home` à `/en/home`). Il ne peut pas changer la langue "à la
  volée" au sein de la même session d'application.

5. **Format de fichier par défaut pour les traductions :**

* **Réponse : C.** XLIFF (`.xlf`).

### Pour aller plus loin (a11y)

1. **Pourquoi `<button>` est préférable à `<div (click)>` ?**

* **Réponse : C.** `<button>` est nativement focusable, utilisable au clavier, et a une sémantique correcte pour les
  technologies d'assistance.
* **Explication :** Utiliser un élément HTML sémantique natif vous donne gratuitement un tas de fonctionnalités
  d'accessibilité qu'il faudrait sinon recréer manuellement et de manière imparfaite.

2. **Rôle de l'attribut ARIA `role` :**

* **Réponse type :** L'attribut `role` permet de surcharger la sémantique native d'un élément ou de donner une
  sémantique à un élément qui n'en a pas (comme une `div`). Il informe les technologies d'assistance sur le "métier" ou
  le "type" d'un widget (ex: `role="tab"`, `role="switch"`, `role="alert"`).

3. **Comment étiqueter un bouton avec une icône seulement ?**

* **Réponse : B.** `aria-label="Rechercher"`.
* **Explication :** `aria-label` fournit une étiquette textuelle accessible qui est lue par les lecteurs d'écran, même
  s'il n'y a pas de texte visible dans le DOM.

4. **Ratio de contraste minimum recommandé :**

* **Réponse :** 4.5:1.

5. **Outil d'audit d'accessibilité intégré aux navigateurs Chromium :**

* **Réponse :** Lighthouse.

---

## Module 8 : Tests E2E et Maintenance

### L'essentiel

1. **Différence entre test unitaire et test E2E :**

* **Réponse : B.** Les tests unitaires vérifient des pièces de code isolées, les tests E2E vérifient un parcours
  utilisateur complet dans un navigateur.
* **Explication :** C'est la différence d'échelle. Un test unitaire teste la "brique", un test E2E teste la "maison" une
  fois assemblée.

2. **Pourquoi Cypress est-il plus rapide/fiable que Protractor ?**

* **Réponse type :** Cypress s'exécute directement dans le même thread que l'application, ce qui lui donne un accès
  direct au DOM, au réseau et à l'application elle-même. Cela élimine la couche de communication externe et les délais
  inhérents à l'architecture de Selenium/WebDriver, rendant les tests moins sujets aux erreurs de synchronisation ("
  flakiness") et plus rapides.

3. **Meilleure pratique pour sélectionner des éléments dans Cypress :**

* **Réponse : C.** Utiliser un attribut HTML dédié aux tests, comme `data-cy`.
* **Explication :** Cela découple les tests de l'implémentation (classes CSS) et du contenu (texte), les rendant
  beaucoup plus robustes face aux changements de l'UI.

4. **Commande Cypress pour taper du texte :**

* **Réponse : D.** `.type()`.

5. **Dernière étape de la recette de base d'un test Cypress :**

* **Réponse :** Vérifier (`.should()`).
* **Explication :** La recette est Visiter (`visit`), Trouver (`get`), Agir (`click`, `type`...), et enfin Vérifier (
  `should`) que l'action a eu l'effet escompté.

### Pour aller plus loin

1. **Avantage principal de `cy.intercept()` :**

* **Réponse : B.** Cela rend les tests plus rapides et plus fiables en les découplant du backend.
* **Explication :** En mockant les réponses de l'API, les tests ne dépendent plus de la disponibilité ou de la vitesse
  du serveur de test. Ils peuvent s'exécuter rapidement et de manière déterministe, en testant des scénarios de succès
  comme d'échec de manière contrôlée.

2. **Qu'est-ce qu'un "schematic de migration" ?**

* **Réponse type :** C'est un script, fourni par une bibliothèque (comme `@angular/core`), qui est exécuté par la
  commande `ng update`. Son rôle est de modifier automatiquement le code source de l'utilisateur pour l'adapter aux
  changements d'API (breaking changes) d'une nouvelle version, ce qui automatise une grande partie du processus de mise
  à jour.

3. **Meilleur outil pour planifier une mise à jour d'Angular v15 à v17 :**

* **Réponse :** Le site web officiel **[update.angular.io](https://update.angular.io/)**.

4. **Vrai ou Faux : `ng update` ne modifie jamais votre code source.**

* **Réponse :** Faux.
* **Explication :** C'est justement la grande force de `ng update` ! Grâce aux schematics de migration, il peut et va
  souvent modifier votre code pour vous faire gagner du temps lors des mises à jour.

5. **Raison la plus critique de maintenir à jour ses dépendances :**

* **Réponse type :** Bien que la performance et les nouvelles fonctionnalités soient importantes, la raison la plus
  critique est la **sécurité**. Les mises à jour corrigent des failles de sécurité qui sont découvertes. Ne pas mettre à
  jour expose votre application et vos utilisateurs à des risques connus.

---

## Module 9 : Conclusion

1. **Quand NgRx est-il particulièrement justifié ?**

* **Réponse : B.** Quand plusieurs composants non liés doivent partager et modifier un état complexe, et que la
  traçabilité des changements est importante.
* **Explication :** NgRx brille lorsque la complexité de l'état et le nombre d'acteurs interagissant avec cet état
  deviennent élevés. Sa structure stricte et ses outils de débogage apportent une prévisibilité et une maintenabilité
  qu'un simple service a du mal à offrir à grande échelle.

2. **Objectif principal de la stratégie `OnPush` :**

* **Réponse : C.** Réduire le nombre de vérifications effectuées par Angular, améliorant ainsi les performances.
* **Explication :** `OnPush` est une pure stratégie d'optimisation des performances de rendu.

3. **But d'un test End-to-End :**

* **Réponse : C.** Valider qu'un parcours utilisateur complet fonctionne comme prévu dans un vrai navigateur.
* **Explication :** Il teste l'intégration de toutes les parties de l'application du point de vue de l'utilisateur.

4. **Problème majeur résolu par le SSR :**

* **Réponse : C.** Le mauvais référencement (SEO) et la lenteur du premier affichage des Single Page Applications.
* **Explication :** En servant une page HTML déjà rendue, le SSR permet aux robots des moteurs de recherche d'indexer le
  contenu et aux utilisateurs de voir quelque chose quasi-instantanément.

5. **Quelle notion a le plus changé votre manière de penser ?**

* **Réponse :** Cette question est personnelle et ouverte. Une bonne réponse pourrait mentionner :
  *   **RxJS et la pensée réactive**, car cela change la manière de gérer l'asynchronisme.
  *   **La gestion d'état centralisée**, car cela résout le problème du "prop drilling" et de la communication entre
  composants.
  *   **Le patron Container/Presentational**, car il impose une structure claire et favorise la réutilisabilité.
  *   **Les tests**, car ils apportent la confiance pour refactoriser et maintenir le code.