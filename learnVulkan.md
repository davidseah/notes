# Learn Vulkan
## Topics

### What I know
### What I don't know
## Assumption (Please verify to see if it's correct)
## Notes

## Basic
### What I don't know
* vkInstance

### What is takes to draw a triangle
1. Instance and physical device selection
    * start by setting up Vulkan API througha vkInstance. 
    * After creating the instance, you can guery for vulkan supported hardware and select one or more vkPhysicalDevices to use for operations. 
    * You can query for properties like VRAM size and device capabities to selct desired devices, for example to prefer using dedicated graphics card


2. Logical device and queue families
    * After slecting the right hardware device to use, you need to create a vkDevice(logical device), where you describe more specifically which vkPhyscialDeviceFeatures you will be using, like multi viewport rendering and 64bits floats. 
    * You also need to specify which queue families
    you would like to use. most operations performed with Vulkan,like draw commands and memory operations are asynchronously executed vy submitting them to a vkQueue. Queues are allocated from queue families, where each quee family supports a specific set of operations in its queues. For example there could be seperate queue families for graphics, compute and memory transfer operations. 
    * The availability of queue families could also be used as a distingishing factor in physical device selection. 
    * It is possible for a device with Vulkan support to not offer any graphical functionaility, however all graphics cards with Vulkan support today will generally support all queue operations that we are interested in.

3. window surface and swap chain
    *  We will be using glfw to create a window to present rendered images to. 
    * We need two more components to actually render to a window: a window surface(vkSurfaceKHR) and a swap chain(vkSwapchainKHR). 
    * Note the KHR postfix, which means that these objects are part of a Vulkan extension. 
    The vulkan API itself is completely platform agnostic, which is why we need to use the standarddized WSI (window system interface) extension to interact with the window manager. 
    * The surface is a cross-platform abstraction over windows to render to and is generally instantiated by providing a reference to the native window handle, for exmaple, HWD on wiondows. Luckilu the GLFW library has a built in function to deal with the platform specific details of this. 
    *  The swap chain is a collection of render targets. Its basic purpose is to ensire that the image that we are currently rendering to is different from the on ethat is currently on the screen. This is important to make sure that only complete images are shown. 
    * Every time we want to draw a frame we have to ask the swap chain to provide us with an image to render to. When we've finished drawing a frame, the imgae is returned to the swap chain for it to be presented to the screen at some point. 
    * The number of render targets and conditions for presenting finished images to the screen depends on the present mode. 
    * Common present modes are double buffering 9vsync) and triple buffer.
    * Some platforms allow you to render directly to a display without interacting with any window manger throught the VK_KHR_display and VK_KHR_display_swapchain extensions. These allow you to create a surface that represents the entire screen and could be used to implement your own window manager
    
4. Image views and frambuffers
    * To draw to an image acquired from the swap chain, we have to wrap it into  vkmageView and VKFrameBuffer. 
    * An image view references a specific part of an image to be used and a framebuffer references image views that are to be sued for color, depth and stencil targets. becasue there could be many different images in the swap chain, we preemptivly create an image view and framebuffer for each of them and selct the right one at draw time

5. render passes

6. graphics pipeline
7. command pools and command buffers
8. main loop


## Practical 
### Resource Management
* Vulkan objects are either created directly with functions like vkCreateXXX, or allocated through another object with functions like vkAllocateXXX. 

* after making sure that an object is no longer used anywhere you need to destory it with the counterparts vkDestoryXXX and vkFreeXXX. 
* The parameters for these functions generally vary for different types of objects but there is one parameter that tehy all share: pAllocator. This is an optional paramter that allows you to specifiy callbacks for a custom memory allocator.

## vulkan- tutorial
* stdexcept and iostream headers are included for reporting and propagating errors

* functional headers will be used for a lambda functions in the resource management section.

* cstdlib header provides the EXIT_SUCCESS and EXIT_FILURE macros

* Program itself is wrapped into a class where we'll store the Vulkan objects as private members and add functions to initiate each of them, which we will be called from initVulkan function. 

* once everything has been prepared, we enter the main loop to start rendering frames. 

* We will fill in the mainLoop function to include a loop that iterates until the window is closed

* Once the window is closed and mainLoop returns, we'll make sure to deallocate the resources we've used in the cleanup function.

### Resouce Management
* Vulkan objects are either created directly with functions like vkCreateXXX, or allocated through another object with functions like vkAllocateXXX. 
* After making sure that an object is no longer used anywhere, you need to destroy it with the counterparts vkDestoryXXX and vkFreeXXX. 

* The parameters for these functions generally vary for different tyoes of objects, but there is one parameter that they all share: pAllocator. This is an optional paramter that allows you to specify callbacks for a custom memory allocator. We will ignore this parameter in the tutorial and always pass nullptr as argument. 

### Integrating GLFW
* Vulkan works perfectly fine without creatig a window if you want to use it off-screen rendering but it's a lot more exciting to actually show something. 
* replace #include <vulkan/vulkan.h> with #define GLFW_INCLUDE_VULKAN and #include <GLFW/glfw3.h>

* this way GLFW will include its own definitions and automatically load the vulkan header with it. 

* because GLFW library was originally designed to create an OpenGL context, we need to tell it to not create an OpenGL context with  
```cpp
glfwWindowHint (GLFW_CLIENT_API, GLFW_NO_API);
```
* Handling resized windows takes special care that we'll look into later, disable it for now with another wondow hint call.
```cpp
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```

## Instance
### Creating an instance
* The very first thing you need to do is initialize the Vulkan library by creating an instance. 

* The instance is the connection between your application and the Vulkan library and creating it involves specifying some details about your application to the driver

* Now to create an instance we'll first have to fill in a struct with some information about our application. 

* this data is technically optional but it may provide some useful information to the driver to optimize for our specific applicable for example because it uses a well know graphics engine with certain behavior. 

* This struct is called vkApplicationInfo

```cpp
VkApplicationInfo appInfo = {};
appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
appInfo.pApplicationName = "Hello Triangle";
appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.pEngineName = "No Engine";
appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.apiVersion = VK_API_VERSION_1_0;
```

* many structs in Vulkan require you to explicitly specify the type in the sType member. This is also one of the many structs with a pNext member that can point to extension information in the future. We are using default initialization here to leave it as nullptr.

* A lot of information in Vulkan is passed through structs instead of function paramters and we'll have to fill in one more struct to provide sufficient information for creating an instance. 

* The next struct is not optional and tells the Vulkan driver which global extensions and validation layers we want to use. 

* Gloabl here means that they apply to the entore program and not a specific device.


``` cpp
vkInstanceCreateInfo createInfo = {};
createinfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
createInfor.pApplicationInfo = &appInfo;
```

* The first two parameters are straightforward.
* The next two layers specify the desired global extensions. 
* As mentioned in the overview chapter, Vulkan is a platform agnostic API, which means that you need an extension to interface with the window system. GLFW has a handy built in function that returns the extension it needs to do which we can pass to thh struct
```cpp
uint32_t glfwExtensionCount = 0;
const char** glfwExtensions;
glfwExtension = glfwGetRequiredInstanceExtnsions(&glfwExtensionCount);

createInfo.enableExtensionCount = glfwExtensionCount;
createInfo.ppEnabledExtensionNames = glfwExtensions;
```


* The last two members of the struct determine the global validation layers to enable. 
```cpp
createInfo.enabledLayerCount = 0;
```

* We have now specified eferything Vulkans needs to create an instance and we can finally issue the vkCreateInstance
```cpp
vkResult result = vkCreateinstance(&createInfo, nullptr, &instance);
```

### Checking for extension support

### Cleaning up
* the vkInstance should be only destroyed right before the program exits. It can be destroyed in cleanup with the vkDestroyInstance function:

``` cpp
void cleanup()
{
    vkDestoryInstance(instance, nullptr);
    glfwDestoryWindow(window);
    glfwTerminate();
}
```
## Validation Layers
* Vulkan API is designed around the idea of minimal driver overhead and one of the manifestations of that goal is that there is very limited error checking in the API by default.

* Even mistakes as simple as setting enumerations to incorrect values or passing null pointers to required paramters are generally not explicitly and will simply result in crashes or undefined behavior. 

* Because Vulkan requires you to ver very explicit about everything you're doing it's easy to make many small mistakes like using a new GPU feature and forgetting to request it at logical device creation time. 

* However that doesn't mean that these checks can't be added to the API. VUlkan introduces an elegant system for this known as validation layers. 

* Validation layers are optional compinents that hook into Vulkan function calls to apply additional operations. 

* Common operations in validation layers are
    * Checking the values of paramters against the specification to detect misues
    * Tracking creation and destruction of objects to find resource leaks
    * Checking thread safety by tracking the threads that calls originate from
    * Logging every call and its parameters to the standard output
    * Tracing Vulkan calls for profiling and replaying

* Here's an example fo what the implementation of a function in a diagnostics validation layer could look like
```cpp
vkResult vkCreateInstance ()
{
    const vkInstanceCreateInfo* pCreateInfo, 
    const vkAllocationCallbacks* pAllocator,
    vkInstance* instance
}
{
    if(pCreateInfo == nullptr || instance == nullptr)
    {
        log("Null pointer passed to required paramter!);
        return VK_ERROR_INITIALIZATION_FAILED;
    }

    return real_vkCreateInstance(pCreateInfo, pAllocator, instance);
}
```

These validation layers can be freely stacked to included all the debugging functionality that you're interested in. You can simply enable validation layers for debug builds and completely disable them for release builds, which gives you the best of both worlds. 

Vulkan does not come with any validation layers built in, but the LunarG vulkan SDK provides a nice set of layers that check for common errors. 

Validation layers can only be used if they have been installed onto the system. For example, the LunarG validation layers are only available on PCs with vulkan SKD installed. 

There were formerly two different types of validation layers in Vulkan.
1. Instance
2. Device specific

The idea was that instance layers would only check calls related to global Vulkan object like instance.

And device specific layers would only check calls related to a specific GPU.

Device specific layers have now been deprecated, which means that instance validaition layers apply to all Vulkan calls. 

The specification document still recommends that you enable validation layers at device level as well for compatibility, which is required by some implementations. 

We simply specify the same layers as the instance at logical device level.

### Using validation layers
Just like extension s, validation layers need to be enabled by specifying their name. Instead of having to explicitly specify all the useful layers, the SDK allows you to request the VK_LAYER_LUNAR_standard_validation layer that implicity enables a whole range of useful diagnostic layers. 



