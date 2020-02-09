---
layout: page-fullwidth
title:  "Cummunication Between Components In Angular"
categories:
    - angular
tags: 
    - angular
    - Cummunication
    - Data Transfer
    - Component Interaction
    - Pass Data
header: no
breadcrumb: true
meta_description: "Cummunication Between Components In Angular"
author: Hanish Totlani
---

In this post, I am going to share how to pass data between components and which is the best way depending upon the scenario.

## 1. Pass Data From URL;

Consider that we are navigating from one page to another in which the previous page is destroyed and we are landing on another page. Then if the data is small (eg. id of an object) then we can use url to pass the data.

There are two ways to pass the data through url in angular.
* Router Parameters.
* Query Params

### Router Parameters

Router parameters are required parameters. If the parameter is mandatory for the component then only we have to use **router parameter**. Other wise we can use **query params**. We have to register the parameter with the url in the router module like,

<p class="file-desc"><span>app-router.module.ts</span></p>

```typescript
    const routes: Routes = [
  { path: 'list/:id', component: AppListComponent}
];
```

Here _list_ is the route url and *:id* is the router param which is mandatory to pass and *AppListComponent* is the component.

#### Pass router param through routerLink

```html
    <button type="button" [routerLink]="['/list', id]">Show List</button>
```

Here *id* is the variable initialized in *.ts* file and the */list* is the route on which we want to navigate.

#### Pass router param through Route Service

<p class="file-desc"><span>app.component.ts</span></p>

```typescript
    id = 28;
    constructor (private router: Router) {
    }

    route() {
        this.router.navigate(['/list', this.id]);
    }
```

#### For reading router params

<p class="file-desc"><span>app-list.component.ts</span></p>

```typescript
    constructor(
        private activatedroute: ActivatedRoute
    ) {
        this.activatedroute.params.subscribe(data => {
            console.log(data);
        })
    }
```

For more detail on router params you can visit [this]()


### Query Params

Query params are optional params. There is no need to register seperate url for the query params.

<p class="file-desc"><span>app-router.module.ts</span></p>

```typescript
    const routes: Routes = [
  { path: 'list', component: AppListComponent}
];
```

Here _list_ is the route url and *AppListComponent* is the component.

#### Pass query param through routerLink

```html
    <button type="button" [routerLink]="['/list']" [queryParams]="{id: '24'}">Show List</button>
```

Here *id* is the key and 24 is the static value. You can also pass the dynamic value through variable.

#### Pass router Param through Route Service

<p class="file-desc"><span>app.component.ts</span></p>

```typescript
    id = 28;
    constructor (private router: Router) {
    }

    route() {
        this.router.navigate(['/list'], {queryParams: {id: this.id}});
    }
```

#### For reading query params

<p class="file-desc"><span>app-list.component.ts</span></p>

```typescript
    constructor(
        private activatedroute: ActivatedRoute
    ) {
        this.activatedroute.queryParams.subscribe(data => {
            console.log(data);
        })
    }
```

Get more detail on [router params](https://alligator.io/angular/query-parameters/)

## 2. Pass Data through @Input and @Output;

If we want to pass the date from child to parent or parent to child and the hierarchy is one or two then we can use @Input and @Output.

<p class="file-desc"><span>app-parent.component.html</span></p>

```html
    <app-child [jsonData]="data" (outputData)="data = $event"></app-child>
```

Here *data* is a variable initialized in typescript file.

<p class="file-desc"><span>app-child.component.ts</span></p>

```typescript
    import { Component, Input, OnInit } from '@angular/core';

    @Component({
        selector:  'app-child',
        template: ''
    })
    export class AppChild implements OnInit {

        @Input() 
        jsonData;
        @Output()
        outputData  = new EventEmitter();
        constructor() {

        }

        ngOnInit() {
            console.log(this.jsonData);
        }

        emitData(data) {
            this.outputData(data);
        }
    }
```

In this way we can pass data from child to parent and from parent to child;

Get more detail on [@Input()](https://alligator.io/angular/inputs-angular/)

Get more detail on [@Output()](https://alligator.io/angular/outputs-angular/)

## 3. Pass Data through Service using Observables;

If two components are siblings or the level of hierarchy is more than it is good to use service for passing the data through observables.
Here I am using subject of rxjs for creating observable.


<p class="file-desc"><span>app.service.ts</span></p>

```typescript
    import { Injectable } from '@angular/core';
    import { Subject } from 'rxjs';

    @Injectable({providedIn: 'root'})
    export class AppService {
        observer = new Subject();
        public subscriber$ = this.observer.asObservable();

        emitData(data) {
            this.observer.next(data);
        }
    }
```

For emitting the data you can call the *emitData* method of this service and for getting the data you have to subscibe *subsciber$* like

```typescript
    constructor(private appService: AppService) {
    }

    ngOnInit() {
        this.appService.subscriber$.subscribe(data => {
            console.log(data);
        });
    }
```

