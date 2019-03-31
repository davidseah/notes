# Learn OpenGL
## Topics
* Blending

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
}

```