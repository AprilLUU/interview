## TypeScript

#### interface和type的区别

- interface和type都可以用来声明一个类型，关键区别在于类型不能添加新属性，而接口始终是可扩展的，支持声明合并
- type适合声明交叉，联合，基本类型，声明函数类型

```typescript
type ID = string | number;
let userId: ID = "123";
userId = 456;

type Animal = { name: string };
type Dog = Animal & { bark(): void };
```

- interface支持声明合并，支持继承，更适合声明一个对象类型

```typescript
interface Person {
  name: string;
}
interface Person {
  age: number;
}
const p: Person = { name: "Alice", age: 25 }; 
```

- 官网推荐，优先使用interface声明（编译性能更高），当需要用到type的功能时再使用type