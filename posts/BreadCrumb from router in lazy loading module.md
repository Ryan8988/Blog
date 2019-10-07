# How to create BreadCrumb in Lazy Loading Module
### 1. Create BreadCrumb component
We need to create a compoent to hold breadcrumb
### 2. Listen to the Router event
in breadcrumb.component.ts, 
```javascript
constructor(private router: Router,
            private activatedRoute: ActivatedRoute) {
  this.breadcrumbs = [];
  this.router.events.pipe(
    filter(event => event instanceof NavigationEnd)).subscribe( event => {
    const root: ActivatedRoute = this.activatedRoute.root;
    this.breadcrumbs = this.getBreadcrumbs(root);
  });
}
```
### 3. Build breadcrumbs with the root of the activated route
in breadcrumb.component.ts
```javascript
getBreadcrumbs(route: ActivatedRoute, url: string = 'home', breadcrumbs: BreadCrumb[] = []): BreadCrumb[] {
    const ROUTE_DATA_BREADCRUMB = 'breadcrumb';
    const children: ActivatedRoute[] = route.children;
    let label = '';
    if (children.length === 0) {
      return breadcrumbs;
    }
    for (const child of children) {
      //verify primary route
      if (child.outlet !== PRIMARY_OUTLET) {
        continue;
      }
      if (!child.snapshot.data.hasOwnProperty(ROUTE_DATA_BREADCRUMB)) {
        return this.getBreadcrumbs(child, url, breadcrumbs);
      }
      if (child.snapshot.url.map(segment => segment.path).length === 0) {
        return this.getBreadcrumbs(child, url, breadcrumbs);
      }
      const routeArray = child.snapshot.url.map(segment => segment.path);
      for ( let i = 0; i < routeArray.length; i++ ) {
        if ( Object.keys(child.snapshot.params).length > 0 ) {
          if ( this.isParam(child.snapshot.params, routeArray[i]) ) {
            label = routeArray[i];
          } else {
            label = child.snapshot.data[ROUTE_DATA_BREADCRUMB];
          }
        } else {
          label = child.snapshot.data[ROUTE_DATA_BREADCRUMB];
        }
        const routeURL = routeArray[i];
        url += `/${routeURL}`;
        const breadcrumb: BreadCrumb = {
          label: label,
          params: child.snapshot.params,
          url: url
        };
        breadcrumbs.push(breadcrumb);
      }
      //recursive
      return this.getBreadcrumbs(child, url, breadcrumbs);
    }
    console.log(breadcrumbs)
    return breadcrumbs;
}
```
### 4. Add data to the routes
in dashboard.routing.ts file
```javascript
export const childRoutes: Routes = [
  {
    path: 'home',
    component: DashboardComponent,
    children: [
      {path: '', redirectTo: 'navi', pathMatch: 'full'},
      {path: 'navi', loadChildren: 'src/app/dashboard/navigation/navigation.module#NavigationModule'},
      {path: 'ia', loadChildren: 'src/app/dashboard/ia/ia.module#IaModule', data: {breadcrumb: 'IA'}},
      {path: 'ia/:id', loadChildren: 'src/app/dashboard/sample-details/sample-details.module#SampleDetailsModule', data: {breadcrumb: 'IA'}},
      {path: 'prep', loadChildren: 'src/app/dashboard/prep/prep.module#PrepModule', data: {breadcrumb: 'PREP'}},
      {path: 'prep/:id', loadChildren: 'src/app/dashboard/sample-details/sample-details.module#SampleDetailsModule', data: {breadcrumb: 'PREP'}},
      {path: 'qc', loadChildren: 'src/app/dashboard/qc/qc.module#QcModule', data: {breadcrumb: 'QC'}},
      {path: 'qc/:id', loadChildren: 'src/app/dashboard/sample-details/sample-details.module#SampleDetailsModule', data: {breadcrumb: 'QC'}},
      {path: 'reviewer', loadChildren: 'src/app/dashboard/reviewer/reviewer.module#ReviewerModule', data: {breadcrumb: 'Distribution Reviewer'}},
      {path: 'nmr', loadChildren: 'src/app/dashboard/nmr/nmr.module#NmrModule', data: {breadcrumb: 'NMR'}},
      {path: 'registration', loadChildren: 'src/app/dashboard/registration/registration.module#RegistrationModule', data: {breadcrumb: 'Registration'}},
      {path: 'reports', loadChildren: 'src/app/dashboard/reports/reports.module#ReportsModule', data: {breadcrumb: 'Reports'}},
      {path: 'samples/:id', loadChildren: 'src/app/dashboard/sample-details/sample-details.module#SampleDetailsModule', data: {breadcrumb: 'All Samples'}},
      {path: 'samples', loadChildren: 'src/app/dashboard/samples/samples.module#SamplesModule', data: {breadcrumb: 'All Samples'}},
      {path: 'sorter', loadChildren: 'src/app/dashboard/vial-sorter/vial-sorter.module#VialSorterModule', data: {breadcrumb: 'Vial Sorter'}},
      {path: 'replacement', loadChildren: 'src/app/dashboard/vial-replacement/vial-replacement.module#VialReplacementModule', data: {breadcrumb: 'Vial Replacement'}},
      {path: 'lookup', loadChildren: 'src/app/dashboard/computer-lookup/computer-lookup.module#ComputerLookupModule', data: {breadcrumb: 'Computer Lookup'}},
      {path: 'shipping', loadChildren: 'src/app/dashboard/shipping/shipping.module#ShippingModule'},
      {path: 'search', loadChildren: 'src/app/dashboard/search/search.module#SearchModule', data: {breadcrumb: 'Search'}},
      {path: 'settings', loadChildren: 'src/app/dashboard/settings/settings.module#SettingsModule', data: {breadcrumb: 'Setting'}},
      {path: '', pathMatch: 'full', redirectTo: 'navi' }
    ]
  },
];

export const routing = RouterModule.forChild(childRoutes);
```
### 5. Display breadcrumb in the screen
in breadcrumb.compnent.html
```html
<div class="breadcrumb">
  <li><a routerLink="">Home</a></li>
  <li *ngFor="let breadcrumb of breadcrumbs">
    <a routerLink="../{{breadcrumb.url}}">{{breadcrumb.label}}</a>
  </li>
</div>
```
### 6. CSS style
in breadcrumb.compnent.css
```css
.breadcrumb li {
  display: inline;
}
.breadcrumb li+li:before {
  padding: 8px;
  color: black;
  content: "/\00a0";
}
.breadcrumb li a {
  color: #0275d8;
  text-decoration: none;
}
```
