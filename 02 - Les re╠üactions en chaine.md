# concatMap, switchMap, mergeMap


Dans l'une des quêtes précedente, nous avons évoqué les opérateurs, qu'il en existe une multitude (c'est à dire plus de 70!) et comment les enchainers avec la méthode `.pipe(...)`.
C'est donc l'heure de s'attaquer à quelques uns d'entre eux.

Dans cette quête, nous allons nous concentrer sur les différents opérateurs utiles aux réactions en chaîne.

## Objectifs

* Comprendre concatMap
* Comprendre switchMap
* Comprendre mergeMap

## Etapes

### Vocabulaire

- **Observable source** : c'est l'`Observable` principal, sur lequel on appel la méthode `.pipe(...)`
- **Inner Observable** : c'est un `Observable` qui est créé à l'intérieur d'un autre `Observable`, notamment aux travers **opérateurs** déclaré au sein de la méthode `.pipe(...)`
- **Obersable active** : c'est un `Observable` sur lequelle une `subscription` est active, c'est à dire que l'on est à l'écoute des valeurs (`.subscribe()`) qu'il émet (`.next()`).
- **Obersable complété** : c'est un `Observable` qui n'émet plus de valeur, dont le flux est terminé (`.complete()`).


![](https://media.giphy.com/media/XknChYwfPnp04/giphy.gif)

### concatMap 

Pour rappel un observable est un flux de données qui envoie plusieurs valeurs. 
L'opérateur `concatMap` accepte 1 seul argument : une fonction **callback** qui doit renvoyer un `Observable`.
L'`Observable` retourné est appelé **inner Observable**, car il est interne à l'`Observable` source...

> Pourquoi retourner un `Observable` dans ce callback? 
>
> Rappelez-vous: Un observable est un flux de données,
> il permet donc de faire des opérations asynchrones.
> C'est précisement grâce à ce dernier point que nous allons pouvoir effectuer des enchainements d'opérations. Une première action, comme un l'ajout d'un caractère dans un champs de recherche, pourrait déclencher une seconde action: la récupération des résults de recherche sur un serveur distant.

Il est créé au sein du `pipe` de l'`Observable` source.
Revenons plus précisement sur le comportement de `concatMap`, lorsque tu renvois un `Observable`, `concatMap` conserve l'odre d'appel des flux pour te restituer les valeurs des l'**inner Observable** dans le même ordre dont les valeurs ont été émises par l'`Observable` source. Pour être plus précis `concatMap` souscris à un **inner Observable** seulment si le précedent **inner Observable** est passé en état **complété** ce qui permet de garantir l'odre d'émission des valeurs.

![](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/concatMap.png)

#### Ressources
  
*  [Interactive diagrams of concatMap](https://rxmarbles.com/#concatMap)
*  [Learn concatMap](https://www.learnrxjs.io/operators/transformation/concatmap.html)

### switchMap

`switchMap` peut semblé similaire dans le comportement à `concatMap`, les deux différences sont :
- L'ordre des données en sortie n'est pas garanti car `switchMap` souscris à l'**inner Observable** même si l'**inner Observable** précedent n'est pas passé à l'état **complété**. 
- Si un **inner Observable** passe à l'état **complété** alors que les autres **inner Observable** sont toujours actifs...  ils seront tous simplement annulé !!

![](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/switchMap.png)

> Cet opérateur est pratique par exemple pour un champ de recherche: On souhaite afficher en priorité les résultats en lien avec la dernière recherche effectuée (et omettre les résultats de recherches précédentes si elles arrivent ensuite)

#### Ressources
  
*  [Interactive diagrams of switchMap](https://rxmarbles.com/#switchMap)
*  [Learn switchMap](https://www.learnrxjs.io/operators/transformation/switchmap.html)

### mergeMap

`mergeMap` est très similaire à `concatMap`, les deux différences sont : 
- Le contrôle du nombre d'**inner Observable** actif. Le nombre maximale d'**inner Observable** actif doit être fournis en deuxième argument de `mergeMap`.
- Les **inner Obersable** actif ne sont pas annulés automatiquement (contrairement à `switchMap`).  

![](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/mergeMap.png)

#### Ressources
  
*  [Interactive diagrams of switchMap](https://rxmarbles.com/#mergeMap)
*  [Learn switchMap](https://www.learnrxjs.io/operators/transformation/mergemap.html)

### Une dernière chose


**Important**: Il faut également savoir que l'innerObservable peut avoir un [typage générique](https://www.typescriptlang.org/docs/handbook/generics.html#generic-classes) différent de celui de son observable parent.

Pour illustrer ce terme barbare, reprenons l'exemple cité dans le premier chapitre de cette quête:

> Une première action, comme un l'ajout d'un caractère dans un champs de recherche, pourrait déclencher une seconde action: la récupération des résults de recherche sur un serveur distant.

Illustrons cet exemple directement avec du code:

```typescript
import { BehaviorSubject, Observable } from 'rxjs';
import { switchMap, pipe} from "rxjs/operators";

interface ISearchResult {
    title: string;
    summary: string;
}

let termObs$ = new BehaviorSubject<string>("");

let fetchValue: (term: string) => Observable<ISearchResult[]> = term => {
    // We fake Angular HttpClient here
    // and pass our search term to the API
    // in order to fetch related query results
    // We assume the method `HttpClient.get(url: string)`
    // returns an `Observable` wich emits list of results (eg: `ISearchResult[]`)
    return httpClient.get<ISearchResult[]>(`https://YOUR_API_ADRESSE/search`, {params: {term}})
}
}

let searchResult$: Observable<ISearchResult[]> = termObs$.pipe(switchMap(term => fetchValue(term))) // searchResult$ type is Observable<ISearchResult[]>  

```

Dans cette exemple nous imagons que `termObs$` écoute ce qui est tapé dans un champ de recherche.
Le "problème" est que le champ de recheche contient une chaine de caractères alors que mon API HTTP retourne une liste de résultats qui contiendront un titre et un résumé (Eg: `ISearchResult[]`)

Rappelez-vous: L'opérateur `map` permet de mapper un type en entrée d'observable à un nouveau type en sortie.

Les opérateurs vue ci-dessus ont également cette factulté, ils y ajoutent seulement des comportements spécifiques pour les opérations asynchrones.

C'est donc là toute la puissance que les opérateurs vuent ci-dessus interviennent ()




### Conclusion

**`concatMap`** :
- Accepte 1 seul argument : une fonction **callback**, qui doit renvoyer un `Observable`.
- Un seul **inner Observable** actif.
- Attend que l'**inner Observable** soit **complété** avant de souscrire au prochain **inner Observable**.
- Conserve l'odre d'émission des valeurs.

**`switchMap`**
- Accepte 1 seul argument : une fonction **callback**, qui doit renvoyer un `Observable`.
- Nombre illimité d'**inner Observable** actif.
- Si un **inner Observable** est **complété**, annule les autres **inner Observable** actifs.
- Ne conserve pas l'odre d'émission des valeurs.

**`mergeMap`**
- Accepte 2 arguments : une fonction **callback** qui doit renvoyer un `Observable` et le nombre d'**inner Observable** actif simultanément
- Nombre définis d'**inner Observable** actif.
- Ne conserve pas l'odre d'émission des valeurs.

#### Ressources

*  [concatMap](https://rxjs-dev.firebaseapp.com/api/operators/concatMap)
*  [switchMap](https://rxjs-dev.firebaseapp.com/api/operators/switchMap)
*  [mergeMap](https://rxjs-dev.firebaseapp.com/api/operators/mergeMap)

## Challenge

### À l'abordage des operateurs `*****Map`

Reprends le repos que tu as cloné ([ce repos](https://github.com/WildCodeSchool/RxJS_TDD)) dans les quêtes précedentes, ouvres le fichier `src/02.map.ts` et complétes chacun des observables afin de fair passer tous les tests.

Vous pouvez vérifier les tests grâce à la commande `npm run test-map` (Note: n'oubliez pas d'installer les dépendances)

### Critères de validation

- Passer tous les tests du fichier `src/02.map.ts`
- Poster le contenu de votre fichier `src/02.map.ts` dans un gist

