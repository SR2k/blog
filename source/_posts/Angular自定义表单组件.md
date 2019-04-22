---
title: Angular自定义表单组件
date: 2019-03-22 13:51:11
tags:
  - TypeScript
  - Angular
  - 前端
---

Angular、Vue、React 等框架为前端引入了「模块化的概念」，我们可以预先定义好不同的组件，然后在各处复用。而表单在各种 WebApp 中都处于十分重要的位置。今天这个简单的小教程就是在 Angular 中实现一个可以复用的自定义表单组建。

这个表单组建功能十分简单：允许用户输入手机号，并选择手机号的国家区号。

# 新建&配置项目

首先我们使用 Angular CLI 来新建一个最最普通的 Angular 应用，并创建一个名叫 `PhoneNumInput` 
的组件作为我们的自定义表单组件：

``` bash
# 我更喜欢用 Yarn 所以这里先跳过 NPM 的安装过程了
ng new ng-custom-form-control-demo --skip-install
cd ng-custom-form-control-demo
yarn

ng g c phone-num-input
```

由于我们会使用响应式表单，因此需要在 `app.module.ts` 中的 `imports` 中导入 
`ReactiveFormsModule`：

``` typescript
// src/app/app.module.ts

// ...
imports: [
  // ...
  ReactiveFormsModule
]
// ...
```

## 输入框

接下来，我们需要为自组件添加一个输入框和一个选择框，修改 `phone-num-input.component.html`：

``` html
<!-- src/phone-num-input/phone-num-input.component.html -->

<select [ngModel]="regionCode">
  <option value="+86">+86</option>
  <option value="+1">+1</option>
</select>
<input type="tel" placeholder="Phone number..." [ngModel]="phoneNum" />
```

然后在 `phone-num-input.component.ts` 中新增两个属性：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements OnInit, ControlValueAccessor {
  regionCode = '+86';
  phoneNum = '';

  // ...
}
```

## 引入子组件

然后，我们需要在 `AppComponent` 中引入我们的表单组件。

将 `app.component.html` 中的内容修改为：

``` html
<!-- src/app/app.component.html -->

<form [formGroup]="form">
  <app-phone-num-input formControlName="phoneNum"></app-phone-num-input>
</form>
```

然后修改 `app.component.ts`：

``` typescript
// src/app/app.component.ts

// ...
export class AppComponent implements OnInit {
  // ...

  // 表单
  form: FormGroup;

  // 注入 FormBuilder 来生成表单
  constructor(private fb: FormBuilder) { }

  ngOnInit() {
    // 生成表单
    this.form = this.fb.group({
      phoneNum: [null, [Validators.pattern(/^\+1/)]]
    });
  }
}
```

# 注册服务提供商

现在，我们需要让 Angular 知道我们的 `PhoneNumInputComponent` 是一个自定义表单组件，因此
需要为其注册服务提供商，在 `phone-num-input.component.ts` 中 `PhoneNumInputComponent` 
的类定义之前假如如下元数据：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export const APP_PHONE_NUM_INPUT_VALUE_ACCESSOR = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => PhoneNumInputComponent),
  multi: true
};
```

然后在 `PhoneNumInputComponent` 的 `@Component` 装饰器中加上一个 `providers` 数组：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

@Component({
  // ...
  providers: [APP_PHONE_NUM_INPUT_VALUE_ACCESSOR]
})
```

# `ControlValueAccessor` 接口

要使用自定义表单组件，我们必须为组件实现 `ControlValueAccessor` 接口，以此来告诉 Angualr 
表单如何与我们的组件交互，此接口的定义如下：

``` typescript
interface ControlValueAccessor {
  // 外部向组件内写入数值
  writeValue(obj: any): void
  // 注册数据变化回调
  registerOnChange(fn: any): void
  // 注册脏值回调
  registerOnTouched(fn: any): void
  // 禁用状态配置
  setDisabledState(isDisabled: boolean)?: void
}
```

将 `PhoneNumInputComponent` 类修改如下：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements ControlValueAccessor {
  registerOnChange(fn: (val: string) => void) {
  registerOnTouched(fn: (touched: boolean) => void) {
  setDisabledState(isDisabled: boolean): void {}
  writeValue(val: string): void {}
}
```

看起来很复杂，下边用例子解释这些方法要怎么使用。

## 禁用组件

首先，我们来实现一个最最简单的功能：在父组件禁用我们的表单组件。

修改 `app.module.ts`，增加表单禁用的属性和方法：

``` typescript
// src/app/app.component.ts

export class AppComponent implements OnInit {
  // ...

  formDisabled = false;

  toggleFormDisable() {
    this.formDisabled = !this.formDisabled;
    if (this.formDisabled) {
      this.form.disable();
    } else {
      this.form.enable();
    }
  }
}
```

然后修改 `app.component.html`，添加禁用表单的按钮：

``` html
<!-- src/app/app.component.html -->

<!-- ... -->
<div>
  <button (click)="toggleFormDisable()">Toggle Disable</button>
</div>
```

接下来我们就要实现 `setDisabledState` 方法了。这个方法会在父组件请求修改子表单组件的禁用状态
时被调用，因此，修改 `phone-num-input.component.ts`：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements OnInit, ControlValueAccessor {
  disabled = false;

  // ...

  setDisabledState(isDisabled: boolean) {
    this.disabled = isDisabled;
  }
}
```

然后在 `phone-num-input.component.html` 中把我们的 `input` 的 `disabled` 属性绑定到
组件上的 `disabled` 中：

``` html
<!-- src/phone-num-input/phone-num-input.component.html -->

<input type="tel" placeholder="Phone number..." [disabled]="disabled" />
```

现在，你在 `AppComponent` 中点击 Toggle Disable 按钮时，子组件的 `input` 就会在禁用/启用
的状态中切换了。

## 父组件向表单组件传值

现在假设我们的用户已经在别处提供过手机号了，因此我们希望在 `AppComponent` 被初始化时就给
`PhoneNumInputComponent` 提供一个初始值。这里就可以使用 `writeValue` 方法了。

首先修改 `AppComponent` 的 `ngOnInit()` 钩子方法：

``` typescript
// src/app/app.component.ts

ngOnInit() {
  // ...

  this.form.get('phoneNum').setValue('+86-13912345678');
}
```

然后修改 `PhoneNumInputComponent` 的 `writeValue()` 方法：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

writeValue(val: string) {
  const s = val.split('-');
  this.regionCode = s[0];
  this.phoneNum = s[1];
}
```

现在，每次打开你的 app，页面上都会显示预先定义好的值了。

## 子组件向父组件传值

现在，我们要在用户输入时告诉父组件数据的变化，这里是用的是 `registerOnChange()` 注册的方法。

在 `app-component.html` 中加入一行：

``` html
<!-- src/app/app.component.ts -->

<!-- ... -->
<p>Your phone number is: {{this.form.get('phoneNum').value}}</p>
<!-- ... -->
```

现在，让我们注册一下子组件向父组件传值的方法：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements ControlValueAccessor {
  // ...

  onChange: (val: string) => void;

  // registerOnChange 会给我们传入一个函数，用来向上传值，我们需要把这个函数保存下来
  registerOnChange(fn: (val: string) => void) {
    this.onChange = fn;
  }

  // ...
}
```

由于我们需要观测 `PhoneNumInputComponent` 中两个 `ngModel` 的变化，因此修改
`phone-num-input.component.html`：

``` html
<!-- src/phone-num-input/phone-num-input.component.html -->

<select [ngModel]="regionCode"
        (ngModelChange)="regionCodeChange($event)">
  <option value="+86">+86</option>
  <option value="+1">+1</option>
</select>
<input type="tel"
       placeholder="Phone number..."
       [ngModel]="phoneNum"
       (ngModelChange)="phoneNumChange($event)" />
```

然后在组件中添加对应的变动处理方法：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements ControlValueAccessor {
  // ...

  regionCodeChange(val: string) {
    this.regionCode = val;
    // 调用传值方法，向父组件传值
    this.onChange(`${this.regionCode}-${this.phoneNum}`);
  }
  phoneNumChange(val: string) {
    this.phoneNum = val;
    // 调用传值方法，向父组件传值
    this.onChange(`${this.regionCode}-${this.phoneNum}`);
  }
}
```

现在，你在输入框里输入数值，首页的 `<p>` 里的电话号码也会跟着变了～

## 更新组件脏值状态

如果你足够细心，你会发现我之前给表单添加了一条验证规则：`Validators.pattern(/^\+1/)`，这就
意味着只有以 +1 开头的手机号才是正确的。

现在我们要给表单增加一个提示，告诉用户你的输入不正确。试想，如果你是用户，你明明还没有修改表单的
数据，表单就告诉你输错了，肯定是个不好的体验。更好的设计应该是：用户已经手动修改过数据了，而且输
错了，再显示提示。

在 Angular 中我们可以使用 `FormControl` 的 `touched` 属性知道值是否被用户动过。

修改你的，添加错误提示语：

``` html
<!-- src/app/app.component.ts -->

<!-- ... -->
<p *ngIf="this.form.get('phoneNum').touched && this.form.get('phoneNum').hasError('pattern')">
  Phone number shall start with +1!
</p>
<!-- ... -->
```

现在，我们需要在子组件 `<input>` 和 `<select>` 被修改时通知父组件：这个值被用户动过啦！

修改 `phone-num-input.component.ts`：

``` typescript
// src/phone-num-input/phone-num-input.component.ts

export class PhoneNumInputComponent implements ControlValueAccessor {
  // ...
  onTouched: (touched: boolean) => void;

  // 和之前一样，把传进来的函数保存下来
  registerOnTouched(fn: (touched: boolean) => void) {
    this.onTouched = fn;
  }

  regionCodeChange(val: string) {
    // ...
    this.onTouched(true);
  }
  phoneNumChange(val: string) {
    // ...
    this.onTouched(true);
  }
}
```

# 最终代码

## `app-module.ts`

``` typescript
@NgModule({
  declarations: [
    AppComponent,
    PhoneNumInputComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## `app-component.html`

``` html
<form [formGroup]="form">
  <app-phone-num-input formControlName="phoneNum"></app-phone-num-input>
  <p *ngIf="this.form.get('phoneNum').touched && this.form.get('phoneNum').hasError('pattern')">
    Phone number shall start with +1!
  </p>
</form>

<p>Your phone number is: {{this.form.get('phoneNum').value}}</p>

<div>
  <button (click)="toggleFormDisable()">Toggle Disable</button>
</div>
```

## `app-component.ts`

``` typescript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.sass']
})
export class AppComponent implements OnInit {
  title = 'ng-custom-form-control-demo';
  form: FormGroup;
  formDisabled = false;

  constructor(private fb: FormBuilder) { }

  ngOnInit() {
    this.form = this.fb.group({
      phoneNum: [null, [Validators.pattern(/^\+1/)]]
    });

    this.form.get('phoneNum').setValue('+86-13912345678');
  }

  toggleFormDisable() {
    this.formDisabled = !this.formDisabled;
    if (this.formDisabled) {
      this.form.disable();
    } else {
      this.form.enable();
    }
  }
}
```

## `phone-num-input.component.html`

``` html
<select [ngModel]="regionCode"
        (ngModelChange)="regionCodeChange($event)">
  <option value="+86">+86</option>
  <option value="+1">+1</option>
</select>
<input type="tel"
       placeholder="Phone number..."
       [ngModel]="phoneNum"
       (ngModelChange)="phoneNumChange($event)" />
```

## `phone-num-input.component.ts`

``` typescript
export const APP_PHONE_NUM_INPUT_VALUE_ACCESSOR = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => PhoneNumInputComponent),
  multi: true
};

@Component({
  selector: 'app-phone-num-input',
  templateUrl: './phone-num-input.component.html',
  styleUrls: ['./phone-num-input.component.sass'],
  providers: [APP_PHONE_NUM_INPUT_VALUE_ACCESSOR]
})
export class PhoneNumInputComponent implements ControlValueAccessor {
  regionCode = '+86';
  phoneNum = '';
  onChange: (val: string) => void;
  onTouched: (touched: boolean) => void;
  disabled = false;

  registerOnChange(fn: (val: string) => void) {
    this.onChange = fn;
  }

  registerOnTouched(fn: (touched: boolean) => void) {
    this.onTouched = fn;
  }

  setDisabledState(isDisabled: boolean) {
    this.disabled = isDisabled;
  }

  writeValue(val: string) {
    const s = val.split('-');
    this.regionCode = s[0];
    this.phoneNum = s[1];
  }

  regionCodeChange(val: string) {
    this.regionCode = val;
    this.onChange(`${this.regionCode}-${this.phoneNum}`);
    this.onTouched(true);
  }
  phoneNumChange(val: string) {
    this.phoneNum = val;
    this.onChange(`${this.regionCode}-${this.phoneNum}`);
    this.onTouched(true);
  }
}
```
