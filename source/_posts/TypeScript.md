---
title: TypeScript 基础入门
---
# TypeScript

### 基本类型(基础对象类型)

```tsx
// array
let arrOfNumber: number[] = [1,2,3]

// tuple (原组)
let user: [string, number] = ['viking', 20]

// function
function add(x: number, y: number): number {
	return x + y
}

let add2 = (x: number, y: number): number => { return x + y } 
```

### interface 用来定义对象类型

```tsx

// 可选属性
interface Animal {
	color?: string;
}

// 只读属性
interface Animal {
	readonly color: string;
}

// 额外的属性检查
interface Animal {
	color: string;
	[propName: string]: string; // 其他属性类型必须是string
}

// 可索引的类型 
interface StringArray {
	[index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];

// 函数类型
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

### 泛型： 函数使用时决定传入的参数类型 （而非定义函数时指定对应类型）

```tsx
// 泛型在函数中的使用
function echo<T>(arg: T): T {
	return arg;
}

let result = echo(123);
result = echo('123');

// 结合interface使用
interface GithubResp {
	name: string;
	count: number;
}

// 使用泛型的方法定义
function withAPI<T>(url: string): Promise<T> {
	return fetch(url).then(resp => resp.json());
}

// 调用方法传入泛型
withAPI<GithubResp>('github.user').then((resp) => {
	// resp 可访问到  GithubResp定义的类型
	console.log(resp.name, resp.count);
})
```

### 类型别名:类型别名用来给一个类型起个新名字

```tsx
// 类型别名 type
interface Persion {
	name: string;
	age: string;
}

type PersionOptional = Partial<Persion>

let viking: PersionOptional = {name: 'json'} 

```

### 交叉类型： 将几种类型合并起来

```tsx
//交叉类型 使用 '&' 连接符 
interface A {
	color: string;
}
type B = A & { name: string }

//使用B类型
let box: B = {color: 'red', name: 'cc'}
```

### 联合类型： 标识或的关系

```tsx
// 联合类型
let numberOrString: number | string;
```

### 类型断言： 用来处理联合类型 里非共有部分的逻辑

```tsx
//类型断言 as
function getLength(input: number | string) {
	const str = input as string
	if (str.length) {
		return str.length;
	} else {
		const number = input as number;
		return number.troString().length;	
	}
}
```

 

### keyof **获取某种类型的所有键，其返回类型是联合类型**

```tsx
interface Box {
	name: string;
	color: string;
}
// keyof 取健值
type Keys = keyof Box // type keys = "name" | "color"
let key: Keys = 'name'

// lookup types 取类型值
type NameType = Box['name'] // type NameType = string

// mapped types： in 循环复制类型
type Test = {
	[key in Keys]: Box[key] // type Test = {name: string; color:string; }
}
```

### extends in generics(泛型)

```tsx
// 用法1
interface IwithLength {
	length: number;
}

// 解决T是动态传入的有可能不支持length属性 所以预选扩充一个这样的属性
function echoWithArr<T extends IwithLength>(arr: T):T {
	console.log(arr.length);
	return arr;
}

// 方法调用
const str = echoWithArr('123');
const arrs = echoWithArr([1,2,3])
const obj = echoWithArr({length: 123})

// 用法2
// extends 表条件类型
type NonType<T> = T extends null | undefined ? never : T

// 类型使用
let demo1: NonType<number> // let demo1: number
let demo2: NonType<null> // let demo2: never
```

### 声明文件 xxx.d.ts （如果源代码就是用ts开发的就不需要该文件）

```tsx

// fetch.js
function myFecth(url, method, data) {
	return fetch(url, {
		body: data ? JSON.stringify(data) : '',
		method
	}).then((resp) => resp.json())
}

myFetch.get = (url) => myFetch(url , 'GET')
myFetch.post = (url) => myFetch(url , 'POST', data)

export default myFetch;

// .d.ts 写注释 针对原来js文件 不支持TS （如果源代码就是用ts开发的就不需要该文件）
// .d.ts文件 (install后)所在的位置 ./node_modules/@types/fetch/index.d.ts
type HTTPMethod = 'GET' | 'POST'
declare function myFetch<T = any>(url: string, method: HTTPMethod, data?: any) : Promise<T>
// 定义一个命名空间
declare namespace myFetch {
	const get: <T = any>(url: string) => Promise<T>;
	const post: <T = any>(url: string, data: any) => Promise<T>
}

// 导出
export = myFetch

// 调用
import myFetch from 'myFetch'

myFetch<string>('test.com', 'POST', {name: 'hello'}).then((data) => {
	// ...
})

myFetch.get<number>('test.com').then(data => {
	// ...
})
```