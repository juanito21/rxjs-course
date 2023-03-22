# Hot, Cold & Subjects

## Objectifs
- Distinguer les observables "hot" et "cold"
- Comprendre les `Subject`

## Observables: Cold vs Hot ?
Il existe en effet 2 types d'Observable, les "chauds" (hot) et les froids (cold), leurs comportements sont bien différents.
Une [métaphore](https://blog.strongbrew.io/my-favorite-metaphor-for-hot-vs-cold-observables/) permet d'imager facilement ces 2 types d'observables: Regarder un film.

De nos jours, les 2 façons les plus populaires pour regarder un film sont le cinéma ou Netflix!

### Les observables froids aka Netflix

Les observables froids ont un comportement similaire à Netflix: Lorsque l'on a trouvé notre film (aka l'observable) et que l'on souhaite le regarderer (aka subscribe),
le film va démarrer depuis le début. Ce n'est pas plus compliqué que cela!

Les propriétés d'un observable "cold" sont:
- Si personne n'a souscrit à l'observable, celui-ci **ne s'exécute pas**.
- À chaque souscription à l'observable, l'observer est branché sur un nouveau flux, celui-ci n'est pas partagé avec les autres observers.

> Comme vous l'avez deviné c'est le comportement par défault. Tout observable créé est "cold" au départ.

Reprenons l'exemple vu en intro:

```typescript

import { range, pipe } from 'rxjs';
import { delay } from 'rxjs/operators'

let secondChronometer$ = range(0, 2).pipe(delay(1000))

secondChronometer$.subscribe(value => { console.log(`A: ${value}`) });

// Connectons un second observable 3 secondes plus tardd
setTimeout(() => { secondChronometer$.subscribe(value => { console.log(`B: ${value}`) }); }, 3000);

// Résultats affiché
// A: 0
// A: 1
// B: 0
// A: 2
// B: 1
// B: 2
```

Résultat: La seconde souscription a redéclenché une séquence dans l'observable qui lui est propre, nous avons donc 2 exécutions en parallèle.

### Les observables chauds aka Cinéma
Contrairement à un film visionné sur Netflix, la séance de cinéma va démarrer à une heure déterminée. 
Toute personne en retard attrapera le film en cours de route et aura donc loupé le début du film. 
Si vous quittez la salle de cinéma, le film continuera de tourner également. Tout comme un observable chaud!

Les propiétés d'un observable "hot" sont:
- L'observable chaud n'a pas besoin d'attendre une souscription pour commencer.
- Un seul flux est partagé parmis tous les souscripteurs.
- Lorsqu'un observer souscrit à un observable, ce premier ne sera pas capable de visionner les évènnements passés.



#### L'opérateur `share`:
Maintenant que tu sais distinguer les 2 comportements d'un `Observable`. Tu dois maintenant naturellement te demander comment passer de l'un à l'autre.
Comme dit plus haut: 
> Tout observable créé est "cold" au départ.

C'est l'utilisation de l'opérateur [`share`](https://www.learnrxjs.io/operators/multicasting/share.html) qui va te permettre de modifier ce comportement.

Transformons donc notre exemple plus haut en passant l'obervable en "Hot":

```typescript
import { range, pipe } from 'rxjs';
import { delay, share } from 'rxjs/operators'


let secondChronometer$ = range(0, 3).pipe(delay(1000)), share());

secondChronometer$.subscribe(value => { console.log(`A: ${value}`) });

// Connectons un second observable 3 secondes plus tardd
setTimeout(() => { secondChronometer$.subscribe(value => { console.log(`B: ${value}`) }); }, 3000);

// Résultats affiché
// A: 0
// A: 1
// A: 2
// B: 2
// A: 3
// B: 3
```

Résultat: La seconde souscription a loupé les 2 premiers événnements (0 et 2) et recois donc uniquement le dernier (4).

## Les subjects
On peut considérer les subjects comme des observable chauds "pré-conçus". Un `subject` implémente l'interface [`Observer`](https://github.com/ReactiveX/rxjs/blob/master/src/internal/Observer.ts#L5) (Soit les méthodes `next(value)`,  `error(err)`, `complete`).
C'est grâce à ses méthodes que nous allons transmettre des événnements dans le flux.
Les subjects peuvent être beaucoup plus simple à utiliser que les `Observable` dans certains contextes: Dans le cas d'un `Observable`, le développeur doit définir un comportement à l'avance dans une fonction qui sera passée à `Observable` lors de son instanciation. De l'autre coté, le `Subject` peut s'instancier librement et on peut lui passer les événnements (On entend par la des données) grâce à ses méthodes publiques issues de l'interface `Observer`.

 
Concrétement, reprenons une fois de plus l'exemple ci-dessus avec un `Subject`

```typescript
import { Subject } from 'rxjs';

let subject = new Subject(-1);
let value = 0;
subject.subscribe(n => console.log(n))

for (let i = 0; i < 4; i++){
    // Nous envoyons les valeurs dans le flux  
    subject.next(i);
} 
// Nous indiquons la fin de notre flux de données.
subject.complete();

// Affiche
// 0
// 1
// 2
// 3

```

> Note: Dans cet exemple nous souscrivons en amont car il n'est pas possible de déclencher l'exécution du flux à la souscription, il faut donc souscrire avant l'exécution du code métier'. 

Tout comme les opérateurs, il existe plusieurs variantes de `Subject` avec des comportements différents.

### BehaviourSubject
`BehaviourSubject` est une classe enfante de `Subject`, elle possède donc les mêmes méthodes. Le changement réside lors de la souscription. Petit rappel: Un Subject est un observable "hot", les évènnements passés avant la souscription sont "perdus".

C'est sur ce dernier point que le `BehaviourSubject` diffère, jetons un oeil au diagramme de séquence:

![Behaviour subject sequence](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.png)

Le diagramme de séquence ci-dessus représente 2 souscriptions à un `BehaviorSubject` initialisé avec une valeur violette.
 
Sur chacune des souscriptions, nous pouvons constater que l'`Observer` **reçoit le dernier évènnement passé** 

C'est l'unique différence entre un Subject et un BehaviorSubject: Lors de la souscription, l'`Observer` reçoit le dernier évènement émis avant sa souscription.

```typescript
import { BehaviorSubject } from 'rxjs';

let subject = new BehaviorSubject(-1);
let value = 0;
subject.subscribe(n => console.log(n))

for (let i = 0; i < 4; i++){
    // Nous envoyons les valeurs dans le flux  
    subject.next(i);
} 
// Nous indiquons la fin de notre flux de données.
subject.complete();

// Affiche
// -1
// 0
// 1
// 2
// 3

```

### Autres subjects

Il existe également le `ReplaySubject` et l'`AsyncSubject`, ce n'est pas le sujet du challenge de cette quête, si vous souhaitez en savoir plus à leurs sujets rendez-vous sur [cette page](https://rxjs-dev.firebaseapp.com/guide/subject).

// TODO: Challenge

