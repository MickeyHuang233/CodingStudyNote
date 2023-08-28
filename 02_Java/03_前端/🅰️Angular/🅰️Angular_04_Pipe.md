# Angular Pipe(管道)
- `{{ 變量名 | 管道名 }}`，用於在View中呈現數據時顯示為另一種格式的函數
- 管道檔命名建議為`xxx.pipe.ts`，以便識別

## 🅰️建立管道(手動)
1. 創建組件 Class
	```js
	import { Pipe, PipeTransform } from '@angular/core';
	
	@Pipe({
	    name: 't27gender' // 管道名稱
	})
	export class T27GenderPipe implements PipeTransform{
	    // 管道執行過濾任務是使用固定的函數
	    transform(value: any, args?: any): any {
	        console.log(args)
	        if (value == 0) {
	            return 'male'
	        } else if (value == 1) {
	            return 'female'
	        }
	        return 'unknow';
	    }
	}
	```
2. 在模塊注冊組件
	```js
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { FormsModule } from '@angular/forms';
	// 引入組件
	import { AppComponent } from './app.component';
	import { T27c01Component } from './t27c01/t27c01.component';
	import { T27GenderPipe } from './t27gender.pipe';
	
	@NgModule({
	  // 聲明組件
	  declarations: [
	    AppComponent,
	    T27c01Component,
	    T27GenderPipe
	  ],
	  imports: [
	    BrowserModule,
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```

## 🅰️新建管道(NPM工具)
`[npx] ng generate pipe <管道名稱>`，使用npm(第三方模塊維護工具)建立Angular管道

```
PS D:\01_MickeyHuang\02_codeProject\20230815_AngularTry\my-app> npx ng g pipe t28p01
CREATE src/app/t28p01.pipe.spec.ts (187 bytes)
CREATE src/app/t28p01.pipe.ts (217 bytes)
UPDATE src/app/app.module.ts (1296 bytes)
```

## 🅰使用管道
`{{ 變量名 | 管道名 : 管道參數 }}`，用於HTML綁定
`[屬性名]="變量名 | 管道名 : 管道參數"`，用於屬性綁定

```js
import { Component } from '@angular/core';

@Component({
  selector: 'app-t27c01',
  templateUrl: './t27c01.component.html',
  styleUrls: ['./t27c01.component.css']
})
export class T27c01Component {
  employeeList = [
    {"id":1, "name":"Mickey", "gender":1, "salary":5000, "department":"PG"},
    {"id":2, "name":"Molly", "gender":1, "salary":6500, "department":"Marketing"},
    {"id":3, "name":"Will", "gender":0, "salary":3500, "department":"Marketing"},
    {"id":4, "name":"Anny", "gender":1, "salary":5500, "department":"System Analyze"},
    {"id":2, "name":"Tai", "gender":0, "salary":5000, "department":"PG"}
    ]
}
```

```html
<table>
    <thead>
        <tr>
            <th>Id</th>
            <th>Employee Id</th>
            <th>Name</th>
            <th>Gender</th>
            <th>Salary</th>
            <th>Department</th>
        </tr>
    </thead>
    <tbody>
        <tr *ngFor="let employee of employeeList; let index = index">
            <td>{{ index }}</td>
            <td>{{ employee.id }}</td>
            <td>{{ employee.name }}</td>
            <!-- 使用管道 -->
            <!-- {{ 變量名 | 管道名 : 管道參數 }} -->
            <td>{{ employee.gender | t27gender : ['test','123'] }}</td>
            <td>{{ employee.salary }}</td>
            <td>{{ employee.department }}</td>
        </tr>
    </tbody>
</table>
```

## 🅰Angular內置管道
[所有管道 - Angular API參考手冊](https://angular.tw/api?type=pipe)

- `{{ value_expression | lowercase }}`，字母全小寫
- `{{ value_expression | uppercase }}`，字母全大寫
- `{{ value_expression | titlecase }}`，首字母大寫
- `{{ value_expression | slice : start [ : end ] }}`，截取字串，相當於substring()
- `{{ value_expression | json }}`，轉為JSON字符串
- `{{ value_expression | number [ : digitsInfo [ : locale ] ] }}`，數字格式化，參數說明可參考：[DecimalPipe - Angular API參考手冊](https://angular.tw/api/common/DecimalPipe)
	```html
	{{ employee.salary | number }}
	{{ employee.salary | number : '4.2-2' }}
	```
- `{{ value_expression | currency [ : currencyCode [ : display [ : digitsInfo [ : locale ] ] ] ] }}`，數字格式化為貨幣字符串
	```html
	{{ employee.salary | currency }}
	{{ employee.salary | currency : '&&' }} <!-- 指定貨幣符號 -->
	```
- `{{ value_expression | date [ : format [ : timezone [ : locale ] ] ] }}`，格式化為日期字符串，參數說明可參考：[DatePipe - Angular API參考手冊](https://angular.tw/api/common/DatePipe)
	```html
	{{ employee.birthday | date }}
	{{ employee.birthday | date : 'yyyy-MM-dd HH:mm:ss' }} <!-- 指定日期時間格式 -->
	```
