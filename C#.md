# eXomorph - C# Language Style Guide

This document discribes how code written in C# should be formatted and designed.

## Variable Naming

All variables should be written in camel case and adhere to the following prefex rules:
|Case                          |Format         |
|:----------------------------:|---------------|
| private/internal member      | `m_camelCase` |
| private/internal static      | `s_camelCase` |
| constant (any visibility)    | `k_camelCase` |

Public, protected, or public static variables should have no prefix. `m_camelCase` may be used for proteced member's if the non-prefix version of the name is required for a property name, in which case all protected members within that class must follow that convention.

Variable names must have meaning and must contain complete words. Word Shortening may be used in instances where the shortened form of the word is common within the gaming industry (such as `pos`, `vel`, etc...).

Single letter variable names should not be used except in the following cases:
  - `i`, `j`, `k` in `for` loops
  - `x`, `y`, `z` in `for` loops pertaining to coordinate positions
  - well known variable in a math expression (example: `t` in lerp equasions)
  - `a`, `b`, etc... may be used when a method or operator override takes 2 or more of the same type.

  *Note:* an index variable outside a `for` loop must be named `index` instead of `i`.

**Booleans:**
Along with the previous variable naming style, booleans must contain "is", "was", "has", or similar words within the name to help distinguice the `true` value state.

## Property Formatting

Properties should be written in camel case. If a property acts as a simple getter or setter, get and set properties can be written on the same line as the declaration. If the property contains any extra steps, they must be written with Allman style braces. `set` state is not required.
```c#
// Simple getter/setter
public int simpleProperty { get { return m_member; } set { m_member = value; }}

// With extra steps
public int complexProperty
{
    get
    {
        return m_member;
    }
    set
    {
        m_member = value;
        PerformNewValueCheck();
    }
}
```
Properties ___Must Not___ contain their own values (ie. do not use `{get; set;}` shortcut). Properties must assign or retrieve a private member or shortcut to some other form of internal implementation (ex. the size of an private list).

## Method Formatting

All methods should be in Pascal Case and follow Allman style braces. Explicitly writing `private` is optional for private methods, however is required in the case where there is a private override of a method that is protected, internal, or public.
```c#
void MethodName()
{
    // Body
}
```
## Structures

Classes and Structs must use Pascal Case naming and use meaningful names.

### Large Classes

*Large Class* denotes the classic "class" form.

*Large Classes* must not contain any public static or non-static variables. These values must be interfaced with through public properties or get and set functions.

Unless using *Unity*, public properties and methods must be on top, followed by, internal, protected, and then private methods. Member variable should be located at the botton. See the *Unity Style Guide* for *Unity* specifiec ordering

### Structs/Small Classes:

*Small Class* denotes a Class that is used in the classic "struct" form but is to be allocated on the heap and should be passed by refernce.

*Small Classes* may contain public non-static variables. These structures should only have a small number of helper-methods.

Variable must be at the top of the structure with helper-methods below.

## Method Calling

When calling or defining a method with many parameters, each parameter should be passed in on a new line. The opening parenthesis should be on the same line as the `new` keyword with the first parameter on a new line and the closing parenthesis in line with the last parameter.

```cs
// Defining methods many parameters
float QuadLerp(
    float start,
    float startHandle,
    float endHandle,
    float end,
    float t)
{
    // Implementation
}

// Calling methods with many Parameters
float result = QuadLerp(
    m_start,
    m_start + m_easeAmount,
    m_end - m_easeAmount,
    m_end,
    time);

```
The following rules apply to constructors as well.

## Code Blocks
All other code blocks must use Allman style braces. `if` statements acting as a "guard" may forgo braces all together if only a single line. If an `if` or `while` statement is a single line, it must still use braces.

`else if` and `else` keywords must *not* be placed on the same line as the previous close-brace.

```cs
// Normal `if` code block
if (isValid)
{
    DoImplementation();
}
else
{
    OtherImplementation();
}

// Guard statement with single line
if (!isValid)
    return;
```

## Callbacks (Actions & Delegates)

Use Actions or delegates as much as possible to reduce dependancies.
Delegates should be used over Actions when parameters are non-trival.

Delegates should also be used in cases where more than one parameter is necissary. A CallbackContext struct or class is required to pack all values in this case.

Callbacks must use camel case and have "on" within their name to signal that they are callback. Actions and delegates may be public, but must always be use in a "subscribe" and "unsubscribe" fashion in this case (using `+=` and `-=` operators). If the callback is to only support one assigned outcome, the callback must be private and the executed method should be assigned via a method of the same name (using Pascal Case).

Always use "nullable" (`?`) character when calling `Invoke()`.

```cs
// Void/Simple callbacks
public class MyClass
{
    public Action onComplete;
    public Action<float> onHealthChange;
}

// Usage
myClass.onComplete += DoSomething;
myClass.onHealthChange += UpdateHealthVisual;
```
```cs
// Complex Callbacks
public class MyClass
{
    public CallbackContext
    {
        Vector2 position;
        Vector2 velocity;
    }
    public delegate MoveEvent;
    public MoveEvent onMove;
}

// Usage
public void Initialize()
{
    myClass.onMove += OnPlayerUpdate;
}

void OnPlayerUpdate(MyClass.CallbackContext ctx)
{
    // Implementation
}

```
```cs
// Single-call callbacks
public class MyClass
{
    private Action m_onComplete;
    public void OnComplete(Action onComplete)
    {
        m_onComplete = onComplete;
    }
    public void Execute()
    {
        // Use `?` when invoking
        onComplete?.Invoke();
    }
}
```

## Operation Formatting

In the case of multiline math or logic statements, the statement can begin  the operator must be places on the new line ahead of the second value. Parenthesis should be added to show explicit order for clarity (No relying on the order of operations)

```cs
// Math Statement
Vector2 squareMagnitude = (positionA.x * positionA.x)
    + (positionA.y * positionB.y);

// Logic Statement
if (boxA.left < boxB.right
    && boxA.right > boxB.left
    && boxA.top > boxB.bottom
    && boxA.bottom < boxB.top;)
{
    boxA.OnCollision(boxB);
    boxB.OnCollision(boxA);
}
```

Standalone *boolean* equasions bust be wrapped in parenthesis.

```cs
public bool isEmpty
{
    get
    {
        return (m_internalList.Count <= 0);
    }
}
```

## Ternary Statements

*Ternary Statements* are allowed, but not nested or placed inline within another statement.

```cs
// Store value first, then use it
int result = (index <= 0) ? 0 : index;
UsingResultAfter(result);
```

## Type-Infering Keywords

Having a variables type viewable is important to maintaining code readablilty.

- `var` is not to be used.

- `new()` may only be used when being called inline with a declaration where the type is visible.

```cs
// `new()` may be called
List<int> myList = new();

// Typed constructor must be called
m_myList = new List<int>();
```

## Execution Flow Keywords

Always prefer "early-out" execution flow. Test for invalid cases and use `return`, `continue`, or `break` skip remaining code.

`goto` is not to be used.

## Switch Statements

Switch statement formatting should have one set of braces in Allman style. The `case` keyword should be have indenting in line with the braces and implementation should be placed on new lines with one intent space. A `default` case should always be present, if only to Debug that there is no implementation for the provided value.

```cs
switch (testValue)
{
case 0:
    Impl0();
    break;
case 1:
case 2:
    Impl1And2();
    break;
default:
    System.Console.WriteLine("Test value not implemented");
    break;
}
```

Always use `switch` over `if` if value can be const-evaluated.

## Assertions

Assert or Error Log any and all expected values (If Initialization assertions are present, subsiquent asserts can be skipped)

## Scripting Defines
Defines should be all-caps and snake-case.

`#else` and `#endif` blocks should be commented with original `#if` value
```cs
// Example of a positive `#if` check
#if ANDROID_BUILD
// Implementation
#else // ANDROID_BUILD
// Else Implementation
#endif // ANDROID_BUILD

// Example of negative `#if` check
#if !ANDROID_BUILD
// Implementation
#else // !ANDROID_BUILD
// Else Implementation
#endif // !ANDROID_BUILD
```

## Comments

Comments should have a space afer the "//" unless they are used to comment-out code.

Todo comments should be formatted as follows "// TODO - Content"
```cs
// This is a comment about a piece of code

//DontCallThisMethod();

// TODO - This task needs to be completed
```

All code should try to be as readable as possible, so excessive commenting is not needed. Comment code that may not be obviouse, is complicated or has multi-steps. Make comments as if you will need to read the code 10 years time.

## Method Length

Methods should be appropriate length. Larger methods are allowed, however only if they are extremely important. It is important to keep Methods organized and readable without having to excessively jump between methods to understand an operation.

## Namespaces

Use namespaces when making something that is generic enough to become it's own library. Namespaces are not necissary in core gameplay programming if there are no other conflicting type names.

`using` lines that are not used, must be removed. If a namespace is only used when a specific scripting define is present, they must be wrapped within a `#if` block (even if it doesn't prevent compilation).

## Reflection

Reflection is not be be used for auto-formatting data structures for serialization. Data must be manually formatted for clean looking data. Key naming conventions may be different depenting on the desired format (ex. "m_targetPosition" will become "target-position" in YAML and JSON style guides).