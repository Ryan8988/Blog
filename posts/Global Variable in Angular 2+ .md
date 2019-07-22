# Global Variable in Angular
Sometimes we need to declare and access global variable in Angular, the best way to do this is using dependency injection(like service)
## 1: Create global variables in a new TS file global.model.ts
```javascript
export class Globals {
  availableColumns: string [] = ['CHIRAL', 'ACTIVITY', 'TIME SPENT', 'IA COLUMN TYPE'];
  seletedColumns: string [] = ['', 'SAMPLE ID', 'PROJECT', 'SUBMITTER', 'SUBMITTER SITE', 'ANALYSIS GROUP'];
}
```
## 2: import global variable into ts
```
import {Globals} from '../../shared/models/globals.model';
```
## 3: Provide the service in the module
```
providers: [Globals]
```
## 4: use it the same way as Angular service
```
constructor(private plato: PlatoService,
            private selected: SelectedSampleService,
            private breadcrum: BreadcrumbService,
            private spinner: NgxSpinnerService,
            private toastr: ToastrService,
            private defaultColumns: Globals) {
            }
```
