# La gestion d'erreur

## Objectifs
- Savoir gérer les erreurs
- Recommencer une opération avec les opérateurs `retry` et `retryWhen`

## Introduction
Il n'est pas impossible (voir très probable) que le code exécuté par un `Observable` ou l'un des opérateurs présents dans la chaine produise une erreur. 
Lorsqu'une erreur est émise dans ce cadre, RxJS va tout simplement parcourir la suite de la chaine d'opérateurs jusqu'à en trouver un permettant de gérer cette erreur.
À savoir que ces opérateurs sont: `catchError`, `retry`, `retryWhen` (Que nous allong voir immédiatement).
Si aucun de ces opérateur n'est rencontré: L'erreur est envoyé dans le callback `onError` de l'`Observer` qui a souscrit à cet `Observable`.
Enfin si cet Observer n'a pas implémenter `onError`, l'erreur sera propagée et stopera potentiellement l'exécution du moteur javascript (Comme une erreur classique)


## Intercepter des erreurs: L'opérateur `catchError`
Comme son nom l'indique, il permet d'attraper des erreurs. Cette opérateur retourne un nouvel observable qui sera passé à la suite de la chaine.
Si aucune erreur est générée en amont cet opérateur sera tout simplement ignoré.

![Catch error schema](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/catch.png)

Le schèma ci-dessus représente le code ci-dessous:

```typescript
import { of } from 'rxjs';
import { map, catchError } from 'rxjs/operators';
 
of('a', 'b', 'c').pipe(
    map(x => {
  	   if (x === 'c') {
  	       // We just don't like the letter c, let send an error
	       throw "We hate 'c'!";
      }
     return x;
    }),
    catchError(err => {
        // Do whatever you want with the error here
        // We'll just return another observable
        return of(1, 2, 3);
    }),
  )
  .subscribe(x => console.log(x));
```

Dans cet exemple, nous utilisons l'opérateur tel un `try/catch` classique: Une erreur est émise, on souhaite l'attraper pour continuer l'exécution de notre flux.
Il existe une autre situation un peu moins classic: il peut permettre à "convertir" une erreur: Il suffit de throw une nouvelle erreur dans l'opérateur.

Voila un exemple un peu basique et sans sens particulier, mais il démontre tout de même que cela est faisable:

```typescript
import { of } from 'rxjs';
import { map, catchError } from 'rxjs/operators';
 
of(1, 2, 3, 4, 5).pipe(
    map(n => {
      if (n === 4) {
        throw 'four!';
      }
      return n;
    }),
    catchError(err => {
      throw 'error in source. Details: ' + err;
    }),
  )
  .subscribe(
    x => console.log(x),
    err => console.log(err)
  );
  // 1, 2, 3, error in source. Details: four!
```

## Retenter une opération: Les opérateurs `retry` et `retryWhen`
Ces 2 opérateurs fonctionnent de la même manière: lorsqu'une erreur est émise dans la chaine en amont, les opérateurs `retry` vont exécuter à nouveau l'observateur (Comme si on souscript à nouveau à cet `Observable)
La différence entre les 2 opérateurs se fait dans les paramètres

### L'opérateur `retry`
L'opérateur `retry` prends un seul paramètre en arguement. Il s'agit d'un nombre qui représente le nombre de tentative à effectuer avant de finalement passer l'erreur. Elle peut être pratique lors d'un appel HTTP: celui-ci peut échouer à cause d'un problème de réseau, il peut être intéressant de retenter l'appel HTTP une seconde fois.

![Retry operator schema](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/retry.png)

### L'opérateur `retryWhen`
Cet opérateur prends également un seul paramètre: une fonction (que l'on appellera notifier) retournant un observable. Cette fonction va nous permettre d'affiner le comportement de `retry`.
Le notifier est donc une fonction:
- Qui prends en argument un paramètre qui sera l'erreur émise pendant l'exécution de l'observable source: Nous allons donc pouvoir spécifier un comportement différent en fonction de l'erreur qui a été déclenchée en amont.
- Qui retourne un `Observable` (que l'on appellera innerObservable). C'est celui-ci qui rythmera la tentative de "retry": À chaque nouvelle donnée déclenchée par cet innerObservable (Par ailleurs ce que contient cette donnée a strictement aucune importance), l'Observable source sera déclenché à nouveau. 


Si l'innerObservable émets à son tour une erreur ou passe en status complété. L'exécution de l'observable source est interrompu.

![Retry when schema](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/retryWhen.png)  
 
