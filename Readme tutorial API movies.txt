Hola mijines el dia de hoy vamos a crear una
aplicacion API utilizando ionic 4 que va a 
leer de un servidor de peliculas abierto.

Para esto vamos a instalar todos los componentes que 
necesarios desde 0. Desde el node packet manager hasta
el emulador de android. Si ya tienes todo instalado puedes
adelantar el video a la parte del codigo. 

Ionic es un framework open source para 
desarrollar aplicaciones híbridas multiplataforma que
utiliza HTML5, CSS y Cordova como base. 

para este tutorial vamos a correr la aplicacion en el
emulador de android studio.

Instalar gradle, instalar jdk de java!

de manera global

los requerimientos son 

Para instalar ionic 

node.js instalado

npm gestor de paquetes

comprobar la version de node

node --version

npm --version

npm install -g ionic

------------------------------------------------
Tutorial online
https://ionicacademy.com/ionic-4-app-api-calls/
------------------------------------------------

ionic start movieApp blank --type=angular
cd movieApp

con este se crea las paginas que vamos a utilizar

ionic g page pages/movies
ionic g page pages/movieDetails
ionic g service services/movie

revisamos que este funcionando
cd movieApp
ionic serve

------------------------------------------
ABRIR LA CARPETA EN SUBLIME 

DESCRIPCION DE CARPETAS
-----------------------------------------
En la carpeta encontramos la carpeta APP
Aqui se tiene todo el codigo

Tiene la carpeta home 
que la podemos borrar la carpeta home por que 
ya se crearon las otras carpetas con el comando previo

dentro de cada pagina encontramos 4 archivos
*.module.ts el modulo de angular con todo lo relacionado a imports y estilo
*.page.html: The HTML markup for a page
*.page.scss: The styling for the specific page (more on global styling later)
*.page.spec.ts:  testing file for your page. para unit tests
*.page.ts: The controller for a page that contains the 
Javascript code that manages the functionality


---------------------------------------------------
PRIMER CAMBIO
---------------------------------------------------

en app/app.module.ts

import { HttpClientModule } from '@angular/common/http';

dentro del NgModule

imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule, HttpClientModule],

 
---------------------------------------------------
SEGUNDO CAMBIO
---------------------------------------------------

app/app-routing.module.ts
y buscar app-routing.module ts 
para cambiar las rutas

import { NgModule } from '@angular/core';
import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: 'movies', pathMatch: 'full' },
borrar el home

  { path: 'movies', loadChildren: './pages/movies/movies.module#MoviesPageModule' },
  { path: 'movies/:id', loadChildren: './pages/movie-details/movie-details.module#MovieDetailsPageModule' },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule { }

--------------------------------------------------------------

cd movieApp 
ionic serve

---------------------------------------------------------

Ir a http://www.omdbapi.com/apikey.aspx

para activar su key

Here is your key: 86f73c9a

------------------------------------------------


Ir a services/movie.service.ts

aqui se crea la interaccion con la API, en este caso de las peliculas

-------------------------------------------

Para las peticiones HTTP

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
 
export enum SearchType {
  all = '',
  movie = 'movie',
  series = 'series',
  episode = 'episode'
}
 
@Injectable({
  providedIn: 'root'
})
export class MovieService {
  url = 'http://www.omdbapi.com/';
  apiKey = ''; // <-- Enter your own key here!
 
  constructor(private http: HttpClient) { }
 
  searchData(title: string, type: SearchType): Observable<any> {
    return this.http.get(`${this.url}?s=${encodeURI(title)}&type=${type}&apikey=${this.apiKey}`).pipe(
      map(results => results['Search'])
    );
  }
 
  /**
  * Get the detailed information for an ID using the "i" parameter
  * 
  * @param {string} id imdbID to retrieve information
  * @returns Observable with detailed information
  */
  getDetails(id) {
    return this.http.get(`${this.url}?i=${id}&plot=full&apikey=${this.apiKey}`);
  }
}

-----------------------------------------------------

Ahora para la parte del controlador
navegamos a la pagina de :

pages/movies/movies.page.ts:

-----------------------------------------------------

import { MovieService, SearchType } from './../../services/movie.service';
import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs';
 
@Component({
  selector: 'app-movies',
  templateUrl: './movies.page.html',
  styleUrls: ['./movies.page.scss'],
})
export class MoviesPage implements OnInit {
 
  results: Observable<any>;
  searchTerm: string = '';
  type: SearchType = SearchType.all;
 
  /**
   * Constructor of our first page
   * @param movieService The movie Service to get data
   */
  constructor(private movieService: MovieService) { }
 
  ngOnInit() { }
 
  searchChanged() {
    // Call our service function which returns an Observable
    this.results = this.movieService.searchData(this.searchTerm, this.type);
  }
}


--------------------------------------------

navegamos a la pagina html 

pages/movies/movies.page.html

--------------------------------------------

<ion-header>
  <ion-toolbar color="primary">
    <ion-title>Movie Search</ion-title>
  </ion-toolbar>
</ion-header>
 
<ion-content>
 
  <ion-searchbar [(ngModel)]="searchTerm" (ionChange)="searchChanged($event)"></ion-searchbar>
 
  <ion-item>
    <ion-label>Select Searchtype</ion-label>
    <ion-select [(ngModel)]="type" (ionChange)="searchChanged($event)">
      <ion-select-option value="">All</ion-select-option>
      <ion-select-option value="movie">Movie</ion-select-option>
      <ion-select-option value="series">Series</ion-select-option>
      <ion-select-option value="episode">Episode</ion-select-option>
    </ion-select>
  </ion-item>
 
  <ion-list>
 
    <ion-item button *ngFor="let item of (results | async)" [routerLink]="['/', 'movies', item.imdbID]">
      <ion-avatar slot="start">
        <img [src]="item.Poster" *ngIf="item.Poster != 'N/A'">
      </ion-avatar>
 
      <ion-label text-wrap>
        <h3>{{ item.Title }}</h3>
        {{ item.Year }}
      </ion-label>
 
      <ion-icon slot="end" *ngIf="item.Type == 'movie'" name="videocam"></ion-icon>
      <ion-icon slot="end" *ngIf="item.Type == 'series'" name="tv"></ion-icon>
      <ion-icon slot="end" *ngIf="item.Type == 'game'" name="logo-game-controller-b"></ion-icon>
 
    </ion-item>
 
  </ion-list>
 
</ion-content>


ionic serve y revisar todo

que funcione la funcion de buscar

----------------------------------------------------

pages/movie-details/movie-details.page.ts:


import { MovieService } from './../../services/movie.service';
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
 
@Component({
  selector: 'app-movie-details',
  templateUrl: './movie-details.page.html',
  styleUrls: ['./movie-details.page.scss'],
})
export class MovieDetailsPage implements OnInit {
 
  information = null;
 
  /**
   * Constructor of our details page
   * @param activatedRoute Information about the route we are on
   * @param movieService The movie Service to get data
   */
  constructor(private activatedRoute: ActivatedRoute, private movieService: MovieService) { }
 
  ngOnInit() {
    // Get the ID that was passed with the URL
    let id = this.activatedRoute.snapshot.paramMap.get('id');
 
    // Get the information from the API
    this.movieService.getDetails(id).subscribe(result => {
      this.information = result;
    });
  }
 
  openWebsite() {
    window.open(this.information.Website, '_blank');
  }
}

---------------------------------------------------

Utilizando los ionic card components se crean las imagenes y otros
items con la informacion

Ahora solo toca cambiar los detalles en

pages/movie-details/movie-details.page.html to:

<ion-header>
  <ion-toolbar color="primary">
    <ion-buttons slot="start">
      <ion-back-button defaultHref="/"></ion-back-button>
    </ion-buttons>
    <ion-title>{{ information?.Genre }}</ion-title>
  </ion-toolbar>
</ion-header>
 
<ion-content padding>
 
  <ion-card *ngIf="information">
    <ion-card-header>
      <ion-card-title>
        {{ information.Title }}
      </ion-card-title>
      <ion-card-subtitle>
        {{ information.Year }}
      </ion-card-subtitle>
    </ion-card-header>
    <ion-card-content text-center>
      <img [src]="information.Poster" class="info-img">
      {{ information.Plot }}
 
      <ion-item lines="none">
        <ion-icon name="star-half" slot="start"></ion-icon>
        <ion-label>{{ information.imdbRating }}</ion-label>
      </ion-item>
 
      <ion-item lines="none">
        <ion-icon name="clipboard" slot="start"></ion-icon>
        <ion-label text-wrap>{{ information.Director }}</ion-label>
      </ion-item>
 
      <ion-item lines="none">
        <ion-icon name="contacts" slot="start"></ion-icon>
        <ion-label text-wrap>{{ information.Actors }}</ion-label>
      </ion-item>
 
      <ion-button expand="full" (click)="openWebsite()" *ngIf="information.Website && information.Website != 'N/A'">
        <ion-icon name="open" slot="start"></ion-icon>
        Open Website
      </ion-button>
    </ion-card-content>
  </ion-card>
  
</ion-content>

----------------------------------------------
ionic serve 



en el scss

pages/movie-details/movie-details.page.scss

.info-img {
    max-height: 30vh;
    object-fit: contain;
    padding: 10px;
}


