# C static extern cheat sheet

## Basic concepts
* 1 source file = 1 compilation unit.
* To produce executable program (such as .exe), all compilation units will be compiled as object files. The object files will then be linked by linker to produce the final executable file.
* By default, symbols (variable/function name) are **shared** among compilation units. When a symbol appears more than once, the compilation fails. In other words, global variables and functions use **external-linkage** by default.

```c
/* foo.c */
int foo_num = 123;
void fooSomething(void) {
}
```
```c
/* bar.c */
int foo_num = 456; // ERROR
void fooSomething(void) { // ERROR
}
```

## I don't want to share, how?

Short answer, use **static**. It makes the **global variables and functions** to use **internal-linkage**. In the example below, ``foo_num`` and ``fooSomething()`` in _foo.c_ are **private** to the _foo.c_ itself only, so _bar.c_ is allowed to use the same name for something totally different.

```c
/* foo.c */
static int foo_num = 123;
static void fooSomething(void) {
    assert(foo_num == 123);
}
```
```c
/* bar.c */
static int foo_num = 456; // OK
static void fooSomething(void) { // OK
    assert(foo_num == 456);
}
```

## To share functions

Just declare the function prototype in source file where you want to use it. ``extern`` keyword is optional.

```c
/* foo.c */
void fooSomething(void) { // to allow other .c use this
}
```
```c
/* bar.c */
void fooSomething(void); // put function prototype
void barApple(void) {
    fooSomething(); // OK
}
```

_In ANSI C (C89/C90), if there is no function declaration (non-prototyped function), it will assume extern int function. However, this is not a good practice, please never do this in new code._

## To share variables

Declare the ``extern`` variable in source file where you want to use it. ``extern`` keyword is mandatory.

```c
/* foo.c */
int foo_num = 123;
```
```c
/* bar.c */
extern int foo_num;
void barApple(void) {
    assert(foo_num == 123); // OK
}
```

## ``static`` inside function

A **local variable** is a variable declared inside a function. It has a short lifetime; its lifetime ends when it is out-of-scope.

```c
/* foo.c */
void fooTest(void) {
    int foo = 1024;
    {
        int bar = 2048;
        if (foo != bar) {
            int foobar = 1;
        }
        /* `foobar` destroyed */
    }
    /* `bar` destroyed */
}
/* `foo` destroyed */
```

A **static local variable** is like local variable but with a long lifetime. It is initialized once when the program starts, and its lifetime ends only when the program ends.

```c
/* foo.c */
int fooAccumulate(void) {
    static int count = 0;
    return ++count;
}
````
```c
/* bar.c */
int fooAccumulate(void);
void barTest(void) {
    assert( fooAccumulate() == 1 );
    assert( fooAccumulate() == 2 );
    assert( fooAccumulate() == 3 );
}
```

And yes, there is no ``extern`` local variable.

## Bouns: in C++...

The concepts above applied in C++ as well, with some exceptions.

First, in C++ `const` global variable implies internal-linkage. This means ``int const MY_MAGIC = 123;`` in C++ is equal to ``static int const MY_MAGIC = 123;`` in C. [Read this Stack Overflow post](http://stackoverflow.com/questions/998425/why-does-const-imply-internal-linkage-in-c-when-it-doesnt-in-c).

Next, in C++, `static` has a totally different meaning in `class`. I think an example would be better than explaination here:
```c++
class Foo {
public:
    static int s_apple;
    int m_axon;
    Foo(int axon) : m_axon(axon) {}
};
int Foo::s_apple = 555;

int main() {
    Foo foo1(23), foo2(10);
    assert( foo1.m_axon == 23 );
    assert( foo2.m_axon == 10 );
    assert( Foo::s_apple == 555 );
    foo1.s_apple = 
    return 0;
}
```
