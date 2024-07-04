---
layout: cover
class: "text-white"
---

# Whats up with TS 5.5?

- Inferred Type Predicates
- Control Flow Narrowing for Constant Indexed Accesses
- Support for New ECMAScript Set Methods
- Isolated Declarations
- The ${configDir} Template Variable for Configuration Files
- Consulting package.json Dependencies for Declaration File Generation
- Editor and Watch-Mode Reliability Improvements
- Performance and Size Optimizations
- Easier API Consumption from ECMAScript Modules
- The transpileDeclaration API

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

### Control Flow Narrowing for Constant Indexed Accesses

```ts
function f1(obj: Record<string, unknown>, key: string) {
  if (typeof obj[key] === "string") {
    // Now okay, previously was error
    obj[key].toUpperCase();
  }
}
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
