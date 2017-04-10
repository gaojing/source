title: typescript basic types
tags: typescript
---

### 简介
TypeScript支持JavaScript中同样的数据类型，还添加了枚举类型。

### Boolean
	let isDone: boolean = false;

### Number
	let decimal: number = 6;
	let hex: number = 0xf00d;
	let binary: number = 0b1010;
	let octal: number = 0o744;

### String
	let color: string = "blue";
模板字符串
	let fullName: string = `Bob Bobbington`;
	let age: number = 37;
	let sentence: string = `Hello, my name is ${ fullName }.
	
	I'll be ${ age + 1 } years old next month.`

### Array
两种定义方式

-  类型后面跟[]

		let list: number[] = [1, 2, 3];

-  通用的array类型

		let list: Array<number> = [1, 2, 3];

### Tuple
	// Declare a tuple type
	let x: [string, number];
	// Initialize it
	x = ["hello", 10]; // OK
	// Initialize it incorrectly
	x = [10, "hello"]; // Error

### Enum
	enum Color {Red, Green, Blue};
	let c: Color = Color.Green;

### Any
	let notSure: any = 4;
	notSure = "maybe a string instead";
	notSure = false; // okay, definitely a boolean

### Void
	function warnUser(): void {
	    alert("This is my warning message");
	}

### Null and Undefined
	// Not much else we can assign to these variables!
	let u: undefined = undefined;
	let n: null = null;

### Never
