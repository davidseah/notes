# Learn OpenGL
## Topics
* Shaders
* Blending

## Shaders
### What I know
* Shader is a piece of code that the GPU can run.
* Each piece of code are to be run on different pipeline
* It will take an input and generate an output
* Isolate programs that are not able to communicate with others, only communication is through the input and output 
* how does location works
    > You can set the location via shader, this will correspond the vertex information with we are create VAO
    > Alternative you can choose not to set it but use glGetAttribLocation to get the location of the variable. 
### What I don't know

## Assumption (Please verify to see if it's correct)

## Notes
## Shader structure
* Shaders always begin with a version declaration
* input variables
* output variables
* uniforms
* main function
* entry point is at main
* process any input variables and output results in its output variables 

## vertex shader
* input is known as vertex attributes
* there is an limit number of vertex attributes allowed to declare by the hardware
* OpenGL guarantees there are always at least 16 4-component vertex attributes available.
* Some hardware might allowed more, which you retrieve by querying GL_MAX_VERTEX_ATTRIBS
``` cpp
int nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes << std::endl; 
```

## Types
A vector in GLSL is a 1,2,3 or 4 component container for any of the following types
* vecn: default vector of n floats
* bvecn: bool
* ivecn: int
* uvecn: unsigned
* dvecn: double

## Ins and Outs
* GLSL defined in and out keyword for inputs and outputs
* You can pass variable from one shader file to the next by the variable names
* vertex shader recieve its input straight from the vertex data
* to define how the vertex data is organised, we specify the input variables with the location metadata so we can configure the vertex attributes on the CPU.
* i.e layout (location = 0) 
* It is possible to omit the layout (location = 0) specifier and query for the attribute locations in OpenGL code via glGetAttribLoctaion.
* Another alternative is to specify the locations yourself
``` GLSL
//vertex shader
#version 330 core
layout (location = 0) in vec3 aPos; // the position variable has attribute position 0
  
out vec4 vertexColor; // specify a color output to the fragment shader

void main()
{
    gl_Position = vec4(aPos, 1.0); // see how we directly give a vec3 to vec4's constructor
    vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // set the output variable to a dark-red color
}
```

``` GLSL
//fragment shader
out vec4 FragColor;
in vec4 vertexColor; //the input varaible from the vertex shader (same name and same type)

void main()
{
    FragColor = vertexColor;
}
```

### Uniforms
* Another way to pass data from our application in CPU to the shaders on the GPU is by uniforms
* Uniforms are global, meaning that they are unique per shader program object and can be accessed from any shader at any stage in the shader program.
* second is that uniforms will keep their values until they are either reset or updated.
* You don't need to set glUseProgram to get the uniform location
* but you need to set glUseProgram if you want to update the value.

#### Example
* We need to first find the index/location of the uniform attribute in our shader.
``` cpp
float timeValue = glfwGeTime()l
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```






## Blending
### What I know
### What I don't know
### What is blending
* mixing two colours together? 

### Notes
* Technique to implement transparency
* Transparency is all about an object not having a solid color but a mix of colour from different objects with varying intensity. 
* how much transparenyc is determine by the alpha value
* An alpha value of 50% tell us that the final color consist of 50% of it's color and 50% of what is behind 
the object.

#### How to load a transparent background
* Load an image with a transparent background
* Tell OpenGL our texture now uses an alpha channel in the texture generation procedure
glTeImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
* In your shader make sure that I retrieve all 4 values instead of just 3. 

```
void main()
{
    //FragColor = vec4(vec3(texture1, TexCoords)), 1.0);
    FragColor = texture(texture1, TexCoords);
}
```

* OpenGL by default does not know what to do with alpha values, nor when to discard them. 
* OpenGL gives us the discard command ensures the fragment will nit be further processed and this not end up in the color buffer. 

```
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;
uniform sampler2D texture1;

void main()
{
    vec4 texColor = texture(texture1, TexCoords);
    if(texColor.a < 0.1)
    {
        discard;
    }

    FragColor = texColor;
}
```

* Note that when sampling textures at their border, OpenGL interpolates the border values with the next repeated value of the texture (because we set its wrapping parameters to GL_REPEAT).  This is usually okay, but since we're using transaparent values, the top of the texture image gets its transparent value interpolated with the bottom. 

## Blending
* to render images with different levels of transparency we have to enable blending.
* We enable blending by enabling GL_BLEND, glEnable(GL_BLEND);
* Blending equation in OpenGL

$C_{result}$ = $C_{source}$ * $F_{source}$ + $C_{destination}$ * $F_{destination}$ 

$C_{source}$: source color vector. This is the color vector that originates from the texture. 

$C_{destination}$: destination color vector. This is the color vector that is currently stored in the color buffer. 

$F_{source}$: source factor value. Set the impact of the alpha value on the source color

$F_{destination}$: destination factor value. Sets the impact of the alpha value on the destination color. 

* After the fragment shader has run and all the tests have passed, this blend equation is let loose on the fragment's color output and with whatever is currently in the color buffer (previous fragment colo stored before the current fragment).

* The source and the destination colors will automatically be set by OpenGL, but the source and destination factor can be set to a value of our choosing.

