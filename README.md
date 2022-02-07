# TD Angular "Simple Album" - partie 2

Ce support prend la suite des slides de la partie 1, qui sont hébergés :

* [Sur mon propre serveur](https://td-angular-part1.bhu.ovh/)
* [Sur Netlify](https://jovial-nightingale-b4e978.netlify.app/) (en cas de panne de mon serveur 😅)

Le fait d'avoir un support distinct pour cette 2ème partie est dû :

* Au fait qu'on va ici aborder des sujets plus avancés (que je n'avais pas le temps d'aborder avec tous mes apprenants),
* À la difficulté de naviguer dans un support contenant plus de 150 slides,
* Et enfin, aux problèmes rencontrés avec le format spécifique de slides utilisé pour la partie 1.

Cette seconde partie vise à mettre en place l'authentification sur l'application "Angular Simple Album".

C'est un prétexte pour aborder des notions d'Angular plus avancées. Entre autres :

* Observables
* Guards
* Interceptors

## Plan

Le support s'inspire largement de ce tutoriel [Angular 8 - JWT Authentication Example & Tutorial](https://jasonwatmore.com/post/2019/06/22/angular-8-jwt-authentication-example-tutorial). On ne le suit cependant pas à la lettre, notamment du fait de petites évolutions d'Angular, entre la version 8 utilisée dans le tutoriel, et la version 12 utilisée pour mettre en place notre application.

Nous verrons les choses dans cet ordre :

* Mise en place du formulaire d'inscription d'un nouvel utilisateur

## _Register_ - Inscription d'un utilisateur

Ici, il va s'agir :

* de générer un nouveau composant,
* de gérer un formulaire en utilisant les [formulaires réactifs](https://angular.io/guide/reactive-forms) d'Angular, et en particulier le [groupage de champs](https://angular.io/guide/reactive-forms#grouping-form-controls) et [l'utilisation du service FormBuilder](https://angular.io/guide/reactive-forms#using-the-formbuilder-service-to-generate-controls).
* d'envoyer les données du formulaire vers le _endpoint_ dédié de l'API Node.js qui vous est fournie.

### Génération du composant

```
ng g c register --skip-tests
```

### Ajout du code HTML dans le template

Le tutoriel ne propose pas de formulaire d'inscription. Aussi on ~~a copié-collé~~ s'est nettement inspiré du [formulaire de _login_](https://jasonwatmore.com/post/2019/06/22/angular-8-jwt-authentication-example-tutorial#login-component-html).

Dans un premier temps, afin d'aller à l'essentiel, on l'a simplifié en supprimant certains éléments.

> **Note** : on a "oublié" (pour aller vite) de supprimer les nombreux attributs `class` des balises. Les classes référencées ici sont des classes de la bibliothèque CSS [Bootstrap](https://getbootstrap.com). On aurait pu l'ajouter, avec le risque que cela entre en conflit avec les styles existants dans notre app.

Voici notre version simplifiée, à coller dans `register.component.html` pour remplacer le contenu initial :

```html
<div class="col-md-6 offset-md-3 mt-5">
  <div class="alert alert-info">
      Username: test<br />
      Password: test
  </div>
  <div class="card">
      <h4 class="card-header">Register</h4>
      <div class="card-body">
          <form [formGroup]="registerForm" (ngSubmit)="onSubmit()">
              <div class="form-group">
                  <label for="username">Username</label>
                  <input type="text" formControlName="username" class="form-control" />

              </div>
              <div class="form-group">
                  <label for="password">Password</label>
                  <input type="password" formControlName="password" class="form-control" />

              </div>
              <button [disabled]="loading" class="btn btn-primary">
                  <span *ngIf="loading" class="spinner-border spinner-border-sm mr-1"></span>
                  Register
              </button>
              <div *ngIf="error" class="alert alert-danger mt-3 mb-0">{{error}}</div>
          </form>
      </div>
  </div>
</div>
```

En l'état, cette modification fait planter le "rebuild" continu de l'application. On écope de pas moins de 6 erreurs, qu'on peut ramener à l'absence des propriétés suivantes dans le pendant TypeScript du composant : `registerForm`, `onSubmit`, `loading`, `error`.

### Partie TypeScript

Une brève explication du rôle de ces propriétés, dont l'absence pose problème :

* `registerForm` : instance de `FormGroup`, générée via la méthode `group` du service `FormBuilder`. En français, c'est un objet dédié au stockage des données du formulaire.
* `onSubmit` : méthode appelée lors de la soumission du formulaire.
* `loading` : booléen indiquant si on est - ou non - en train de soumettre le formulaire. On s'en servira pour, par exemple, _empêcher de cliquer une seconde fois_ sur le bouton _Register_, si on est déjà en train d'envoyer les données au serveur (binding d'attribut `[disabled]="loading"` sur le `button`).
* `erreur` : chaîne initialement vide, indiquant si une erreur s'est produite.

> **Note** Dans `app.module.ts`, on devra également importer `ReactiveFormsModule` depuis `'@angular/forms'`, et ajouter `ReactiveFormsModule` dans le tableau `imports: [...]`.

Voici maintenant le code - provisoire - à ajouter pour au moins régler ces erreurs.

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-register',
  templateUrl: './register.component.html',
  styleUrls: ['./register.component.css']
})
export class RegisterComponent implements OnInit {
  registerForm: FormGroup;
  loading = false;
  error = '';

  constructor(private formBuilder: FormBuilder) {
    this.registerForm = this.formBuilder.group({
      username: ['', Validators.required],
      password: ['', Validators.required]
    });
  }

  ngOnInit(): void {
  }

  onSubmit() {}

}
```

### Routing vers le composant Register

Dans `app-routing.module.ts`, référencer une nouvelle route :

```typescript
// ...
import { RegisterComponent } from './register/register.component';
// ...
const routes: Routes = [
  { path: '', component: AlbumComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: 'details/:id', component: DetailsComponent },
  { path: 'add-post', component: AddPostComponent },
  { path: 'register', component: RegisterComponent },
];

// ...
```

Dans `navbar.component.html`, ajouter à la liste de liens :

```html
<li><a routerLink="/register">Register</a></li>
```

### Envoi des données vers l'API

On a déjà créé un service, `PostService` était dédié à l'envoi de requêtes HTTP vers l'API. Plus précisément, des requêtes concernant les _posts_. Afin de ne pas le "surcharger", on va créer un service dédié à l'authentification.

```
ng g s authentication --skip-tests
```

Dans ce service, on va :

* Importer le `HttpClient` d'Angular, et le référencer dans les paramètres du constructeur.
* Définir des attributs pour l'URL de base de l'API (`serverUrl`) et le chemin relatif vers le _endpoint_ register (`registerPath`). Notez qu'avoir une valeur hardcodée pour `serverUrl` n'est pas une bonne pratique (il vaut mieux à cette fin utiliser les [environnements](https://www.softfluent.fr/blog/angular-6-bien-demarrer-lutilisation-parametres-configuration-dans-environnements/)).
* Implémenter une méthode `register` qui va envoyer les données du formulaire d'inscription (un objet contenant des champs `login` et `pwd`) vers le _endpoint_ register.

Voici le code TypeScript correspondant, à ce stade :

```typescript
// src/app/authentication.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class AuthenticationService {
  // URL absolue
  serverUrl = 'https://album-api.benoithubert.me';
  // chemin relatif sur le serveur
  registerPath = '/api/v2/auth/register';

  constructor(private http: HttpClient) { }

  public register(login: string, pwd: string): Observable<{}> {
    return this.http.post<{}>(
      `${this.serverUrl}${this.registerPath}`,
      { login, pwd }
    );
  }
}
```

> Note : la méthode `register` renvoie non pas une `Promise` (comme on l'avait vu dans `PostService`), mais un `Observable`. On va expliquer le tout un peu plus loin, après avoir vu comment la méthode `register` est appelée depuis `RegisterComponent`.

Dans `register.component.service`, on va :

* importer le nouveau service,
* le référencer dans les parenthèses du constructeur,
* appeler sa méthode `register` depuis `onSubmit`

Import du service :

```typescript
import { AuthenticationService } from '../authentication.service';
```

Ajout dans les parenthèses du constructeur :

```typescript
  constructor(private formBuilder: FormBuilder, private authenticationService: AuthenticationService) {
    // ...
  }
```

Appel de `register` :

```typescript
  onSubmit() {
      // Récupération des valeurs (username/email et password) par décomposition
      const { username, password } = this.registerForm.value;
      this.authenticationService.register(username, password)
        .subscribe(newUser => {
          console.log(newUser);
        })
    }
```

**Note** :

* On récupère les valeurs `username` et `password` par décomposition : [MDN](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) et [article](https://mindsers.blog/fr/post/decomposition-et-destructuration-en-javascript/) (section _Décomposer des objets_).
* Si `register` renvoyait une `Promise`, on aurait "chaîné" un `.then` derrière l'appel. Comme elle renvoie un `Observable`, on utilise `.subscribe` à la place. On détaille cela dans la section suivante, "Introduction aux Observables".

**Après ces modifications**, l'inscription d'un utilisateur est censée fonctionner, en tenant compte de ces contraintes :

* Dans le champ "Username", il faut saisir **une adresse e-mail**.
* Le champ "Password" doit contenir **au moins 5 caractères**.

L'application serveur renverra une erreur si l'une de ces deux contraintes n'est pas satisfaite.

## Introduction aux Observables

Les `Observable`s offrent une façon unifiée de gérer des flux de données, que celles-ci soient obtenues de façon **synchrone** ou **asynchrone**. Pour voir ce que cela signifie, on va surtout se pencher sur l'aspect asynchrone.

JavaScript a été, à l'origine, créé pour ajouter de l'interactivité aux pages web. Il a donc intégré, dès le début, des mécanismes de gestion d'évènements. Ceux-ci fonctionnent de façon asynchrone. Par exemple, une certaine fonction (_callback_) va être appelée :

* lorsqu'on clique sur un bouton,
* ou toutes les _x_ secondes,
* ou suite au retour d'une requête HTTP

### Asynchrone classique

Nous allons voir trois exemples de code asynchrone, correspondant à ces trois cas de figure.

Ce premier exemple permet de gérer l'évènement `click` sur un bouton :

```javascript
const btn = document.getElementById('btn');

// Lorsqu'un clic est effectué, la fonction passée comme
// 2nd paramètre à addEventListener va être appelée
btn.addEventListener('click', () => {
  console.log('Le bouton a été cliqué');
});
```

Ce deuxième exemple montre l'appel d'une fonction toutes les 2 secondes :

```javascript
// Cette fois, la fonction à appeler est le 1er paramètre,
// le 2nd étant l'intervalle de temps en millisecondes
setInterval(() => {
  console.log('Message affiché toutes les 2 secondes');
}, 2000);
```

Ce troisième exemple montre l'appel d'une fonction suite au retour d'une requête HTTP. Elle utilise l'API "Fetch" disponible dans tous les navigateurs modernes.

```javascript
fetch('https://api.github.com/users')
  .then(res => res.json())
  .then(data => console.log('Utilisateurs GitHub', data));
```

Ces trois exemples sont visibles [sur CodeSandbox](https://codesandbox.io/s/javascript-asynchrone-3mh5k) - **ouvrez la console** via le bouton situé juste sous le panneau d'affichage, en bas à droite de l'écran.

Les deux premiers exemples ont un point commun : la fonction callback peut être appelée plusieurs fois.

Dans le 3ème exemple, les fonctions passées dans `.then()` ne seront appelée qu'une fois. Un autre point qui distingue ce 3ème exemple est l'utilisation de "promesses" (l'appel `fetch` renvoie une Promise).

Ce qu'il faut en retenir, c'est qu'on a différentes façons de gérer l'appel du callback.

### Asynchrone avec observables

Les observables permettent d'avoir une écriture commune pour gérer l'asynchronisme.

Dans Angular comme dans les exemples qui suivent, les observables sont fournis par la bibliothèque [RxJS](https://rxjs.dev/), dont des déclinaisons existent dans de nombreux autres langages (Java, C#, etc.).

Réécrivons le 1er exemple avec RxJS, et plus particulièrement `fromEvent`, qui crée un observable à partir d'une certaine source d'évènements (un élément du DOM dans le navigateur, un `EventEmitter` dans Node.js, etc.) :

```javascript
import { fromEvent } from 'rxjs';

const btn = document.getElementById('btn');
const btnClickObs = fromEvent(btn, 'click');
btnClickObs.subscribe(() => {
  console.log('Le bouton a été cliqué');
});
```

Un observable _émet_ des valeurs. On peut appeler la méthode `subscribe` d'un observable, et lui passer un callback. Ce callback sera appelé à chaque fois que l'observable émet une valeur.

**Note** : un observable n'émet des valeurs **que** s'il y existe au moins un "abonnement" à cet observable, effectué via `subscribe`.

Réécrivons le second exemple, en utilisant la fonction `interval` fournie par RxJS :

```javascript
import { interval } from 'rxjs';

const intvObs = interval(2000);
intvObs.subscribe((n) => {
  console.log('Message affiché toutes les 2 secondes', n);
});
```

Il y a une petite différence avec `setInterval` : l'observable créé via `interval` émet une séquence de nombres (`n`) démarrant à zéro.

Enfin, réécrivons le 3ème exemple, cette fois en créant nous-mêmes un `Observable` :

```javascript
import { Observable } from 'rxjs';

const httpObs = new Observable(subscriber => {
  fetch('https://api.github.com/users')
    .then(res => res.json())
    .then(data => subscriber.next(data));
});
httpObs.subscribe((data) => {
  console.log('Utilisateurs GitHub', data);
});
```

Ces trois exemples sont accessibles [sur CodeSandbox](https://codesandbox.io/s/javascript-rxjs-observables-vs2pi).

Qu'est-ce qui change par rapport aux versions initiales ?

Dans tous les cas, on a maintenant des observables (`btnClickObs`, `intvObs` et `httpObs`) sur lesquels on appelle une méthode `subscribe`. On passe à cette méthode `subscribe` un callback, qui est appelé quand l'observable émet une donnée.

Ce qu'il faut en retenir, c'est que les observables permettent de gérer les évènements de façon unifiée, quelle que soit la source de ces évènements.

Les observables permettent également d'effectuer des traitements, entre l'émission de la donnée par l'observable, et sa récupération dans le callback (ce qu'on n'a pas fait ici).

