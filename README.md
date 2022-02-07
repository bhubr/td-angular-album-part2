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
