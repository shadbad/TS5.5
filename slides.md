---
layout: cover
class: "text-white"
---

# Whats up with TS 5.5?

- Inferred Type Predicates
- Support for New ECMAScript Set Methods
- Isolated Declarations
- The ${configDir} Template Variable for Configuration Files

---

### Inferred Type Predicates - The problem

```ts
interface Bird {
  commonName: string;
  scientificName: string;
  sing(): void;
}

declare const nationalBirds: Map<string, Bird>;

function makeNationalBirdCall(country: string) {
  const bird = nationalBirds.get(country);
  bird.sing(); // Error: 'bird' is possibly 'undefined'
}
```

---

### Inferred Type Predicates - The solution

```ts
interface Bird {
  commonName: string;
  scientificName: string;
  sing(): void;
}

declare const nationalBirds: Map<string, Bird>;

function makeNationalBirdCall(country: string) {
  const bird = nationalBirds.get(country);
  if (bird) bird.sing();
  else // bird has type undefined here.
}
```

---

### Inferred Type Predicates - The collection solution

```ts
interface Bird {
  commonName: string;
  scientificName: string;
  sing(): void;
}

declare const nationalBirds: Map<string, Bird>;

function makeNationalBirdCall(country: string) {
  const bird = nationalBirds.get(country);
  if (bird) bird.sing();
  else // do something with the undefined situation
}

function makeBirdCalls(countries: string[]) {
  const birds = countries
    .map(country => nationalBirds.get(country)) // (Bird | undefined)[]
    .filter(bird => bird !== undefined); // Bird[]

  for (const bird of birds) bird.sing();  // now TS can keep up with the filtering
}

```

---

### Inferred Type Predicates - What's going on under the hood

- We use type guards which are simple boolean functions that return true if the input variable type checks out
- Typescript now can infer if a function returns a type predict
- Here is the syntax for a type predicate:

```ts
function isType(variable: any): variable is SpecificType {
  // logic to determine if variable is SpecificType
}

// Before update TS infer that this function returns boolean
// But now it correctly infers that this is a type guard which returns a type predicate
function isType(variable: any) {
  // logic to determine if variable is SpecificType
}
```

---

### Inferred Type Predicates - When a function is a considered as a type guard?

- The function does not have an explicit return type or type predicate annotation.
- The function has a single return statement and no implicit returns.
- The function does not mutate its parameter.
- The function returns a boolean expression that’s tied to a refinement on the parameter.

#### Example

```ts
// Basically by passing a type guard to the filter not only we're filtering the elements
// inside the collection we're also changing the data type of the collection
// the inferred type predicates free us from writing explicit type guards and using them
// as input parameter of the filter function
const nums = [1, 2, 3, null, 5].filter((x) => x !== null);

nums.push(null); // ok in TS 5.4, error in TS 5.5
```

---

### Support for New ECMAScript Set Methods

```ts
let fruits = new Set(["apples", "bananas", "pears", "oranges"]);
let applesAndBananas = new Set(["apples", "bananas"]);
let applesAndOranges = new Set(["apples", "oranges"]);
let oranges = new Set(["oranges"]);
let emptySet = new Set();

console.log(fruits.union(oranges)); // Set(4) {'apples', 'bananas', 'pears', 'oranges'}
console.log(applesAndBananas.union(oranges)); // Set(3) {'apples', 'bananas', 'oranges'}

console.log(fruits.intersection(applesAndBananas)); // Set(2) {'apples', 'bananas'}
console.log(applesAndBananas.intersection(oranges)); // Set(0) {}
console.log(applesAndBananas.intersection(applesAndOranges)); // Set(1) {'apples'}
```

---

### Support for New ECMAScript Set Methods Pt.2

```ts
let fruits = new Set(["apples", "bananas", "pears", "oranges"]);
let applesAndBananas = new Set(["apples", "bananas"]);
let applesAndOranges = new Set(["apples", "oranges"]);
let oranges = new Set(["oranges"]);
let emptySet = new Set();

console.log(fruits.difference(oranges)); // Set(3) {'apples', 'bananas', 'pears'}
console.log(fruits.difference(applesAndBananas)); // Set(2) {'pears', 'oranges'}

console.log(applesAndBananas.difference(applesAndOranges)); // Set(1) {'bananas'}
console.log(applesAndBananas.symmetricDifference(applesAndOranges)); // Set(2) {'bananas', 'oranges'}

// difference: none shared members in the source set
// symmetricDifference: none shared members in both sets

console.log(applesAndBananas.isDisjointFrom(oranges)); // true
console.log(applesAndBananas.isDisjointFrom(applesAndOranges)); // false
console.log(fruits.isDisjointFrom(emptySet)); // true
console.log(emptySet.isDisjointFrom(emptySet)); // true
```

---

### Support for New ECMAScript Set Methods Pt.3

```ts
let fruits = new Set(["apples", "bananas", "pears", "oranges"]);
let applesAndBananas = new Set(["apples", "bananas"]);
let applesAndOranges = new Set(["apples", "oranges"]);
let oranges = new Set(["oranges"]);
let emptySet = new Set();

console.log(applesAndBananas.isSubsetOf(fruits)); // true
console.log(fruits.isSubsetOf(applesAndBananas)); // false
console.log(applesAndBananas.isSubsetOf(oranges)); // false
console.log(fruits.isSubsetOf(fruits)); // true
console.log(emptySet.isSubsetOf(fruits)); // true

console.log(fruits.isSupersetOf(applesAndBananas)); // true
console.log(applesAndBananas.isSupersetOf(fruits)); // false
console.log(applesAndBananas.isSupersetOf(oranges)); // false
console.log(fruits.isSupersetOf(fruits)); // true
console.log(emptySet.isSupersetOf(fruits)); // false
```

---

### Isolated Declarations - Type definition (declaration) files

- What are the benefits of generating type definitions (.d.ts) from a TS project and consume it in a dependent TS project?

  - Faster Type Checking
  - Reduced Build Times
  - Parallel Development

- How can I create a type definition file?
  - Set the "declaration" flag to true in the compilerOptions node of your tsconfig.json file
  - Run your compile script (tsc)
  - You can also use the declaration switch in the CLI --> tsc --declaration --outDir ./dist src/index.ts

---

### Isolated Declarations - d.ts generation issues

```ts
// util.ts
export let one = "1";
export let two = "2";

// add.ts
import { one, two } from "./util";
export function add() {
  return one + two;
}
```

The generator tool has to crawl the dependencies (util.ts) and infer the exported variables' types, in order to generate the following

```ts
// add.d.ts
export declare function add(): string;
```

- As you can imagine this process can become time consuming
- Wouldn't it be nice to leverage the parallel checking?

---

### Isolated Declarations - Parallel checking issues

In one word, circular dependencies

<img src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2024/04/5-5-beta-isolated-declarations-deps.png" alt="" style="width:33%;"/>

What if the generator tool could generate the d.ts files of each library without going into the rabbit hole and start crawling and inferring the dependencies' types like a psychopath?

- it would need some love from the developers
- we need to explicitly type all our module exports
- how TS is going to help us in this regard? kill surprise, we have a new kind of error!

---

### Isolated Declarations - Yet another error message

```ts
export function foo() {
  //              ~~~
  // error! Function must have an explicit
  // return type annotation with --isolatedDeclarations.
  return x;
}
```

```ts
import { add } from "./add";

const x = add("1", "2"); // no error on 'x', it's not exported.

export function foo(): string {
  return x;
}
```

Bottomline if you have these flags isolatedDeclarations, declaration or composite in your ts config file expect to see this error, and now you know why it exists.

---

### The ${configDir} Template Variable for Configuration Files - The issue

In a micro service or multi project situation, We usually extract the common TS configurations to a base file and extend this base
config file in each micro or project.

```json

{ // tsconfig.base.json
    "compilerOptions": {
        "typeRoots": [
            "./node_modules/@types"
            "./custom-types"
        ],
        "outDir": "dist"
    }
}
```

But the addresses are all relative to the location of the base file not the extending config file!

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

---

### The ${configDir} Template Variable for Configuration Files - The solution

the template variable ${configDir} always maps to the extender's root folder not the base config file

```json
{
    "compilerOptions": {
        "typeRoots": [
            "${configDir}/node_modules/@types"
            "${configDir}/custom-types"
        ],
        "outDir": "${configDir}/dist"
    }
}
```
