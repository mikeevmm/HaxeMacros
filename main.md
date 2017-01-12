# Haxe Macros

Recently, and only after extensive experimenting, I've found myself getting more and more comfortable with Haxe
macros. They're incredibly powerful tools, that not only can improve performance, but make your code a lot more
expressive.

Despite this, macros are still a very undocumented feature of Haxe (although initiatives such as the [Haxe Cookbook](1)
are starting to fix this), and, as such, I decided to share what I have learnt about them.

(**Note**: I will be assuming a good grasp on Haxe syntax and its workings, so I recommend only
reading this if you've had some experience programming in Haxe.)

## What Are Haxe Macros

If you're looking for a more complete or accurate definition, you can find Haxe's official overview on macros [here](2).

In layman's terms, macros are a way of unrolling logic into explicit commands at compile time, i.e., having code that
dynamically replaces itself for other code, before your program is compiled.

There are three types of macros:

- Initialization macros

- Build macros

- Expression macros

We'll be focusing on the last two, as initialization macros are only useful if you are compiling pure Haxe
(as opposed to using, for example, OpenFL), and are, as such, not ideal to get started.
However, once you grasp Build and Expression macros, it's fairly easy to understand how Initialization macros work.

## Expressions

In Haxe, at compile time, everything is an expression. Expressions, or `Expr`s, are `typedef`s that contain two elements:

```haxe
// Expr typedef:
{
    var expr : ExprDef;
    var pos  : Position;
}
```

The `ExprDef` defines what this expression is doing. For example, an array declaration expression has an `ExprDef`
of type `EArrayDecl`.

`ExprDef`s are declared as enums in Haxe source, but aren't really used like enums; we'll see this later.

The `Position` element, on the other hand, determines where this expression is being called from; if you're
using an IDE (such as [HaxeDevelop](3)) and the expression results in an error, this is where you'll be pointed to.

Although `Position` elements are `typedefs` themselves, we won't need to fiddle with them very often, as
the very handy `haxe.macro.Context` class allows us to get the position at which the macro is being executed,
through `Context.getPosition()`. On other situations, we might want to use the position of an existing expression.

(By the way, all of the mentioned classes can be found inside the `haxe.macro` package,
which I highly recommend going through, especially `Expr.hx` and `Type.hx`.)

## Creating an Expression Macro

Expression macros are special functions that, where called, replace themselves with the expression they return.
These functions are marked as `macro` functions, and must be static.

(Notice that it wouldn't really make sense them not to be static; macros logic happens before the code is even
compiled, so the notion of instantiation can never apply.)

Here's an example of a class containing a macro that declares an array:

```haxe
import haxe.macro.Context;
import haxe.macro.Expr;

class ExprMacro
{
    macro static public function makeConst():Expr {
        var currentPosition = Context.currentPos();

        return
            {
                expr : EConst(CInt('1')),
                pos  : currentPos
            };
    }
}
```

Exciting. Let's break down what's happening.

```haxe
import haxe.macro.Context;
import haxe.macro.Expr;
```

The `haxe.macro` package contains every class related to macros.
In this case we imported the arguably most used classes, `Context` and `Expr`
(the latter containing a lot of macros' elements), but there are more.

```haxe
class ExprMacro
```

All macros must be encapsulated in a class.
Mixing macros and other code is possible, using conditional compilation (`#if macro` and `#if build_macro`),
but, as a rule of thumb, avoid this, as it can cause unusual problems
(in particular imports must also be conditionally compiled, classes must be accessed through package...).

```haxe
macro static public function makeConst():Expr
```

There's a lot of things happening here: first, we signal this function as a `macro` function.
It must also be `static`, because macros can't be instantiated, as said before.
Finally, the function returns an `Expr`, which is to replace the macro function, where called.

```haxe
var currentPosition = Context.currentPos();
```

We'll be inserting our constant wherever the macro was called from, so we get that position
(which is also where are as we're running the macro).

```haxe
return {
         expr : EConst(CInt('1')),
         pos  : currentPos
       };
```

Here we define the expression to return. Notice, in particular, the "construction" of the `ExprDef`:
there's no `new` keyword, but `EConst` accepts a `Constant`, in our case `CInt`, and `CInt` accepts
a string, in our case `'1'`.

Congratulations, we've just defined `1`, macro style!

Let's test it:

```haxe
class Test {
  static function main() {
    var a:Int = ExprMacro.makeConst();
    trace(a); // 1
  }
}
```

Neat! It works as expected. Remember, at run time, this is *literally the same* as

```haxe
class Test {
  static function main() {
    var a:Int = 1;
    trace(a);
  }
}
```


[1]: http://code.haxe.org/
[2]: https://haxe.org/manual/macro.html
[3]: http://haxedevelop.org/
