# C Functions

Functions make repeating the same task repeatedly in code easier by removing the need to copy the same code multiple times and it can make the code look cleaner by making `main()` smaller and moving functions to differnt files.

This post will explain (albeit, poorly) C functions and how to make them.

## Structure of a C Function

The basic structure of a C fuction is as follows:

```c
return_type function_name(parameter list) {
    body of the function;
}
```
The `return_type` is what will be returned when the function finishes, here is a table of C return types:

<table class="boxed"><tbody><tr><th>Group</th><th>Type names*</th><th>Size / precision</th></tr><tr><td rowspan="4">Character types</td><td><code><b>char</b></code></td><td>Exactly one byte in size. At least 8 bits.</td></tr><tr><td><code><b>char16_t</b></code></td><td>Not smaller than <code>char</code>. At least 16 bits.</td></tr><tr><td><code><b>char32_t</b></code></td><td>Not smaller than <code>char16_t</code>. At least 32 bits.</td></tr><tr><td><code><b>wchar_t</b></code></td><td>Can represent the largest supported character set.</td></tr><tr><td rowspan="5">Integer types (signed)</td><td><code><b>signed char</b></code></td><td>Same size as <code>char</code>. At least 8 bits.</td></tr><tr><td><code><i>signed</i> <b>short</b> <i>int</i></code></td><td>Not smaller than <code>char</code>. At least 16 bits.</td></tr><tr><td><code><i>signed</i> <b>int</b></code></td><td>Not smaller than <code>short</code>. At least 16 bits.</td></tr><tr><td><code><i>signed</i> <b>long</b> <i>int</i></code></td><td>Not smaller than <code>int</code>. At least 32 bits.</td></tr><tr><td><code><i>signed</i> <b>long long</b> <i>int</i></code></td><td>Not smaller than <code>long</code>. At least 64 bits.</td></tr><tr><td rowspan="5">Integer types (unsigned)</td><td><code><b>unsigned char</b></code></td><td rowspan="5">(same size as their signed counterparts)</td></tr><tr><td><code><b>unsigned short</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned long</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned long long</b> <i>int</i></code></td></tr><tr><td rowspan="3">Floating-point types</td><td><code><b>float</b></code></td><td></td></tr><tr><td><code><b>double</b></code></td><td>Precision not less than <code>float</code></td></tr><tr><td><code><b>long double</b></code></td><td>Precision not less than <code>double</code></td></tr><tr><td>Boolean type</td><td><code><b>bool</b></code></td><td></td></tr><tr><td>Void type</td><td><code><b>void</b></code></td><td>no storage</td></tr><tr><td>Null pointer</td><td><code><b>decltype(nullptr)</b></code></td><td></td></tr></tbody></table>

The function name will be used for calling the function, `function_name(parameters);`.

The parameter list is the list of parameters that are required to be inputted when calling the program. For example, declaring the function: ` int help(int argc, char **argv)` and calling the function: `help(argc, argv);`.

## Let's make a C Function

For this example we'll find the larger number of two given integers. 

First, lets decide on the return type. We want to be able to print the number result of the program so the return type should be `int`. Next, the function will just be called `max`. So far we have:
```c
int max() {
    ...
}
```

Next we will need the parameters of the two numbers to compare, as well as a variable to store the result of the program. For the two numbers, let's call them num1 and num2, and for the result, we'll call it result. They will be declared as `int num1 = 2`,  `int num2 = 5`, and `int result`. Now we just need the body of the function, in order to compare the two numbers we will use the `>` operator for greater than.

```c
if (num1 > num2)   // if num1 is greater than num2
    result = num1; // num1 is greater
else               // else
    result = num2; // num2 is greater
```

And if we put it all together we get:

```c
int max(int num1, int num2) { // include parameters that will be used in the funciton
  if (num1 > num2)
    result = num1;
  else
    result = num2;

  return result;   // return result to use in other function
}
```