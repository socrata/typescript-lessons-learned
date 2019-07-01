# Compiler Options

### strictNullChecks: true

`null` and `undefined` are types in Typescript. By default, they are in the domain of all other types. This means the compiler lets you do this: `const foo: number = undefined` .

Enabling `strictNullChecks`  disallows this. If something is nullable, you have to say so explicitly, with either a union type or an optional property. And because of the explicit declaration, the compiler now forces you to guard against potential nullables:

```typescript
interface Foo {
  bar: string;
  baz?: number;
}

function badPlusTen(foo: Foo): number {
  // error "foo.baz" is possibly "undefined" 
  return foo.baz + 10;
}

function goodPlusTen(foo: Foo): number {
  if (foo.baz) {
    return foo.baz + 10;
  } else {
    return 10;
  }
}
```

This turns the compiler into an essential refactoring tool, especially in React apps where it's easy too lose track of potential nulls and undefineds as you pass data from parent to children.

### noImplicitAny: true

By default, when the Typescript compiler can't infer a type for some variable, it assigns that variable the type `any`. This reduces compiler errors but decreases type safety, thereby decreasing the benefit of using Typescript in general. We chose to disallow this behavior with one major exception: the compiler can still infer `any` for a file's imports. This is done by adding `declare module '*'`  in a `declarations.d.ts` file at the project's root. This makes it much easier to gradually add types to an existing JavaScript codebase because it allows you to focus on eliminating implicit `any` types only within the file you are converting, without worrying about that file's imports, and those imports' imports, and so on.  

### moduleResolution: "Node"

The official documentation has an [excellent article](https://www.typescriptlang.org/docs/handbook/module-resolution.html) explaining module resolution. One reason we chose "Node" over "Classic" was to better handle declaration files that were included in a library outside the `@types` folder in `node_modules`, e.g. Redux. "Classic" resolution would miss these files, leading to unnecessary type errors.

We do tweak module resolution behavior to accord with existing import methods. This is done in the tsconfig file here:

```json
baseUrl": ".",
"paths": {
  "*": ["./frontend/public/javascripts/*", "./common/*", "./frontend/node_modules"]
}
```

This only affect absolute imports. It tells the compiler: "when you come across an absolute import that matches the pattern *--so, every absolute import-- first look in `./frontend/public/javascripts`, then `./common`,  then finally `node_modules`". The `baseUrl` tells the compiler where to start its search from. The `.` value just means start from whatever directory contains the `tsconfig.js` file. This all to inform the compiler that when we write `import Foo from 'common/Foo'`, we probably don't mean a library called `common` in `node_modules`.

