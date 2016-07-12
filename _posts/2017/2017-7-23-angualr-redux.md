---
layout: post
title: 在Angular中使用类Redux工具—ngrx/store 2017-07-22 19:12
categories:
- web
tags:
- angular
---

![head](https://cdn-images-1.medium.com/max/1000/1*r_2RBJXnxtJM3ZjOGNVtug.png)

这篇文章中，我们将要讨论如何用[ngrx/effects](https://github.com/ngrx/effects)连接Angular表单和[ngrx/store](https://netbasal.com/connect-angular-forms-to-ngrx-store-c495d17e129)

我们最终的[结果](https://gist.github.com/NetanelBasal/8c033644ea0c6c8105a27072ae1af461)是这样的

```javascript
\\new-story0.component.ts

@Component({
  selector: 'new-story-form',
  template: `
    <form [formGroup]="newStory"
          (submit)="submit($event)"
          (success)="onSuccess()"
          (error)="onError($event)"
          connectForm="newStory">
       ...controls
    </form>
    <div *ngIf="success">Success!</div>
    <div *ngIf="error">{{error}}</div>
})
class NewStoryFormComponent {...}
```

![gif](https://cdn-images-1.medium.com/max/800/1*IY8vvVANltaxbCQz2kU50g.gif)

### The Reducer

方便起见，我们会写一个简单的reducer来组织管理我们应用里的所有的表单

状态（the state）将由一个用ID作为key，表格数据作为value的简单对象构成

举个例子

```javascript
\\connect-form.reducer.ts

const initialState = {
  newStory: {
    title: '',
    description: ''
  },
  contactUs: {
    email: '',
    message: ''
  }
}

export function forms(state = initialState, action) {
}

```


我们先构建一个action——UPDATE_FORM。这个action由两个key：path和value组成



```javascript

\\connect-form1.reducer.ts

store.dispatch({
  type: UPDATE_FORM,
  payload: {
    path: 'newStory',
    value: formValue
  }
});

```


然后这个reducer将负责更新state


```javascript
\\connect-form2.reducer.ts

export function forms(state = initialState, action) {
  if(action.type === UPDATE_FORM) { 
                       // newStory:           formValue
    return { ...state, [action.payload.path]: action.payload.value }
  }
}

```

### 连接表单的组件——ConnectForm Directive

### 获取State

我们想要基于state更新表单，所以我们需要path作为输入，然后取出store中正确的片段

```javascript

\\connect-form.directive.ts

@Directive({
  selector: '[connectForm]'
})
export class ConnectFormDirective {
  @Input('connectForm') path: string;

  constructor(private formGroupDirective: FormGroupDirective,
    private store: Store<AppState> ) {
      
    ngOnInit() {
      // Update the form value based on the state
      this.store.select(state => state.forms[this.path]).take(1).subscribe(formValue => {
        this.formGroupDirective.form.patchValue(formValue);
      });
    }
  }
}

```

我们抓取表单directive实例然后从store里更新表单数据

### 更新State

当表单数据改变时我们也需要更新表单状态。我们可以通过订阅（subscribe）这个`valueChanges`的可观察对象（observable）然后调度（dispatch）这个UPDATE_FORM的action来获取值

```javascript
\\connect-form1.directive.ts

this.formChange = this.formGroupDirective.form.valueChanges
  .subscribe(value => {
    this.store.dispatch({
      type: UPDATE_FORM,
      payload: {
        value,
        path: this.path, // newStory
      }
    });
})

```


这就是表单和State同步所要做的全部工作了


### 通知和重置

有两件事我们要在这个部分完成

1. 基于HTTP响应返回来显示通知给用户——我们需要保证通知直接传给组件并且不储存信息在store里

有两点原因

- 通常，没有其他的组件需要这个信息

- 我们不想每次都重置store

2. 当提交成功时重置表单

我们将让Angular尽其所能，处理好前端表单校验并重置表单

### 成功的Action

成功的Action包含表单的path属性所以我们可以知道到底哪个表单需要重置，同时什么时候需要去使用（emit）这个成功的事件

```javascript

\\connect-form2.directive.ts

const FORM_SUBMIT_SUCCESS = 'FORM_SUBMIT_SUCCESS';
const FORM_SUBMIT_ERROR = 'FORM_SUBMIT_ERROR';
const UPDATE_FORM = 'UPDATE_FORM';

export const formSuccessAction = path => ({
  type: FORM_SUBMIT_SUCCESS,
  payload: {
    path
  }
});

```

### 异常的Action

同成功的action一样，因为有path的存在，我们也知道何时去使用（emit）错误异常 的事件

```javascript
\\connect-form3.directive.ts

export const formErrorAction = ( path, error ) => ({
  type: FORM_SUBMIT_ERROR,
  payload: {
    path,
    error
  }
});
```

我们需要创建 成功 和 错误异常 的输出 然后 监听 `FORM_SUBMIT_ERROR` 和 `FORM_SUBMIT_SUCCESS` 的 action。

因为我们正好要使用 ngrx/effects ，此时我们就可以用 Action 的服务（service）来监听actions了

```javascript

\\connect-form3.directive.ts

@Directive({
  selector: '[connectForm]'
})
export class ConnectFormDirective {
  @Input('connectForm') path : string;
  @Input() debounce : number = 300;
  @Output() error = new EventEmitter();
  @Output() success = new EventEmitter();
  formChange : Subscription;
  formSuccess : Subscription;
  formError : Subscription;

  constructor( private formGroupDirective : FormGroupDirective,
               private actions$ : Actions,
               private store : Store<any> ) {
  }

  ngOnInit() {
    this.store.select(state => state.forms[this.path])
      .debounceTime(this.debounce)
      .take(1).subscribe(val => {
      this.formGroupDirective.form.patchValue(val);
    });

    this.formChange = this.formGroupDirective.form.valueChanges
      .debounceTime(this.debounce).subscribe(value => {
        this.store.dispatch({
          type: UPDATE_FORM,
          payload: {
            value,
            path: this.path,
          }
        });
      });

    this.formSuccess = this.actions$
      .ofType(FORM_SUBMIT_SUCCESS)
      .filter(( { payload } ) => payload.path === this.path)
      .subscribe(() => {
        this.formGroupDirective.form.reset();
        this.success.emit();
      });

    this.formError = this.actions$
      .ofType(FORM_SUBMIT_ERROR)
      .filter(( { payload } ) => payload.path === this.path)
      .subscribe(( { payload } ) => this.error.emit(payload.error))
  }
}
```

当然，我们不能忘了清空订阅

```javascript

\\connect-form4.directive.ts

ngOnDestroy() {
  this.formChange.unsubscribe();
  this.formError.unsubscribe();
  this.formSuccess.unsubscribe();
}

```

最后一步就是在有返回的时候调用表单的actions

```javascript

\\connect-form4.directive.ts

import {
  formErrorAction,
  formSuccessAction
} from '../connect-form.directive';

@Effect() addStory$ = this.actions$
  .ofType(ADD_STORY)
  .switchMap(action =>
    this.storyService.add(action.payload)
    .switchMap(story => (Observable.from([{
      type: 'ADD_STORY_SUCCESS'
    }, formSuccessAction('newStory')])))
    .catch(err => (Observable.of(formErrorAction('newStory', err))))
  )

 
```

 现在我们可以在组件里显示提醒了

```javascript

\\new-story.component.ts

 @Component({
  selector: 'new-story-form',
  template: `
    <form [formGroup]="newStory"
          (submit)="submit($event)"
          (success)="onSuccess()"
          (error)="onError($event)"
          connectForm="newStory">
       ...controls
      <button [disabled]="newStory.invalid" type="submit">Submit</button>
    </form>
    <div *ngIf="success">Success!</div>
    <div *ngIf="error">{{error}}</div>
  `
})
class NewStoryFormComponent {

  constructor(private store: Store<AppState> ) {}

  onError(error) {
    this.error = error;
  }

  onSuccess() {
    this.success = true;
  }

  submit() {
    // You can also take the payload from the form state in your effect 
    // with the withLatestFrom observable
    this.store.dispatch({
      type: ADD_STORY,
      payload: ...
    })
  }

}

```