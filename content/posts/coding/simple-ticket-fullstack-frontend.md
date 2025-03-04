+++
title = "Building the Frontend of SimpleTicket with Angular"
date = "2022-02-02"
tags = ["coding", "angular", "web development", "tutorial"]
draft = false
+++

# Introduction

In this tutorial, we will guide you through building the frontend of the SimpleTicket web application using Angular. We will focus on the meaningful parts of the code, skipping over boilerplate and less relevant files. By the end of this tutorial, you will have a functional Angular application that interacts with a backend API to manage tickets.

## Project Structure

Here's the structure of the Angular project:

```
angular/
│
├── src/
│   ├── app/
│   │   ├── app-routing.module.ts
│   │   ├── app.component.html
│   │   ├── app.component.spec.ts
│   │   ├── app.component.ts
│   │   ├── app.module.ts
│   │   ├── auth/
│   │   │   ├── auth.interceptor.spec.ts
│   │   │   ├── auth.interceptor.ts
│   │   │   ├── auth.service.spec.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── login/
│   │   │   │   ├── login.component.html
│   │   │   │   ├── login.component.spec.ts
│   │   │   │   ├── login.component.ts
│   │   │   ├── signup/
│   │   │   │   ├── signup.component.html
│   │   │   │   ├── signup.component.spec.ts
│   │   │   │   ├── signup.component.ts
│   │   ├── components/
│   │   │   ├── home/
│   │   │   │   ├── home.component.html
│   │   │   │   ├── home.component.spec.ts
│   │   │   │   ├── home.component.ts
│   │   │   ├── navbar/
│   │   │   │   ├── navbar.component.html
│   │   │   │   ├── navbar.component.spec.ts
│   │   │   │   ├── navbar.component.ts
│   │   │   ├── ticket/
│   │   │   │   ├── ticket.component.html
│   │   │   │   ├── ticket.component.spec.ts
│   │   │   │   ├── ticket.component.ts
│   │   │   ├── ticket-all/
│   │   │   │   ├── ticket-all.component.html
│   │   │   │   ├── ticket-all.component.spec.ts
│   │   │   │   ├── ticket-all.component.ts
│   │   │   ├── ticket-form/
│   │   │   │   ├── ticket-form.component.html
│   │   │   │   ├── ticket-form.component.spec.ts
│   │   │   │   ├── ticket-form.component.ts
│   │   ├── modal/
│   │   │   ├── modal.component.html
│   │   │   ├── modal.component.spec.ts
│   │   │   ├── modal.component.ts
│   │   ├── models/
│   │   │   ├── ticket.ts
│   │   │   ├── user.ts
│   │   ├── services/
│   │   │   ├── error.interceptor.spec.ts
│   │   │   ├── error.interceptor.ts
│   │   │   ├── ticket.service.spec.ts
│   │   │   ├── ticket.service.ts
│   │   │   ├── user.service.spec.ts
│   │   │   ├── user.service.ts
│   ├── assets/
│   ├── environments/
│   │   ├── environment.prod.ts
│   │   ├── environment.ts
│   ├── index.html
│   ├── main.ts
│   ├── polyfills.ts
│   ├── test.ts
```

## Setting Up the Angular Application

First, let's set up the basic structure of the Angular application. We'll initialize the necessary modules and components.

### `main.ts`

The `main.ts` file is the entry point of the Angular application. It bootstraps the root module (`AppModule`) to start the application.

```typescript
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppModule)
  .catch(err => console.error(err));
```

### `app.module.ts`

The `AppModule` is the root module of the application. It declares the components, imports other modules, and provides services.

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HomeComponent } from './components/home/home.component';
import { TicketAllComponent } from './components/ticket-all/ticket-all.component';
import { TicketComponent } from './components/ticket/ticket.component';
import { NavbarComponent } from './components/navbar/navbar.component';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';
import { TicketFormComponent } from './components/ticket-form/ticket-form.component';

import { FormsModule, ReactiveFormsModule } from "@angular/forms";
import { ModalComponent } from './modal/modal.component';
import { LoginComponent } from './auth/login/login.component';
import { JwtModule } from "@auth0/angular-jwt";

import { AuthInterceptor } from './auth/auth.interceptor';
import { ErrorInterceptor } from './services/error.interceptor';
import { SignupComponent } from './auth/signup/signup.component';

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    TicketAllComponent,
    TicketComponent,
    NavbarComponent,
    TicketFormComponent,
    ModalComponent,
    LoginComponent,
    SignupComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
    NgbModule,
    FormsModule,
    ReactiveFormsModule,
    JwtModule.forRoot({
      config: {
        tokenGetter: () => localStorage.getItem('access_token')
      }
    })
  ],
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: ErrorInterceptor, multi: true },
  ],
  bootstrap: [AppComponent],
  entryComponents: [ModalComponent]
})
export class AppModule { }
```

### `app-routing.module.ts`

The `AppRoutingModule` defines the routes for the application. Each route maps a URL path to a component.

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { TicketAllComponent } from './components/ticket-all/ticket-all.component';
import { TicketFormComponent } from './components/ticket-form/ticket-form.component';
import { TicketComponent } from './components/ticket/ticket.component';
import { LoginComponent } from './auth/login/login.component';
import { SignupComponent } from './auth/signup/signup.component';

const routes: Routes = [
  { path :  '', component : HomeComponent},
  { path : 'ticketall', component : TicketAllComponent},
  { path : 'ticketcreate', component: TicketFormComponent},
  { path : 'ticketcreate/:ticket:edit', component: TicketFormComponent},
  { path : 'ticketdetail', component: TicketComponent},
  { path: 'login', component: LoginComponent},
  { path: 'signup', component: SignupComponent}
];

@NgModule({
  imports: [RouterModule.forRoot(routes), RouterModule.forRoot(
    routes,
    { enableTracing: false }
  )],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

## Authentication

The authentication module handles user login, signup, and token management.

### `auth.service.ts`

The `AuthService` manages authentication-related operations, such as login, logout, and token storage.

```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import * as moment from 'moment';
import { User } from '../models/user';
import { JwtHelperService } from '@auth0/angular-jwt';
import { BehaviorSubject, Observable, tap } from 'rxjs';
import { Router } from '@angular/router';
import { UserService } from '../services/user.service';

export interface Token {
  access: string,
  refresh: string
}

export interface TokenPayload {
  token_type: string
  exp: string
  jti: string
  iat: string
  user_id: string
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  isLoggedInSubject = new BehaviorSubject<boolean>(this.isTokenValid());

  constructor(private http: HttpClient,
    private jwt: JwtHelperService,
    private router: Router,
    private userService: UserService) {
  }

  login(username: string | undefined | null, password: string | undefined | null) {
    return this.http.post<Token>('http://localhost:8000/api/token/', { username, password })
      .subscribe(
        res => {
          this.setSession(res)
        }
      );
  }

  private setSession(token: Token) {
    const payload = this.jwt.decodeToken(token.access)
    const expiresAt = moment().add(payload.exp, 'second');
    localStorage.setItem('access_token', token.access);
    localStorage.setItem("expires_at", JSON.stringify(expiresAt.valueOf()));
    this.userService.getUserById(payload.user_id).subscribe(
      (data) => {
        localStorage.setItem("logged_user", JSON.stringify(data))
      }
    )
    this.isLoggedInSubject.next(true)
    this.router.navigate(['/ticketall'])
  }

  logout() {
    localStorage.removeItem("access_token");
    localStorage.removeItem("expires_at");
    this.isLoggedInSubject.next(false)
  }

  public isTokenValid() {
    return moment().isBefore(this.getExpiration());
  }

  isLoggedOut() {
    return !this.isLoggedIn();
  }

  getExpiration() {
    const expiration = localStorage.getItem("expires_at");
    const expiresAt = expiration ? JSON.parse(expiration) : "";
    return moment(expiresAt);
  }

  isLoggedIn(): Observable<boolean> {
    return this.isLoggedInSubject.asObservable();
  }

  getLoggedInUser() {
    const user = localStorage.getItem("logged_user")
    const parsedUser = user ? JSON.parse(user) : new Object
    return parsedUser
  }

  public signup(email: string | undefined | null, username: string | undefined | null, password: string | undefined | null, password2: string | undefined | null) {
    return this.http.post<Object>('http://localhost:8000/register', { email, username, password, password2 })
    .pipe(
      tap ( (res) => {
      }
      )
    )
  }
}
```

### `auth.interceptor.ts`

The `AuthInterceptor` adds the authorization token to the headers of outgoing HTTP requests.

```typescript
import { Injectable } from '@angular/core';
import { AuthService } from './auth.service';
import { Router } from "@angular/router"

import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor
} from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class AuthInterceptor implements HttpInterceptor {

  constructor(private authService: AuthService, private router: Router) { }

  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const isTokenValid = this.authService.isTokenValid()

    if (isTokenValid) {
      const clonedReq = req.clone({
        headers: req.headers.set("Authorization",
          "Bearer " + localStorage.getItem("access_token"))
      });

      return next.handle(clonedReq);
    }
    else {
      return next.handle(req);
    }
  }
}
```

## Components

The components are the building blocks of the Angular application. Each component handles a specific part of the UI and its logic.

### `navbar.component.ts`

The `NavbarComponent` displays the navigation bar with links to different parts of the application.

```typescript
import { Component, OnInit } from '@angular/core';
import { Ticket } from 'src/app/models/ticket';
import { TicketService } from 'src/app/services/ticket.service';
import { AuthService } from 'src/app/auth/auth.service';
import { Observable } from 'rxjs';

@Component({
  selector: 'app-navbar',
  templateUrl: './navbar.component.html',
  styleUrls: ['./navbar.component.scss']
})
export class NavbarComponent implements OnInit {
  isLoggedIn : Observable<boolean>;
  isCollapsed: Boolean = true;
  constructor(public authService : AuthService ) {
    this.isLoggedIn = authService.isLoggedIn();
  }

  ngOnInit(): void { }
}
```

### `ticket-all.component.ts`

The `TicketAllComponent` displays a list of all tickets. It fetches the tickets from the backend and allows the user to view, edit, or delete each ticket.

```typescript
import { Component, OnInit } from '@angular/core';
import { TicketService } from 'src/app/services/ticket.service';
import { Router } from '@angular/router';
import { Ticket } from './../../models/ticket';
import { AuthService } from 'src/app/auth/auth.service';
import { BehaviorSubject, tap } from 'rxjs';

@Component({
  selector: 'app-ticket-all',
  templateUrl: './ticket-all.component.html',
  styleUrls: ['./ticket-all.component.scss']
})

export class TicketAllComponent implements OnInit {

  isLoaded: BehaviorSubject<boolean> = new BehaviorSubject(false)
  tickets!: Ticket[]
  ticket!: Ticket

  constructor(
    private api: TicketService,
    private router: Router,
    private authService: AuthService) {
  }

  ngOnInit(): void {
    if (!this.authService.isTokenValid()) {
      this.router.navigate(['/login'])
    }

    this.loadTicketData()
    this.api.onAddedTicket
      .pipe(
        tap(
          () => {
            this.loadTicketData()
          }
        )
      )
  }

  deleteTicket(id: number) {
    if (!id) return console.log('Ticket ' + id + ' not found')
    if (confirm('Are you sure to delete ticket ' + id + ' ?') == true)
      this.api.deleteTicket(id)
        .subscribe(
            () => {
              this.router.navigate(["/ticketall"])
              this.loadTicketData()
            }
        )
  }

  loadTicketData() {
    this.api.getTickets()
      .pipe(
        tap(
          (tickets: Ticket[]) => {
            this.tickets = tickets
            this.tickets = this.tickets.filter(
              ticket =>
              (ticket.createdBy || ticket.requestedBy || ticket.requestedFor ) === this.authService.getLoggedInUser().url)
            this.isLoaded.next(true)
          }
        )
      )
      .subscribe(() => this.isLoaded.next(true))
  }
}
```

### `ticket-form.component.ts`

The `TicketFormComponent` handles the creation and editing of tickets. It includes a form for entering ticket details and submitting the data to the backend.

```typescript
import { Component, OnInit } from '@angular/core';
import { TicketService } from 'src/app/services/ticket.service';
import { UserService } from 'src/app/services/user.service';
import { Ticket } from 'src/app/models/ticket';
import { User } from 'src/app/models/user';
import { NgForm } from '@angular/forms';

import { BehaviorSubject, Observable, OperatorFunction } from 'rxjs';
import { debounceTime, distinctUntilChanged, map } from 'rxjs/operators';
import { ActivatedRoute, Router } from '@angular/router';
import { AuthService } from 'src/app/auth/auth.service';

@Component({
  selector: 'app-ticket-create',
  templateUrl: './ticket-form.component.html',
  styleUrls: ['./ticket-form.component.scss']
})

export class TicketFormComponent implements OnInit {

  users!: User[];
  ticketModel!: Ticket
  editMode: boolean = false
  id: number = 0
  isLoaded: BehaviorSubject<boolean> = new BehaviorSubject(false)

  constructor(
    private userService: UserService,
    private ticketApi: TicketService,
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private authService: AuthService
  ) { }

  ngOnInit(): void {
    if (!this.authService.isTokenValid()) {
      this.router.navigate(['/login'])
    }
    this.userService.getUsers().subscribe(data => this.users = data)
    this.activatedRoute.queryParams
      .subscribe(
        params => {
          if (params["edit"] === 'true') {
            this.editMode = true
            this.id = params["id"]
            this.loadTicketData()
          } else {
            this.editMode = false
            this.ticketModel = {
              url: "",
              title: "",
              description: "",
              completed: false,
            };
            this.isLoaded.next(true)
          }
        }
      )
  }

  loadTicketData() {
    this.userService.getUsers()
      .subscribe((data: User[]) => {
        this.users = data
        this.ticketApi.getTicket(this.id).subscribe(data => {
          this.ticketModel = data
          this.ticketModel.createdBy = this.users.filter(user => user.url === this.ticketModel.createdBy)[0]
          this.ticketModel.requestedBy = this.users.filter(user => user.url === this.ticketModel.requestedBy)[0]
          this.ticketModel.requestedFor = this.users.filter(user => user.url === this.ticketModel.requestedFor)[0]
          this.isLoaded.next(true)
        }
        )
      });
  }

  addTicket(form: NgForm) {
    form.value["createdBy"] = form.value["createdBy"]["url"];
    form.value["requestedBy"] = form.value["requestedBy"]["url"];
    form.value["requestedFor"] = form.value["requestedFor"]["url"];
    form.value["completed"] = false // set default

    this.ticketApi.addTicket(form.value).subscribe(
      () => {
        this.ticketApi.onAddedTicket.next(this.ticketModel);
      }
    )
    form.reset()
    this.router.navigate(['/ticketall'])
  }

  updateTicket(form: NgForm) {
    form.value["createdBy"] = form.value["createdBy"]["url"];
    form.value["requestedBy"] = form.value["requestedBy"]["url"];
    form.value["requestedFor"] = form.value["requestedFor"]["url"];

    this.ticketApi.updateTicket(this.id, form.value).subscribe(
      () => {
        this.ticketApi.onAddedTicket.next(this.ticketModel)
      }
    )
    form.reset()
    this.router.navigate(['/ticketall'])
  }

  searchUser: OperatorFunction<string, readonly User[]> = (text$: Observable<string>) =>
    text$.pipe(
      debounceTime(200),
      distinctUntilChanged(),
      map((term) =>
        term.length < 1 ? [] : this.users.filter(
          (user) => user.username.toLowerCase().indexOf(term.toLowerCase()) > -1)
          .slice(0, 10),
      ),
    );

  userResultFormatter = (user: { email: string }) => user.email;
  userInputFormatter = (user: { email: string }) => user.email;
}
```

## Services

Services in Angular are used to handle business logic and data fetching. They are injectable classes that can be used across different components.

### `ticket.service.ts`

The `TicketService` handles CRUD operations for tickets. It communicates with the backend API to fetch, create, update, and delete tickets.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Ticket } from '../models/ticket';
import { Observable, Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})

export class TicketService {

  baseUrl: string = "http://localhost:8000"
  onAddedTicket = new Subject<Ticket>();

  constructor(private http: HttpClient) { }

  getTickets() {
    return this.http.get<Ticket[]>(`${this.baseUrl}/tickets/`)
  }

  getTicket(ticketId: number): Observable<Ticket> {
    return this.http.get<Ticket>(`${this.baseUrl}/tickets/${ticketId}/`)
  }

  addTicket(ticket:Ticket) {
    return this.http.post<Ticket>(`${this.baseUrl}/tickets/`, ticket)
  }

  deleteTicket(ticketId: number) {
    return this.http.delete<Ticket>(`${this.baseUrl}/tickets/${ticketId}/`)
  }

  updateTicket(ticketId: number, ticket: Ticket ){
    return this.http.put<Ticket>(`${this.baseUrl}/tickets/${ticketId}/`, ticket)
  }
}
```

## Conclusion

You've now created a functional Angular application for managing tickets. This application includes authentication, CRUD operations for tickets, and a user-friendly interface. You can further enhance the application by adding more features or improving the existing ones.

### Resources

- Checkout the code for this tutorial [on my GitHub](https://github.com/raphael2692/simpleticket)