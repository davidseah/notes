# Lambda
```cpp
#include <algorithm>
#include <math>

void abssort(float* x, unsinged n)
{
    std::sort (x, x + n,
        //Lamda expression begins
        [] (flaot a, float b)
        {
            return (std::abs(a) < std::abs(b));
        }
    )
}


```


![](https://docs.microsoft.com/en-us/cpp/cpp/media/lambdaexpsyntax.png?view=vs-2019)

1. capture clause(Also known as the lambda-introducer in the C++ specification)
2. paramter list optional (also known as the lambda declarator)
3. mutable specification optional
4. exception-specification optional
5. trailing return type optional
6. lambda body

## Capture Clause
---
A lambda can intriduce new variables in its body (in C++14), and it can also acess or capture variables from the surrounding scope. 
