# Introduction à RxJS 

Pars à la rencontre de la programmation réactive avec RxJs !

## Objectifs

* Comprendre ce qu'est un observable
* Comprendre l'enchaînement d'opérateurs
* Comprendre la souscription

## Etapes

### Un peu d'histoire

*Avant de commencer ce chapitre, il est important de préciser que nous traiterons exclusivement des versions 6+ de RxJS dans nos exemples. Il se peut donc que tu trouves des exemples légérement différents sur internet.*

La programmation réactive est une façon de développer une application en écoutant des évènnements et d'y réagir.
Ce concept n'est pas du tout nouveau, mais il est devenu populaire grâce à Microsoft et sa librairie [Reactive](https://github.com/dotnet/reactive) pour son framework .Net fin 2012.
Depuis, les librairies Reactive Extensions ("Rx") ont fleuri dans la plupart des languages (RxJS, RxJava, RxCpp, RxSwift) et font de Reactive Extension un vrai standard cross-languages.

> Il te sera donc facile de communiquer avec des développeurs d'autres plateformes ou de reproduire un comportement sur un autre language, puisque les outils seront identiques.

#### Ressources

*[]()


### Rx, qu'est-ce que c'est?
Dans la programmation réactive, tout est **flux**. Chaque composant d'une application produit des événenements, tandis que d'autres les écoutent. Un **flux** est donc une **série d'événements** : ces événements peuvent être des valeurs, des erreurs ou un événement de terminaison.
Tous ces événements sont ensuites écoutés par un consommateur.

> Dans le cas des librairies Rx, les flux sont appelés des ["Observables"](http://reactivex.io/documentation/observable.html) et les consommateurs des ["Observer"](http://reactivex.io/rxjs/class/es6/MiscJSDoc.js~ObserverDoc.html).

Cet embranchement de flux et de consommateurs n'est pas nouveau, c'est ce qu'on appelle le design pattern [Observer](https://fr.wikipedia.org/wiki/Observateur_\(patron_de_conception\)).
La **puissance de Rx réside dans sa multitude d'opérateurs** : en effet, il est possible d'agir directement sur un flux pour filtrer, trier, rassembler ou encore temporiser des événements.
Par ailleurs, tous ces opérateurs possèderont un nom identique et un comportement identique dans les différents languages de programmation (RxJs, RxJava, RxSwift).

#### Ressources

*[]()

### Les Observables

Un **observable** est un flux. Pour t'en faire une idée concrète, consulte [cette vidéo](https://www.youtube.com/watch?v=XbOuCBuQepI).

Il existe cependant plusieurs façons d'en créer : nous pouvons directement donner une série de valeurs qui seront propagées, ou bien passer une fonction qui sera exécutée.

#### Créer un observable à partir d'une série de valeurs

La façon la plus simple de créer un observable est de le faire à partir d'une série de valeurs. Il suffit d'utiliser la fonction [`of`](https://www.learnrxjs.io/operators/creation/of.html). Cette fonction prend une série d'arguments en paramètre et les émet un à un aux consommateurs.

```typescript

import { of, Observable } from 'rxjs';

// On crée un observable qui émettra les valeurs 
const source: Observable<number> = of(1, 2, 3, 4, 5);
```

> Il est possible d'enchaîner des paramètres de types différents ! Exemple `of(1, "Bonjour")`. Mais attention à ce genre de pratiques, qui peuvent être compliquées à gérer. 

#### Créer un observable à partir d'une fonction

Cette méthode est légèrement plus compliquée, mais de loin la plus intéressante : grâce à la fonction [`create`](https://www.learnrxjs.io/operators/creation/create.html) à laquelle nous passons une fonction (qu'on appelera *callback*), il est possible d'exécuter des actions plus complexes.
**Ce callback passé à `create` sera exécuté à chaque nouvelle souscription à l'observable.** L'envoi de données à travers le flux se fait grâce à la méthode `next` de l'`Observer` passée en paramètre à notre callback.

```typescript
import { Observable } from 'rxjs';

const secondChronometer$ = Observable.create(function(observer) {
  let value = 0;
  // Nous temporisons l'envoie de valeur grâce à setInterval qui émettra une valeur chaque seconde
  const interval = setInterval(() => {
    observer.next(value);
    value++;
    if (value > 1000){
        // Nous indiquons la fin de notre flux de données.
        observer.complete();
    }
  }, 1000); // milliseconds

  // La fonction retournée permet de libérer la ressource.
  return () => clearInterval(interval);
});
```

Lors de l'utilisation de la fonction `create`, le développeur a deux responsabilités supplémentaires :
- La première est facultative, il s'agit d'indiquer la fin du flux de données grâce à l'utilisation de la méthode [`complete()`](http://reactivex.io/rxjs/class/es6/MiscJSDoc.js~ObserverDoc.html) de l'observer.
- La seconde est, en revanche, beaucoup plus importante. Il s'agit d'**interrompre la fonction lorsque l'observable est libéré (tu verras ça en détail plus bas) ou complété**.
Dans l'exemple ci-dessus, tu peux noter qu'à la dernière ligne, une fonction est retournée. Celle-ci sera exécutée lorsque le consommateur se décrochera de l'observable. Si cette fonction est omise, le compteur tournera indéfiniment. Donc si tu souscris 10 000 fois un même observable, tu auras 10 000 compteurs qui tourneront indéfiniment. (Ce qui n'est pas tout à fait vrai dans notre cas, car nous arrêtons l'observable dès que `value` atteint une valeur supérieure à 1000, grâce à `observer.complete()`).

> Note : le nom de la variable `secondChronometer$` se termine par un `$` uniquement pour respecter la convention de [nommage Angular](https://angular.io/guide/rx-library#naming-conventions-for-observables).

**Important** : L'utilisation de `setInterval` (et de son homologue `clearInterval`) nous permet d'illustrer, dans l'exemple ci-dessus, l'utilisation d'un observable qui effectue des opérations à intervalles de temps réguliers à partir de fonctions javascript qui devraient t'être familières.
À l'avenir, nous utiliserons des fonctions qui permettent de recréer le même comportement mais sont propres à RxJs :

```typescript
import { range, pipe } from 'rxjs';
import { delay } from 'rxjs/operators'

let secondChronometer$ = range(0, 1000).pipe(delay(1000))
```

Tu comprends maintenant que les librairies RxJS, bien que longues en termes d'apprentissage, peuvent te faire économiser énormément de temps ! 
Nous utiliserons ce dernier exemple dans les cas suivants si besoin, il faut simplement retenir que son comportement est identique à l'exemple plus haut comprenant `setInterval`.

#### Consommer un flux de données

Maintenant que nous savons créer un observable, il est temps de le consommer ! Et pour ce faire, nous utilisons la méthode `subscribe` d'un `Observable`, à laquelle nous pouvons donner 3 callbacks :
- Le premier sera appelé à chaque nouvelle donnée émise dans le flux
- Le second sera appelé à chaque fois qu'une erreur sera émise pendant l'exécution du flux
- Le troisième sera appelé lorsque le flux sera complété

Reprenons notre premier exemple :

```typescript
import { of, Observable } from 'rxjs';

// On crée un observable qui émettra les valeurs 
const source: Observable<number> = of(1, 2, 3, 4, 5);

source.subscribe(
  (value) => {
      console.log(value);
  }, 
  (error) => {
      console.error(error);
  },
  () => {
      console.log("The observable is closed")
  }
);
```

Cet exemple affichera donc:
```
1
2
3
4
5
The observable is closed
```

#### Libérer un flux
Comme dit plus haut, il est possible de libérer un observable (c'est même un devoir), afin de libérer les ressources que cet observable peut consommer. Attention, en cas d'oubli, dans la plupart des cas (99 %), cela crée une fuite mémoire (= ton application va consommer de plus en plus de mémoire et peut ralentir drastiquement la machine).

> Il est donc impératif de libérer une soucription à un observable lorsqu'on n'en a plus besoin. Dans le cas d'Angular, cela s'effectue la plupart du temps dans `ngOnDestroy` du cycle de vie d'un composant.

Voici un petit exemple d'un composant qui afficherait un décompte :

```typescript
const counter$ = of(5, 4, 3, 2, 1, 0).pipe(delay(1000));

export class ExampleComponent implements OnInit, OnDestroy {
    counter: number = 5;
    subscription: Subscription;
  
  constructor() {}
 
  ngOnInit() {
    // Start the counter and assign the latest value to the `counter` property of the component
    this.subscription = counter$.subscribe((counter) => this.counter = counter); 
  }
 
  ngOnDestroy(): void {
      this.subscription.unsubscribe();
  }
}
```

Grâce à l'appel de la méthode `unsubscribe()` de la souscription réalisée dans le `ngOnInit`, on notifie à l'Observable qu'on ne l'écoute plus et qu'il peut donc arrêter son activité.

#### Ressources

* [ESSENTIEL - Know:in:Mins - What is an Observable?](https://www.youtube.com/watch?v=XbOuCBuQepI) Présentation d'un observable en une minute.

### Les Opérateurs
Maintenant que l'on a bien fait chauffer les cerveaux, on va s'attaquer à la partie la plus intéressante des librairies Rx : **les opérateurs**.

Il existe plus de [70 opérateurs](http://reactivex.io/documentation/operators.html) (je peux déjà lire la joie dans tes yeux).
Ces opérateurs permettent de trier, agréger, temporiser, gérer les erreurs (et bien d'autres). Il est possible de créer ses propres opérateurs, mais il est fortement probable que tu n'en aies jamais besoin.


Rassure-toi, nous n'allons pas les voir tous un par un dans cette quête (ni même dans les suivantes). Tous les aborder serait un vrai parcours du combattant, tu les aborderas au fur et à mesure de tes besoins. D'ailleurs, si jamais tu es perdu·e, il est recommandé de [lire ce texte](http://reactivex.io/documentation/operators.html#tree) pour trouver l'opérateur qui te convient.

Nous allons simplement voir maintenant **comment les enchaîner**. Cela se fait grâce à la fonction `pipe(...)`.

Voici son utilisation avec les opérateurs `filter` et `map`. Ces opérateurs reprennent le comportement des méthodes [`filter`](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/Array/filter) et [`map`](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/Array/map) de la classe `Array`.

```typescript
import { of, Observable } from 'rxjs';
import { filter, map } from 'rxjs/operators';

const translate = ["Zero", "One", "Two", "Three", "Four", "Five", "Six"]

// On crée un observable qui émettra les valeurs 
const source: Observable<number> = of(0, 1, 2, 3, 4, 5, 6)
                                       .pipe(filter(x => x % 2),    // 0, 2, 4, 6
                                             map(x => translate[x])) // "Zero", "Two", "Four", "Six" 

```

#### Ressources

* [OPTIONNEL - ReactiveX - Operators](http://reactivex.io/documentation/operators.html#tree) Doc de référence sur les opérateurs.

### Conclusion

Pour résumer Rx en une ligne, on pourrait écrire :

> Observable -> \[Operators\] -> Observer = Subscription

Un **observable** (`Observable`) est un **flux** sur lequel on peut **enchaîner des opérateurs** (`Operators`), afin qu'un **consommateur** (`Observer`) puisse les **écouter**. Cette liaison entre un flux et son consommateur génère une `Subscription`, elle permet par la suite de les décrocher afin de libérer les ressources.

#### Ressources

*[]()

## Challenge

### A l'attaque des observables

1. Clone [ce repo](https://github.com/WildCodeSchool/RxJS_TDD).
2. Ouvre le fichier `src/00.intro.ts` et complète chacun des observables, afin de faire passer tous les tests.
*Tu peux vérifier les tests grâce à la commande `npm run test-intro` (note : n'oublie pas d'installer les dépendances).*
3. Colle le contenu de ton fichier `src/00.intro.ts` dans un gist et partage un lien vers ce dernier en guise de solution.

### Critères de validation

* Le lien donné en solution présente bien le contenu de `src/00.intro.ts` complété.
* La solution passe tous les tests du fichier `src/00.intro.ts`.

