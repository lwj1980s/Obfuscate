:heavy_check_mark: [Source available on GitHub](https://github.com/adamyaxley/Obfuscate)

# Obfuscate
Guaranteed compile-time string literal obfuscation header-only library for C++14.

## Quick start guide
1. Copy `obfuscate.h` into your project
2. Wrap security sensitive strings with `AY_OBFUSCATE("My String")`

Now your project will not expose those strings in plain text in the binary image.

## Whats the problem?
When plain text string literals are used in C++ programs, they will be compiled as-is into the resultant binary. This causes them to be incredibly easy to find. One can simply open up the binary file in a text editor to see all of the embedded string literals in plain view. A special utility called [strings](https://en.wikipedia.org/wiki/Strings_(Unix)) actually exists which can be used to search binary files for plain text strings.

## What does this library do?
This header-only library seeks to make it much much more difficult for embedded string literals in binary files to be found by encrypting them at compile-time, forcing the compiler to store the encrypted string literal instead of the plain text version. This will then be decrypted at runtime to be utilised within the program.

### Technical features
* _Guaranteed compile-time obfuscation_ - the string is compiled via a constexpr expression.
* _Global lifetime_ - the obfuscated string is stored in a function level static variable in a unique lambda.
* _Implicitly convertible to a char*_ - easy to integrate into existing codebases.

By simply wrapping your string literal `"My String"` with `AY_OBFUSCATE("My String")` it will be encrypted at compile time and stored in an `ay::obfuscated_data` object which you can manipulate at runtime. For convenience it is also implicitly convertable to a `char*`.

For example, the following program will not store the string "Hello World" in plain text anywhere in the compiled executable.
```c++
#include "obfuscate.h"

int main()
{
  std::cout << AY_OBFUSCATE("Hello World") << std::endl;
  return 0;
}
```

### Examples of usage
Because the obfuscated data that is generated by `AY_OBFUSCATE` has global lifetime it is completely fine to also use it in both a local and a temporary context.
```c++
char* var = AY_OBFUSCATE("string");
const char* var = AY_OBFUSCATE("string");
static const char* var = AY_OBFUSCATE("string");
std::string var = AY_OBFUSCATE("string");
function_that_takes_char_pointer(AY_OBFUSCATE("string"));
```

## Binary file size overhead
This does come at a small cost. In a very naive login program, which obfuscates two strings (username and password) the following binary file bloat exists.

| Config | Plain string literals | Obfuscated strings | Bloat |
|:------:|:---------------------:|:------------------:|:-----:|
| Release | 16384 | 18944 | 2560 (15.6%) |
| Debug | 77312 | 83456 | 6144 (7.9%) |

This output is generated by running calculate_bloat.py
