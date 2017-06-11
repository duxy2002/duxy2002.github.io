# Angular父组件和子路由之间传值的方法
## 方法1(通过Rx来传递内容)

1. 建立一个service类。(文件名：test-scenario.transfer.service.ts)

```typescript
import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Rx';

import { TestScenario } from './test-scenario.model';
import { ResponseWrapper, createRequestOption } from '../../shared';
import {BehaviorSubject} from 'rxjs/BehaviorSubject';

@Injectable()
export class TestScenarioTransferService {

    selectedRowsSubject: BehaviorSubject<any[]>;
    constructor() {
        this.selectedRowsSubject = new BehaviorSubject<any[]>([]);
    }

    get selectedScenarioRows() {
        return this.selectedRowsSubject.asObservable();
    }

    next(selectedRows: any[]) {
        this.selectedRowsSubject.next(Object.assign([], selectedRows));
    }
}
```

2. 在module中引入
```typescript
@NgModule({
    imports: [
         ...
    ],
    declarations: [
        ...
    ],
    providers: [
        ...
        TestScenarioTransferService,
        ...
    ],
    exports: [
        EtTestToolbarActionsDirective
    ],
    schemas: [ CUSTOM_ELEMENTS_SCHEMA ]
})
```

3. 在子路由中的构造器中注入这个Service，而后在相对应方法(rowSelect)中把想发给父组建的内容发送出去。
```typescript
@Component({
    selector: 'et-test-scenarios',
    templateUrl: './test-scenario.component.html',
})
export class TestScenarioComponent implements OnInit, AfterViewInit {
    selectedRows: any[] = [];
    constructor(
        ...
        private testScenarioTransferService: TestScenarioTransferService) {
       ...
    }
    rowSelect(rowSelectEvent: ITdDataTableSelectEvent): void {
        this.isRowSelectHappened = true;
        this.testScenarioTransferService.next(Object.assign([], this.selectedRows));
    }   
}
```

4. 在父组件中的构造器中注入这个Service，而后在相对应方法(rowSelect)中把想发给父组建的内容发送出去。
```typescript
@Component({
  selector: 'et-design-mgr',
  templateUrl: 'design-mgr.component.html',
})
export class TestDesignManagementComponent implements OnInit,  AfterViewInit {

    toolbarTitle: string;
    selectedScenarioRows: Observable<any[]>;
    routedComponent: any;

    constructor(
        ...
                private testScenarioTransferService: TestScenarioTransferService) {
        ...
        this.selectedScenarioRows = this.testScenarioTransferService.selectedScenarioRows;
    }
```

5. 而后直接在父组件的HTML中直接用异步管道监听内容就可以了。
```html
<button md-icon-button color="accent" *ngIf="(selectedScenarioRows | async)?.length >0"><md-icon class="md-24">play_circle_filled</md-icon></button>
```

## 方法2(通过@ViewChild和内容投影)
1. 建立一个子组件(toolbar)

* toolbar.component.ts
```typescript
import { Component, OnInit, AfterViewInit, OnDestroy } from '@angular/core';
@Component({
    selector: 'et-test-toolbar-action',
    templateUrl: './toolbar.component.html',
})
export class ToolbarActionComponent implements OnInit {
    ngOnInit(): void {
    }
}
```
* toolbar.component.html
```html
<ng-content select="[et-test-toolbar-buttons]"></ng-content>
```

2. 在父组件的HTML中使用内容投影，以及在router-outlet要素中监听activate事件。  
  里面需要特别注意的是[cdkPortalHost]="routedComponent?.toolbarActions"。这句话的意思是
  得到子路由相对应实例（routedComponent）。而后访问子路由中的@ViewChild
```html
            <ng-template  et-test-toolbar-buttons [cdkPortalHost]="routedComponent?.toolbarActions">
            <!--&lt;!&ndash;<ng-template  et-test-toolbar-action [ngIf]="true">&ndash;&gt;-->
                <!--<button md-icon-button color="accent" ><md-icon class="md-24">play_circle_filled</md-icon></button>-->
                <!--<button md-icon-button><md-icon class="md-24">settings</md-icon></button>-->
            </ng-template>

        </et-test-toolbar-action>
        <!--<button md-icon-button><md-icon class="md-24">settings</md-icon></button>-->
    </div>
    <router-outlet (activate)="routedComponent = $event"></router-outlet>
```

3. 对子路由组件做下面的修改。

* test-scenario.component.ts
```typescript
@Component({
    selector: 'et-test-scenarios',
    templateUrl: './test-scenario.component.html',
})
export class TestScenarioComponent implements OnInit, AfterViewInit {
    @ViewChild(EtTestToolbarActionsDirective) toolbarActions: EtTestToolbarActionsDirective;
    ....
```

* test-scenario.component.html
```html
<ng-template et-test-toolbar-buttons>
    <button md-icon-button color="accent" *ngIf="selectedRows?.length >0"><md-icon class="md-24">play_circle_filled</md-icon></button>
    <button md-icon-button><md-icon class="md-24">settings</md-icon></button>
</ng-template>
```