# How to integrate existing angular project to another one
This is a task I have met when I was working. We need to integrate our existing Angular application(let's say project A) into another developing Angular project(let's say project B).

The basic idea is encapsulate A as a module and publish it to npm so that B can install it by configure the package.json as a dependency library module.
## Set up poject A
### 1.Create Angular Elements
Angular elements are Angular components packaged as custome elements so that the Angular elements can be included into other web application that enable to reuse
```bash
npm install @webcomponents/custom-elements
```
### 2.Register Custom Elements
```bash
npm install @angular/elements
```
The @angular/elements package provides createCustomElement() API to convert together its dependencies to custom elements.   
The JavaScript function customElements.define() will register the custom element tag with the Browser.    
The ngDoBootstrap method will tell angular to use this module for bootstrapping.

in app.module.ts file:
```javascript
import { Injector }                     from '@angular/core';
import { createCustomElement }        from '@angular/elements';

export class DistributionReviewerModule {
    constructor (private injector: Injector) {
        const distributionReviewer = createCustomElement(AppComponent, { injector });
        customElements.define('dr-root', distributionReviewer);
    }
    ngDoBootstrap() {}
}
```
**Note: never import BrowserModule and BrowserAnimationsModule** Your module is a feature module, only the final user should import BrowserModule in the app root module. You can use CommonModule if you need common directives like ngIf ngFor
### 3. Export the public API
Create index.ts file under src folder, exporting all the public API of your module (at lease contains your module)
```javascript
export { DistributionReviewerModule } from './app/app.module';
```
### 4.Gulp configuration
Create gulpfile.js under root directory(same level with package.json), converting all files to inline template
```javascript
const gulp = require('gulp');
const inlineTemplates = require('gulp-inline-ng2-template');

/**
 * Inline templates configuration.
 * @see  https://github.com/ludohenin/gulp-inline-ng2-template
 */
const INLINE_TEMPLATES = {
    SRC: './src/**/*.ts',
    DIST: './tmp/src-inlined',
    CONFIG: {
        base: '/src',
        useRelativePaths: true,
    }
};

/**
 * Inline external HTML and SCSS templates into Angular component files.
 * @see: https://github.com/ludohenin/gulp-inline-ng2-template
 */
gulp.task('inline-templates', () => {
    return gulp.src(INLINE_TEMPLATES.SRC)
        .pipe(inlineTemplates(INLINE_TEMPLATES.CONFIG))
        .pipe(gulp.dest(INLINE_TEMPLATES.DIST));
});
```
### 5. Typescript configuration
Create tsconfig-build.json file under root directory to control compiler option to use by Typescript and Angular compiler(NGC)
Make sure you have installed **gulp, gulp-inline-ng2-template, @angular/compiler, @angular.compiler-cli, typescript, and rollup**
```json
{
    "angularCompilerOptions": {
        "strictMetadataEmit": true
    },
    "compilerOptions": {
        "baseUrl": "./tmp/src-inlined",
        "outDir": "./dist-npm",
        "declaration": true,
        "moduleResolution": "node",
        "experimentalDecorators": true,
        "emitDecoratorMetadata":true,
        "stripInternal" : true,
        "rootDir":"./tmp/src-inlined",
        "target": "es5",
        "module": "es2015",

        "lib": [
            "es2017",
            "dom"
        ],
        "types" :[]
    },
    "exclude": [
        "node_modules",
        "**/*.spec.ts",
        "e2e"
    ], "files" : ["./tmp/src-inlined/index.ts"]
}
```
### 6. Rollup configuration
Create rollup.config.js file under root directory,to deliver application in UMD format, which is similar to Angular modules   
the entry point is named index.js
```javascript
export default {
    input: 'dist-npm/index.js',
    output: {
        file: 'dist-npm/bundles/distribution-reviewer.umd.js',
        format: 'umd',
        sourceMap: false,
        name: 'ng.distribution-reviewer'
    }
}
```
### 7. Build
In the package.json file,
```json
"scripts": {
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "build:esm": "gulp inline-templates",
    "transpile": "ngc -p ./tsconfig-build.json",
    "package": "rollup -c",
    "build": "npm run build:esm && npm run transpile && npm run package"
},
```
type **npm run build"** in the command line
### 8. Publish to npm
Create another package.json file under dist-npm folder (generated after build) to include all the details to publish to NPM
```json
{
  "name": "distribution-reviewer",
  "author":"Ryan Zhang",
  "version": "0.0.7",
  "license": "MIT",
  "main": "bundles/distribution-reviewer.umd.js",
  "module": "index.js",
  "typings": "index.d.ts"
}

```
```bash
cd dist-npm
npm publish
```
Note: **You will be asked to log in with your npm account**
## Set up Project B
### Notes:
1. Install @angular/elements, @webcomponents/custom-elements, and your published module (In my project: distribution-reviewer)
2. In angular.json file, add scripts under "build" property
```json
"scripts": [
              "node_modules/@webcomponents/custom-elements/src/native-shim.js"
            ]
```
3.In the place where you want to use your feature module, add schemas
```javascript
import {CUSTOM_ELEMENTS_SCHEMA, NgModule} from '@angular/core';
import { CommonModule } from '@angular/common';
import { PrepRoutingModule } from './prep.routing';
import { PrepComponent } from './prep.component';

@NgModule({
  imports: [
    CommonModule,
    PrepRoutingModule
  ],
  declarations: [
    PrepComponent
  ],
  schemas: [ CUSTOM_ELEMENTS_SCHEMA ]
})
export class PrepModule { }
```
Note: **In DR, you cannot use lazy-loading twice, comment AppRoutingModule in app.module.ts and use dr-dashboard instead of router-outlet**
## Reference:
[https://itnext.io/building-micro-frontend-applications-with-angular-elements-34483da08bcb](https://itnext.io/building-micro-frontend-applications-with-angular-elements-34483da08bcb)
[https://medium.com/@cyrilletuzi/how-to-build-and-publish-an-angular-module-7ad19c0b4464](https://medium.com/@cyrilletuzi/how-to-build-and-publish-an-angular-module-7ad19c0b4464)
