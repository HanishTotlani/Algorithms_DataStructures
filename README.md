---
layout: page-fullwidth
title:  "Custom validation in angular"
categories:
    - angular
tags: 
    - angular
    - validation
    - template driven forms
    - reactive forms
header: no
breadcrumb: true
meta_description: "Creating custom validator in angular."
author: Hanish Totlani
---

In this post I am going to share how to build custom validator and add this validator in `Template Driven Forms` or `Reactive Forms`.


Here, I am going to demonstrate the validation of phone number which should be of 10 digits.


[Invalid-Phone](./images/custom-validation/invalid-form-control.PNG)
[Valid-Phone](./images/custom-validation/valid-form-contol.PNG)


## Validator for Template Driven Form

For validation in template driven forms directives are used.


<p class="file-desc"><span>phone-number-validator.ts</span></span></p>

```typescript
    import { Directive } from '@angular/core';
    import { Validator, AbstractControl, NG_VALIDATORS } from '@angular/forms';

    @Directive({
        selector: '[phoneValidateDirective]',
        providers: [{
            provide: NG_VALIDATORS,
            useExisting: AppPhoneValidateDirective,
            multi: true
        }]
    })
    export class AppPhoneValidateDirective implements Validator {
        validate(control: AbstractControl) : {[key: string]: any} | null {
            if (control.value && control.value.length != 10) {
                return { 'phoneNumberInvalid': true }; // return object if the validation is not passed.
            }
            return null; // return null if validation is passed.
        }
    }
```

To register and add the validator to the existing validator array `NG_VALIDATORS` provided by angular this is **important**;


```typescript
    providers: [{
            provide: NG_VALIDATORS,
            useExisting: Your_Class_Name,
            multi: true
        }]
```

Here I have created the phone directive which implements `Validator` of `@angular/forms` for which we have to provide implementation `validate(control: AbstractControl): : {[key: string]: any} | null`  method. This validator will return an object if the validation is not passed which is `{ 'phoneNumberInvalid': true }`  and will return null if the validation is passed.



Angular adds the return value of the validation function in errors property of [FormControl](https://angular.io/api/forms/FormControl) / [NgModel](https://angular.io/api/forms/NgModel). If the errors property of the [FormControl](https://angular.io/api/forms/FormControl) / [NgModel](https://angular.io/api/forms/NgModel) is not empty then the form is invalid and if the errors property is empty then the form is valid.


<p class="file-desc"><span>app.component.html</span></span></p>

```html
    <div class="form-group col-sm-4">
        <label for="">Phone</label>
        <input type="text" class="form-control" name="phone" [(ngModel)]="phone" [class.is-invalid]="phonengModel.errors?.phoneNumberInvalid && (phonengModel.touched || phonengModel.dirty)" #phonengModel="ngModel" phoneValidateDirective>    <!-- Added directive to validate phone -->
        <span class="invalid-feedback" *ngIf="(phonengModel.touched || phonengModel.dirty) && phonengModel.errors?.phoneNumberInvalid"> <!-- Checked the errors property contains the 'phoneNumberInvalid' property or not which is returned by the validation function -->
            Phone number must be of 10 digit
        </span>
    </div>
```

## Validator for Reactive Form

For validation in Reactive Form we have to create a function.

<p class="file-desc"><span>app.component.ts</span></span></p>

```typescript
    import { FormBuilder, AbstractControl } from '@angular/forms';
    import { Component, OnInit } from "@angular/core";

    @Component({
        selector: 'reactive-form',
        templateUrl: './reactive-form.component.html'
    })
    export class AppReactiveForm implements OnInit {

        myForm: FormGroup;

        constructor(
            private fb: FormBuilder
        ) {

        }

        ngOnInit() {
            this.myForm = this.fb.group({
                phone: ['', [ValidatePhone]] // added the function in validators array of form-control
            });
        }
    }
    
    function ValidatePhone(control: AbstractControl): {[key: string]: any} | null  {
        if (control.value && control.value.length != 10) {
            return { 'phoneNumberInvalid': true };
        }
        return null;
    }
```

we are adding function to the validators array of Form-Control

<p class="file-desc"><span>app.component.html</span></span></p>

```html
    <form [formGroup]="myForm" novalidate class="needs-validation" (ngSubmit)="saveForm(myForm)">
    <div class="row">
        <div class="form-group col-sm-4">
            <label for="">Password</label>
            <input type="text" class="form-control " formControlName="phone" [class.is-invalid]="(myForm.get('phone').touched || myForm.get('phone').dirty) && myForm.get('password').">
            <span class="invalid-feedback" *ngIf="(myForm.get('password').touched || myForm.get('password').dirty) && myForm.get('password').invalid">
                Password is required
            </span> 
        </div>
    </div> 
    </form>
```

## Combining both the validators

Combining both the validators so that we do not repeat the code and follow **DRY**(Don't Repeat Yourself) principle

<p class="file-desc"><span>phone-number-validator.ts</span></span></p>

```typescript
    import { Directive } from '@angular/core';
    import { Validator, AbstractControl, NG_VALIDATORS } from '@angular/forms';

    export function ValidatePhone(control: AbstractControl): {[key: string]: any} | null  {
        if (control.value && control.value.length != 10) {
            return { 'phoneNumberInvalid': true };
        }
        return null;
    }

    @Directive({
        selector: '[phone]',
        providers: [{
            provide: NG_VALIDATORS,
            useExisting: AppPhoneValidateDirective,
            multi: true
        }]
    })
    export class AppPhoneValidateDirective implements Validator {
        validate(control: AbstractControl) : {[key: string]: any} | null {
        return ValidatePhone(control);
        }
    }
```

For better understanding visit [Providers](https://alligator.io/angular/providers-angular/) and [AbstractControl](https://angular.io/api/forms/AbstractControl)
