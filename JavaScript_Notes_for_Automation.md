# JavaScript Notes for Test Automation (Playwright / WebdriverIO / Cucumber)

> Goal: cover exactly the JS you need before moving to Playwright, Cucumber, and WebdriverIO. Every topic has a plain explanation \+ runnable example. Use this as a revision sheet and as a base to write your own practice scripts.

---

## Table of Contents

1. [Variables: var, let, const](#1-variables-var-let-const)  
2. [Data Types](#2-data-types)  
3. [Operators](#3-operators)  
4. [Template Literals](#4-template-literals)  
5. [Control Flow](#5-control-flow)  
6. [Loops](#6-loops)  
7. [Functions](#7-functions)  
8. [Arrow Functions](#8-arrow-functions)  
9. [Arrays](#9-arrays)  
10. [Array Methods (very important for tests)](#10-array-methods-very-important-for-tests)  
11. [Objects](#11-objects)  
12. [Destructuring](#12-destructuring)  
13. [Spread & Rest Operators](#13-spread--rest-operators)  
14. [Optional Chaining & Nullish Coalescing](#14-optional-chaining--nullish-coalescing)  
15. [Scope & Closures](#15-scope--closures)  
16. [`this` Keyword](#16-this-keyword)  
17. [Classes (OOP) — core of Page Object Model](#17-classes-oop--core-of-page-object-model)  
18. [Error Handling: try/catch/finally](#18-error-handling-trycatchfinally)  
19. [Asynchronous JavaScript](#19-asynchronous-javascript)  
    - [19.1 Callbacks](#191-callbacks)  
    - [19.2 Promises](#192-promises)  
    - [19.3 async/await](#193-asyncawait)  
20. [JSON](#20-json)  
21. [Modules: import/export](#21-modules-importexport)  
22. [Node.js & npm Basics](#22-nodejs--npm-basics)  
23. [Timing Functions](#23-timing-functions)  
24. [Why This Matters for Playwright / WebdriverIO / Cucumber](#24-why-this-matters-for-playwright--webdriverio--cucumber)

---

## 1\. Variables: var, let, const

JavaScript has three ways to declare variables.

```javascript
var name = "old style";   // function-scoped, avoid using this
let age = 25;              // block-scoped, can be reassigned
const city = "Chennai";    // block-scoped, CANNOT be reassigned
```

- Use **`const`** by default (most values in test scripts don't change — URLs, selectors, expected results).  
- Use **`let`** when the value will change (like a loop counter or a variable you update).  
- Avoid **`var`** — it leaks out of blocks (`if`, `for`) and causes bugs.

```javascript
if (true) {
  var x = 10;
  let y = 20;
}
console.log(x); // 10  (leaked out of the block)
console.log(y); // Error: y is not defined (stayed inside the block)
```

**Note:** `const` prevents reassignment, not mutation:

```javascript
const user = { name: "Ravi" };
user.name = "Kumar"; // ✅ allowed, object contents can change
user = {};           // ❌ Error, cannot reassign the variable itself
```

---

## 2\. Data Types

```javascript
let str   = "hello";        // String
let num   = 42;             // Number
let dec   = 3.14;           // Number (no separate "float" type)
let bool  = true;           // Boolean
let arr   = [1, 2, 3];      // Array (object type)
let obj   = { a: 1 };       // Object
let undef;                  // undefined (declared, no value assigned)
let empty = null;           // null (intentionally empty)
let big   = 10n;            // BigInt
let sym   = Symbol("id");   // Symbol
```

Check a type using `typeof`:

```javascript
console.log(typeof "test");   // "string"
console.log(typeof 42);       // "number"
console.log(typeof true);     // "boolean"
console.log(typeof undefined);// "undefined"
console.log(typeof {});       // "object"
console.log(typeof []);       // "object"  (arrays are objects!)
console.log(Array.isArray([])); // true -> correct way to check for array
```

---

## 3\. Operators

```javascript
// Arithmetic
5 + 2   // 7
5 - 2   // 3
5 * 2   // 10
5 / 2   // 2.5
5 % 2   // 1 (remainder / modulus)
5 ** 2  // 25 (exponent)

// Comparison
5 == "5"   // true  -> loose equality (avoid, does type conversion)
5 === "5"  // false -> strict equality (ALWAYS prefer this in test code)
5 !== "5"  // true

// Logical
true && false  // false (AND)
true || false  // true  (OR)
!true          // false (NOT)

// Assignment shorthand
let count = 0;
count += 1;   // count = count + 1
count -= 1;
count *= 2;
```

**Rule for automation code:** always use `===` and `!==`, never `==` / `!=`.

---

## 4\. Template Literals

Backticks (`` ` ``) let you embed variables directly in strings — very useful for building dynamic selectors, test data, and log messages.

```javascript
const username = "admin";
const env = "staging";

// Old way
console.log("Logging in as " + username + " on " + env);

// Template literal way
console.log(`Logging in as ${username} on ${env}`);

// Multi-line strings
const message = `
Test Case: Login
User: ${username}
Environment: ${env}
`;
```

Common in Playwright/WebdriverIO for dynamic selectors:

```javascript
const rowId = 5;
const selector = `#row-${rowId} .delete-btn`;
```

---

## 5\. Control Flow

```javascript
const status = "failed";

if (status === "passed") {
  console.log("Test passed ✅");
} else if (status === "failed") {
  console.log("Test failed ❌");
} else {
  console.log("Status unknown");
}

// Ternary operator (shorthand if-else)
const result = status === "passed" ? "OK" : "NOT OK";

// switch statement
switch (status) {
  case "passed":
    console.log("Green");
    break;
  case "failed":
    console.log("Red");
    break;
  default:
    console.log("Grey");
}
```

---

## 6\. Loops

```javascript
// for loop
for (let i = 0; i < 5; i++) {
  console.log(`Iteration ${i}`);
}

// while loop
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}

// for...of  -> use for arrays (most common in test data iteration)
const users = ["admin", "guest", "manager"];
for (const user of users) {
  console.log(user);
}

// for...in -> use for object keys
const config = { browser: "chrome", headless: true };
for (const key in config) {
  console.log(key, config[key]);
}
```

**In real test scripts**, `for...of` is used constantly to loop over test data sets (e.g., running the same test with multiple users).

---

## 7\. Functions

```javascript
// Function declaration
function login(username, password) {
  console.log(`Logging in with ${username}/${password}`);
}
login("admin", "1234");

// Function with default parameter
function openBrowser(browserName = "chrome") {
  console.log(`Opening ${browserName}`);
}
openBrowser();          // Opening chrome
openBrowser("firefox"); // Opening firefox

// Function expression
const logout = function () {
  console.log("Logged out");
};

// Function returning a value
function add(a, b) {
  return a + b;
}
const sum = add(2, 3); // 5
```

---

## 8\. Arrow Functions

Shorter syntax, and **does not have its own `this`** (important — see [Section 16](#16-this-keyword)).

```javascript
// Regular function
function square(x) {
  return x * x;
}

// Arrow function equivalent
const squareArrow = (x) => x * x;

// Multiple parameters
const add = (a, b) => a + b;

// No parameters
const greet = () => console.log("Hello!");

// Multi-line arrow function needs braces + explicit return
const multiply = (a, b) => {
  const result = a * b;
  return result;
};
```

You'll see arrow functions everywhere in Playwright/WebdriverIO test files:

```javascript
test('login test', async ({ page }) => {
  await page.goto('https://example.com');
});
```

---

## 9\. Arrays

```javascript
const browsers = ["chrome", "firefox", "edge"];

browsers[0];          // "chrome"
browsers.length;       // 3
browsers.push("safari");     // adds to end
browsers.pop();               // removes from end
browsers.unshift("opera");   // adds to start
browsers.shift();             // removes from start
browsers.includes("chrome"); // true
browsers.indexOf("firefox"); // 1
```

---

## 10\. Array Methods (very important for tests)

These are used constantly to process lists of elements, table rows, API responses, and test data.

```javascript
const numbers = [1, 2, 3, 4, 5];

// map -> transforms every element, returns NEW array
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10]

// filter -> keeps elements that pass a condition
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4]

// forEach -> just loops, does NOT return a new array
numbers.forEach(n => console.log(n));

// find -> returns the FIRST matching element (or undefined)
const found = numbers.find(n => n > 3);
// 4

// some -> true if AT LEAST ONE element matches
const hasEven = numbers.some(n => n % 2 === 0); // true

// every -> true if ALL elements match
const allPositive = numbers.every(n => n > 0); // true

// reduce -> reduces array to a single value
const total = numbers.reduce((sum, n) => sum + n, 0);
// 15

// sort
const sorted = [...numbers].sort((a, b) => b - a); // descending
```

**Real automation example** — checking if an expected user is present in a list fetched from the UI:

```javascript
const displayedUsers = ["Ravi", "Kumar", "Anita"];
const isPresent = displayedUsers.includes("Anita"); // true

const activeUsers = displayedUsers.filter(name => name !== "Kumar");
// ["Ravi", "Anita"]
```

---

## 11\. Objects

```javascript
const testCase = {
  id: "TC001",
  title: "Verify login",
  priority: "High",
  isAutomated: true,
  run: function () {
    console.log(`Running ${this.title}`);
  }
};

testCase.title;          // dot notation
testCase["priority"];    // bracket notation (needed for dynamic keys)
testCase.run();          // calling a method

// Adding / updating properties
testCase.status = "Passed";
testCase.priority = "Medium";

// Delete a property
delete testCase.isAutomated;

// Looping through an object
Object.keys(testCase);    // ["id", "title", ...]
Object.values(testCase);  // ["TC001", "Verify login", ...]
Object.entries(testCase); // [["id","TC001"], ["title","Verify login"], ...]
```

---

## 12\. Destructuring

Extract values out of objects/arrays into separate variables. Used **everywhere** in Playwright/WebdriverIO (e.g. `{ page }`, `{ browser }`).

```javascript
// Object destructuring
const user = { name: "Ravi", role: "Admin", age: 30 };
const { name, role } = user;
console.log(name); // "Ravi"
console.log(role); // "Admin"

// Rename while destructuring
const { name: userName } = user;

// Default value if property doesn't exist
const { country = "India" } = user;

// Array destructuring
const coordinates = [10, 20];
const [x, y] = coordinates;

// Common Playwright pattern
async function test({ page, browser }) {
  await page.goto("https://example.com");
}
```

---

## 13\. Spread & Rest Operators

Both use `...` but work differently.

```javascript
// SPREAD -> expands/unpacks elements
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }

// Copying arrays/objects (avoids mutating original)
const original = [1, 2, 3];
const copy = [...original];

// REST -> collects remaining items into an array
function logAll(first, ...rest) {
  console.log(first);  // first value
  console.log(rest);   // array of everything else
}
logAll(1, 2, 3, 4); // 1  [2, 3, 4]
```

---

## 14\. Optional Chaining & Nullish Coalescing

Extremely useful when handling API responses or UI data that might be missing.

```javascript
const response = {
  user: {
    name: "Ravi"
  }
};

// Optional chaining (?.) -> avoids error if a property doesn't exist
console.log(response.user?.name);    // "Ravi"
console.log(response.user?.email);   // undefined (no error thrown)
console.log(response.address?.city); // undefined (no error, even though 'address' doesn't exist)

// Nullish coalescing (??) -> gives a fallback ONLY for null/undefined
const timeout = null;
const finalTimeout = timeout ?? 5000; // 5000

const retries = 0;
const finalRetries = retries ?? 3; // 0 (because 0 is not null/undefined, unlike ||)
```

---

## 15\. Scope & Closures

**Scope** \= where a variable is accessible.

```javascript
function outer() {
  let counter = 0; // local to outer()

  function inner() {
    counter++; // inner "remembers" counter even after outer() finishes
    return counter;
  }

  return inner;
}

const increment = outer();
console.log(increment()); // 1
console.log(increment()); // 2
console.log(increment()); // 3
```

This is a **closure**: `inner()` keeps access to `counter` from its parent scope even though `outer()` has already returned. Useful for things like keeping a counter of retries or step numbers in a test.

---

## 16\. `this` Keyword

`this` refers to "who is calling the function" — and it behaves differently for regular functions vs arrow functions.

```javascript
const testSuite = {
  name: "Login Suite",
  regularFunction: function () {
    console.log(this.name); // "Login Suite" -> works, 'this' = testSuite
  },
  arrowFunction: () => {
    console.log(this.name); // undefined -> arrow functions don't bind their own 'this'
  }
};

testSuite.regularFunction();
testSuite.arrowFunction();
```

**Rule of thumb:** use regular functions (or `function` methods) when you need `this` to refer to the object; use arrow functions for short callbacks (like inside `.map()`, `.filter()`) where you don't need `this`.

---

## 17\. Classes (OOP) — core of Page Object Model

Playwright and WebdriverIO test frameworks are built around the **Page Object Model (POM)** — each page of the app is represented as a class.

```javascript
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameField = "#username";
    this.passwordField = "#password";
    this.loginButton = "#loginBtn";
  }

  async login(username, password) {
    await this.page.fill(this.usernameField, username);
    await this.page.fill(this.passwordField, password);
    await this.page.click(this.loginButton);
  }
}

// Usage in a test
const loginPage = new LoginPage(page);
await loginPage.login("admin", "1234");
```

Key class concepts:

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

// Inheritance
class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks`);
  }
}

const d = new Dog("Rex");
d.speak(); // "Rex barks"
```

---

## 18\. Error Handling: try/catch/finally

Critical for tests — you often need to handle elements that might not appear, or API calls that might fail.

```javascript
try {
  const el = document.querySelector(".missing-element");
  el.click(); // will throw if el is null
} catch (error) {
  console.log("Something went wrong:", error.message);
} finally {
  console.log("This always runs, pass or fail");
}

// Throwing your own errors
function validateAge(age) {
  if (age < 18) {
    throw new Error("User must be an adult");
  }
  return true;
}

try {
  validateAge(15);
} catch (e) {
  console.log(e.message); // "User must be an adult"
}
```

---

## 19\. Asynchronous JavaScript

**This is the single most important section for Playwright/WebdriverIO.** Almost every browser action (`click`, `fill`, `goto`, `waitFor`) is asynchronous, because the browser needs time to respond.

### 19.1 Callbacks

The oldest way to handle async code — a function passed into another function, to be called later.

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback("data loaded");
  }, 1000);
}

fetchData((result) => {
  console.log(result); // "data loaded" (after 1 second)
});
```

Callbacks get messy when chained repeatedly ("callback hell"), which is why Promises were introduced.

### 19.2 Promises

A `Promise` represents a value that will be available now, later, or never (error). A promise has 3 states: **pending → fulfilled** or **pending → rejected**.

```javascript
function fetchUser() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) {
        resolve({ name: "Ravi" });
      } else {
        reject("Failed to fetch user");
      }
    }, 1000);
  });
}

fetchUser()
  .then((user) => console.log(user.name))   // runs if resolved
  .catch((err) => console.log(err))          // runs if rejected
  .finally(() => console.log("Done"));       // always runs
```

### 19.3 async/await

Modern, cleaner way to work with Promises — reads like normal, synchronous code. **This is exactly what you'll write in Playwright/WebdriverIO scripts.**

```javascript
function fetchUser() {
  return new Promise((resolve) => {
    setTimeout(() => resolve({ name: "Ravi" }), 1000);
  });
}

async function main() {
  console.log("Start");
  const user = await fetchUser(); // pauses here until the promise resolves
  console.log(user.name);         // "Ravi"
  console.log("End");
}

main();
```

Error handling with async/await uses try/catch:

```javascript
async function login() {
  try {
    const user = await fetchUser();
    console.log(`Logged in as ${user.name}`);
  } catch (error) {
    console.log("Login failed:", error);
  }
}
```

**This is exactly the shape of a Playwright test:**

```javascript
const { test, expect } = require('@playwright/test');

test('user can log in', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'admin');
  await page.fill('#password', '1234');
  await page.click('#loginBtn');
  await expect(page.locator('.welcome-msg')).toBeVisible();
});
```

And a WebdriverIO test looks almost identical:

```javascript
describe('Login Test', () => {
  it('should log in successfully', async () => {
    await browser.url('https://example.com/login');
    await $('#username').setValue('admin');
    await $('#password').setValue('1234');
    await $('#loginBtn').click();
    await expect($('.welcome-msg')).toBeDisplayed();
  });
});
```

Notice: **every browser action is preceded by `await`**, and the enclosing function is `async`. If you understand Section 19 well, the rest is just learning the library's specific methods (`.fill()`, `.click()`, `$()`, etc.).

**Running multiple promises together:**

```javascript
async function runParallel() {
  const [a, b] = await Promise.all([fetchUser(), fetchUser()]);
  console.log(a, b);
}
```

---

## 20\. JSON

Test data, API responses, and config files are almost always in JSON format.

```javascript
const user = { name: "Ravi", age: 30 };

// Convert JS object -> JSON string (e.g. to send in an API request)
const jsonString = JSON.stringify(user);
console.log(jsonString); // '{"name":"Ravi","age":30}'

// Convert JSON string -> JS object (e.g. parsing an API response)
const parsedObject = JSON.parse(jsonString);
console.log(parsedObject.name); // "Ravi"
```

---

## 21\. Modules: import/export

Test projects are split across multiple files (page objects, step definitions, config, utilities). Modules let you share code between files.

```javascript
// file: mathUtils.js
export function add(a, b) {
  return a + b;
}
export const PI = 3.14;

// default export (only one per file)
export default function multiply(a, b) {
  return a * b;
}
```

```javascript
// file: app.js
import multiply, { add, PI } from './mathUtils.js';

console.log(add(2, 3));      // 5
console.log(multiply(2, 3)); // 6
console.log(PI);             // 3.14
```

**CommonJS style** (older, still common in Node.js/WebdriverIO configs):

```javascript
// exporting
module.exports = { add, multiply };

// importing
const { add, multiply } = require('./mathUtils');
```

---

## 22\. Node.js & npm Basics

Playwright and WebdriverIO run on **Node.js**, so you need to know the basic project setup commands.

```shell
node -v                     # check Node.js version
npm init -y                 # create a package.json for your project
npm install playwright      # install a package
npm install --save-dev cucumber
npm run test                # run a script defined in package.json
```

`package.json` structure (you'll edit this often):

```json
{
  "name": "my-automation-project",
  "version": "1.0.0",
  "scripts": {
    "test": "wdio run wdio.conf.js"
  },
  "dependencies": {
    "playwright": "^1.45.0"
  },
  "devDependencies": {
    "@wdio/cli": "^8.0.0"
  }
}
```

---

## 23\. Timing Functions

Understanding these will help you understand explicit/implicit waits later.

```javascript
// setTimeout -> runs once, after a delay
setTimeout(() => {
  console.log("Runs after 2 seconds");
}, 2000);

// setInterval -> runs repeatedly
const interval = setInterval(() => {
  console.log("Runs every 1 second");
}, 1000);

// stop it
clearInterval(interval);
```

> **Note:** In Playwright/WebdriverIO, you almost never manually use `setTimeout` for waits — both frameworks provide smarter built-in waits (`waitForSelector`, `waitUntil`, auto-waiting) that wait for the *condition* to be true instead of a fixed time. But understanding this section helps you understand *why* those smarter waits exist.

---

## 24\. Why This Matters for Playwright / WebdriverIO / Cucumber

| JS Concept | Where you'll use it |
| :---- | :---- |
| `async/await`, Promises | Every single browser action (`click`, `fill`, `goto`) is asynchronous |
| Classes | Page Object Model — one class per page/screen |
| Arrow functions | Test blocks: `test('...', async () => {...})`, `it('...', async () => {...})` |
| Destructuring | `async ({ page }) => {...}` in Playwright fixtures |
| Array methods (`map`, `filter`, `forEach`) | Looping through table rows, dropdown options, API results |
| Template literals | Building dynamic selectors and readable log/report messages |
| try/catch | Handling flaky elements, optional popups, retries |
| Modules (import/export) | Splitting code into Page Objects, Step Definitions, Utils, Config |
| JSON | Test data files, API request/response bodies |
| Objects | Storing test data, config, locators |

---

## Suggested Practice Order

1. Variables, data types, operators, template literals  
2. Functions \+ arrow functions  
3. Arrays \+ array methods (spend extra time here)  
4. Objects \+ destructuring \+ spread/rest  
5. **Async/await \+ Promises** (spend the MOST time here — it's the foundation of Playwright/WebdriverIO)  
6. Classes (build 2–3 dummy "Page Object" classes as practice)  
7. Modules (split your practice code into separate files and import them)  
8. Then move to Playwright basics — you'll recognize almost every syntax pattern already.

---

*Tip: Ask me to generate small practice exercises for any section above (e.g. "give me 5 practice problems on array methods with answers"), or paste your own code here and I'll review/debug it.*  
