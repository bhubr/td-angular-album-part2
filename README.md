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
* de gérer un formulaire en utilisant les [formulaires réactifs](https://angular.io/guide/reactive-forms) d'Angular, et en particulier le [groupage de champs](https://angular.io/guide/reactive-forms#grouping-form-controls).
* d'envoyer les données du formulaire vers le _endpoint_ dédié de l'API Node.js qui vous est fournie.

### Génération du composant

```
ng g c register --skip-tests
```

### Ajout du code HTML dans le template

Le tutoriel ne propose pas de formulaire d'inscription. Aussi on ~~a copié-collé~~ s'est nettement inspiré du [formulaire de _login_](https://jasonwatmore.com/post/2019/06/22/angular-8-jwt-authentication-example-tutorial#login-component-html).

```html
<div class="col-md-6 offset-md-3 mt-5">
    <div class="alert alert-info">
        Username: test<br />
        Password: test
    </div>
    <div class="card">
        <h4 class="card-header">Angular 8 JWT Login Example</h4>
        <div class="card-body">
            <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" formControlName="username" class="form-control" [ngClass]="{ 'is-invalid': submitted && f.username.errors }" />
                    <div *ngIf="submitted && f.username.errors" class="invalid-feedback">
                        <div *ngIf="f.username.errors.required">Username is required</div>
                    </div>
                </div>
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" formControlName="password" class="form-control" [ngClass]="{ 'is-invalid': submitted && f.password.errors }" />
                    <div *ngIf="submitted && f.password.errors" class="invalid-feedback">
                        <div *ngIf="f.password.errors.required">Password is required</div>
                    </div>
                </div>
                <button [disabled]="loading" class="btn btn-primary">
                    <span *ngIf="loading" class="spinner-border spinner-border-sm mr-1"></span>
                    Login
                </button>
                <div *ngIf="error" class="alert alert-danger mt-3 mb-0">{{error}}</div>
            </form>
        </div>
    </div>
</div>
```