# Form Plugin - 실험적인 상태
리액티브 폼을 앵귤러로 작성하려면, 스토어에서 폼으로 값을 바인드하고 반대로도 바인딩해야 한다.
스토어의 값은 감시가 가능하고 리액티브 폼은 로우 오브젝트를 받는다.
결과적으로 앞뒤로 몽키 패치한다.

이러한 이슈 외에도 폼을 채우다가 페이지를 나갔다가 다시 돌아와 현재의 상태를 계속하려는 워크 플러가 있다.
이는 스토어를 위한 훌륭한 유스케이스이고 이 플러그인으로 이러한 상황을 극복 할 수 있다.

요컨데, 이 플러그인은 폼과 상태를 동기화된 상태로 유지하도록 돕는다.

## Installation
```bash
npm install @ngxs/form-plugin --save

# yarn을 사용
yarn add @ngxs/form-plugin
```

## Usage
애플리케이션의 최상위 모듈의 imports에 `NgxsFormPluginModule`을 추가한다.

```TS
import { NgxsFormPluginModule } from '@ngxs/form-plugin';
import { PizzaState } from './pizza.state';

@NgModule({
  imports: [
    NgxsModule.forRoot([PizzaState]),
    NgxsFormPluginModule.forRoot(),
  ]
})
export class AppModule {}
```

폼을 하위 모듈에서 사용한다면, 하위 모듈에도 포함해야 한다.

```TS
import { NgxsFormPluginModule } from '@ngxs/form-plugin';

@NgModule({
  imports: [
    NgxsFormPluginModule,
  ]
})
export class SomeModule {}
```

### Form State
기본 폼 상태를 애플리케이션 상태의 일부로 정의한다.

```TS
import { State } from '@ngxs/store';

@State({
  name: "todos",
  defaults: {
    pizzaForm: {
      model: undefined,
      dirty: false,
      status: "",
      errors: {}
    }
  }
})
export class PizzaState {}
```

### Form Setup
컴포넌트에서, 리액티브 폼을 구현하고 `ngxsForm` 디렉티브로 꾸미면서 상태 오브젝트의 경로를
_문자열_ 로 전달 한다. 디렉티브는 이 경로를 사용하여 자신을 스토어에 연결하고 바인딩을 설정한다.

```TS
import { Component } from '@angular/core';
import { FormBuilder } from '@angular/forms';

@Component({
  selector: 'pizza-form',
  template: `
    <form [formGroup]="pizzaForm" novalidate ngxsForm="todos.pizzaForm" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="toppings" />
      <button type="submit">Order</button>
    </form>
  `
})
export class PizzaComponent {
  pizzaForm = this.formBuilder.group({
    toppings: ''
  });

  constructor(private formBuilder: FormBuilder) {
  }

  onSubmit() {
    //
  }
}
```

이제 폼이 업데이트 될 때마다, 상태도 새로운 상태를 반영한다.

디렉티브는 활용할 수 있는 두개의 주입자(Input)도 있다.

- `ngxsFormDebounce: number` - 폼의 값 변경 바운스 제거 시간을 설정한다. 기본값은 `100`이다.
- `ngxsFormClearOnDestroy: boolean` - 폼 파괴될 때 상태를 지운다

### Actions
자동으로 폼의 상태를 추적하는 것 외에도, 수동으로 폼 상태를 초기화하는 것 같은 액션을 디스패치할 수도 있다. 예:

```TS
this.store.dispatch(
  new UpdateFormDirty({
    dirty: false,
    path: 'todos.pizzaForm'
  })
);
```

폼 플러그인은 다음과 같은 `actions`을 바로 제공한다.
- `UpdateFormStatus({ status, path })` - 폼 상태 업데이트
- `UpdateFormValue({ value, path })` - 폼 값을 업데이트
- `UpdateFormDirty({ dirty, path })` - 폼 수정 상태 업데이트
- `SetFormDisabled(path)` - 폼을 사용 불가로 설정
- `SetFormEnabled(path)` - 폼을 사용 가능으로 설정
- `SetFormDirty(path)` - 폼을 수정 상태로 설정(`UpdateFormDirty`의 단축)
- `SetFormPristine(path)` - 폼을 미수정 상태로 설정( `UpdateFormDirty`의 단축)
