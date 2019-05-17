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
    * The surface is a cross-platform abstraction over windows to render to and is generally instantiated by providing a reference to the native window handle, for exmaple, HWD on wiondows. Luckily the GLFW library has a built in function to deal with the platform specific details of this. 
    *  The swap chain is a collection of render targets. Its basic purpose is to ensure that the image that we are currently rendering to is different from the on ethat is currently on the screen. This is important to make sure that only complete images are shown. 
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
Just like extension s, validation layers need to be enabled by specifying their name. Instead of having to explicitly specify all the useful layers, the SDK allows you to request the VK_LAYER_LUNAR_standard_validation layer that implicitly enables a whole range of useful diagnostic layers.

Let's first add two configuration variables to the program to specify the layers to enable and whether to enable them or not. I've chosen to base that value on whether the program is being compiled in debug mode or not. 
The NDEBUG macro is part of the C++ standard and means not debug.

```cpp
const int WIDTH = 800;
const int HEIGHT = 600;

const std::vector<const char*> validationLayers = {
    "VK_LAYER_LUNARG_standard_validation"
};

#ifdef NDEBUG
    const bool enableValidationLayers = false;
#else
    const bool enableValidationLayers = true;
#endif
```

We'll add a new function checkValidationLayerSupport that checks if all of the requested layers are available. First list all of the available layers using the vkEnumerateInstanceLayerProperties function. It's usage is identical to that of vkEnumerateinstanceExtensionProperties which was discussed in the instance creation chapter. 

```cpp
book checkValidationLayerSupport()
{
    uint32_t layerCount;
    vkEnumerateInstanceLayerProperties(&layerCount, nullptr);
    std::vector<vklayerproperties> availableLayers(layerCount);
    vkEnumerateInstanceLayerProperties(&layerCount, availableLayers.data());

    return false;
}
```
Next, check if all of the layers in validationlayers exist in the availableLayers list. You may need to include <cstring> for strcmp.
```cpp

for(const char* layername : validationLayers)
{
    bool layerFound = false;
    for(const auto& laterproperties : availableLayers)
    {
        if(strcmp(layerName, layerProperties.layername) == 0)
        {
            layerFound = true;
            break;
        }
    }

    if(!layerFound)
    {
        return false;
    }

}

return true;
```

We can now use this function in createInstance:

```cpp
void createInstance()
{
    if(enablevalidationLayers && !checkValidationLayerSupport())
    {
        throw std::runtime_error("validation layers requested, buit not available!");
    }

    ...
}
```

Now run the program in debug mode and ensure that error does not occur. If it does , then make sure you have properly installed the Vulkan SDK. if none or very few layers are being reported, then you may be dealing with this issue(requires a LunarG account to view). See that page for help fixing it.

Finally, modify the vkInstanceCreateInfo struct instantiation to include the validation layer names if they are enabled.

```cpp
if(enableValidationLayers)
{
    createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
    createInfo.ppEnabledLayerNames = validationLayers.data();
}
else
{
    createInfo.enabledLayerCount = 0;
}
```

if the check was successful then vkCreateInstance should not ever return a VK_ERROR_LAYER_NOT_PRESENT error but you hsould run the program to make sure.

## Message callback
unfortunately just enabling the layers doesn't help much, because they currently have no way to relay the debug messages back to our program. To receive those messages we have to set up a debug messenger with a callback, which requires the VK_EXT_debug_utils extension.

We'll first create a getReuiredExtensions function that will return the required list of extensions based on whether validation layers are enabled or not: 

```cpp
std::vector<const char*> getRequiredExtensions()
{
    uint32_t glfwExtensionCount = 0;
    const char** glfwExtensions;
    glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);

    if(enableValidationLayers)
    {
        extensions,push_back(VK_EXT_DEBUG_UTILS_EXTENSION_NAME);
    }

    return extensions;
}
```
The extensions specified by GLFW are always required but the debug messenger extension is conditionally added. Note that I've used the VK_EXT_DEBUG_UTILS_EXTENSION_NAME macro here which is equal to the literal string "VK_EXT_debug_utils". Using this macro lets you avoid typos.

We can now use this function in createInstance:
```cpp
auto extensions = getRequiredExtensions();
createInfo.enabledExtensionCount = static_cast<uint32_t>(extensions.size());
createInfo.ppEnabledExtensionsNames = extensions.data();
```
Run the program to make sure you don't receive a VK_ERROR_EXTENSION_NOT_PRESENT error. We don't really need to check for the existence of this extension, because it should be implied by the availability of the validation layers.


The VKAPI+ATTR and VKAPI_CALL ensure that the function has the right signature for Vulkan to call it.

```cpp
static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback(
)
{
    std::cerr << "validation layer: " << pCallbackData->pMessage << std::endl;

    return VK_FALSE;
}
```
The first parameter specifies the severity of the message, which is one of the following flags:
* VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT: Diagnostic message
* VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT: Informational message like the creation of a resource
* VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT: Message about behavior that is not necessarily an error, but very likely a bug in your application
* VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT: Message about behavior that is invalid and may cause crashes


```cpp
{
    //message is important enough to show
}
```
The messageType parameter can have the following values:
* VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT: Some event has happened that is unrelated to the specification or performance


* pMessage: The debug message as a null-terminated string
* pObjects: Array of vulkan object handles related to the message



All that remains now is telling Vulkan about the callback function. Perhaps somewhat surprisingly, even the debug callback in Vulkan is managed with a handle that needs to be explicitly created and destroyed. Such a callback is part of a debug messenger and you can have as many of them as you want. Add a class member for this handle right under instance:

```cpp
VKDebugUtilsMessengerEXT debugMessenger;
```

Now add a function setupDebugMessenger to be called from init Vulkan right after createInstance:
```cpp
void initVulkan()
{
    createInstance();
    setupDebugMessenger();
}

void setupDebugMessenger()
{
    if(!enabledValidationLayers)
    {
        return;
    }
}
```

We'll need to fill in a structure with details about the messenger and its callback:

```cpp
VkDebugUtilsMessengerCreateInfoEXT createInfo = {};
createInfo.sType = VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;
createInfo.messageSeverity = VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT;
createInfo.messageType = VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT;
createInfo.pfnUserCallback = debugCallback;
createInfo.pUserData = nullptr; // Optional
```

The messageSeverity field allows you to specify all the types of severities you would like your callback to be called for. I've specified all types except for VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT here to receive notification about possible problems while leaving out verbose general debug info. 

Similarly the messageType field lets you filter which types of messages your callback is notified about. I've simply enabled all types here. You can always disable some if they're not useful to you.

Finally, the pfnUserCallback field specifies the pointer to the callback function. You can optionally pass a pointer to the pUseData field which will be passed along to the callback function via the pUserData parameter. You could use this to pass a pointer to the HelloTriangleApplication class, for example. 

Note that they are many more ways to configure validation layer messahes and debug callbacks but this is a good setup to get started with for this tutorial. 

This struct should be passed to the vkCreateDebugUtilsMessengerEXT function to create the vkDebugUtilsMessengerEXT object. Unfortunately because this function is an extension function, it is not automatically loaded. We have to look up its address ourselves using vkGetInstanceProcAdd. We're going to create out own proxy function that handles this in the background. I've added it right above the HelloTriangleApplication class definition.

```cpp

VkResult CreateDebugUtilsMessengerEXT9VkInstance instancem const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo, const VKAllocationCallbacks* pAllocator, VKDebugUtilsMessengerEXT* pDebugMessenger)
{

    auto func = (PFN_vkCreateDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkCreateDebugUtilsMessengerEXT");
    if (func != nullptr) {
        return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
    } else {
        return VK_ERROR_EXTENSION_NOT_PRESENT;
    }
}
```
The vkGetInstanceProcAddr function will return nullptr if the function couldn't be loaded. We can now call this function to create the extension object if it's available:
```cpp
if(CreateDebugUtilsMessengerEXT(instance, &createInfo, nullptr, &debugMessenger) != VK_SUCESS)
{
    throw std::runtime_error("failed to set up debug messenger!");
}
```

Create another proxy function right below createDebugUtilsMessengerEXT:
```cpp
void DestroyDebugUtilsMessengerEXT(VKInstance instance, VKDebugUtilsMessengerEXT debugMessenger, const VKAllocationCallbacl* pAllocator)
{
     auto func = (PFN_vkDestroyDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkDestroyDebugUtilsMessengerEXT");
    if (func != nullptr) {
        func(instance, debugMessenger, pAllocator);
    }
}
```

Make sure that this function is either a static class function or a function outside the class. We can then call it in the cleanup function.

# Physcial Devices and queue Families
## Selecting a physical device
After initializing the Vulkan library through a VkInstance we need to look for and select a graphocs card in the system that supports the features we need. In fact we can select any number of graphics cards and use them simultanesously,  but in this tutorial we'll stick to the first graphics card that suits our needs. 
we'll add a function pickPhysicalDevice and add a call to it in the initvulkan function.
```cpp
void initVulkan()
{
    createInstance();
    setupDebugCallback();
    pickPhysicalDevice();
}

void pickPhysicalDevice()
{

}
```

The grpahics card that we'll end up selecting will be stored in a VkPhysicalDevice handle that is added as a new class member. This object will be implicity destroyed when the VkInstance is destroyed, so we won't need to do anything new in the cleanup function.

```cpp
VkPhysicalDevice phsyicalDevice = VK_NULL_HANDLE;
```

Listing the graphics cards is very similar to listing extensions and starts with querying just the number
```cpp
uint32_t deviceCount = 0;
vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);

```

if there are 0 devices with Vulkan support then there is no point going further
```cpp

if(deviceCount == 0)
{
    throw std::runtime_error("failed to find GPUs with Vulkan suppor!);
}
```

Otherwise we can now allocate an arry to hold all of the vkPhyscialDevices handles.
```cpp
std::vector<vkPhysicalDevice> devices(deviceCount);
vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());
```

Now we need to evulate each of them and check if they are suitable for the operations we want to perform because not all graphics cards are created equal. For that we'll introduce a new function.

```cpp
bool isDeviceSuitable(vkPhysicalDevice device)
{
    return true
}
```

And we'll check if any of the physcial devices meet the requirements that we'll add to that functions.
```cpp
for(const auto& device: devices)
{
    if(isDeviceSuitable(device))
    {
        physicalDevice = device;
        break;
    }
}

if(physicalDevice == VK_NULL_HANDLE)
{
    throw std::runtime_error("failed to find a suitable GPU!");
}

```
Th next section will intriduce the first rquirements that we'll check for in the isDeviceSuitable funcyion. As we'll start using more Vulkan features in the later chapters we will also extend thsi function to include more checks.

## Base device suitability checks
To evulate the suitability of a device we can start by querying for some details. Basic device properties like the name, type and supported Vulkan version can be queried using vkGetPhyscialDeviceProperties.
```cpp
vkPhysicalDeviceProperties deviceProperties;
vkGetPhysicalDeviceProperties(device, &deviceProperties);
```

The support for optional features like textyre compression, 64 bit floats and multi viewport rendering (Useful for VR) can be queried using vkGetPhysicalDeviceFeatures:
```cpp
vkPhysicalDeviceFeatures deviceFeatures;
vkGetPhysicalDeviceFeatures(device, &deviceFeatures);
```

There are more details that can be queried from devices that we'll discuss later concerning device memory and queue families. 

As an example, let;s say we consider our application only usable for dedicated graphics cards that support geometry shaders. Then the isDeviceSuitable function would look like this:
```cpp
bool isDeviceSuitable(vkPhysicalDevice device)
{
    vkPhysicalDeviceProperties deviceProperties;
    vkPhysicalDeviceFeatures deviceFeatures;
    vkGetPhysicalDeviceProperties(device, &deviceProperties);
    vkGetPhysicalDeviceFeatures(device, &deviceFeatures);

    return fevcieProperties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU && deviceFeatures.geometryShader;
}
```
Instead of just checking of a device is suitable or not and going with the first one, you could also give each device a score and pick the highest one. That way you could favor a dedicated graphics card by giving it a higher score but fall back to an integrated GPU of that

```cpp
#include <map>
...
void pickPhysicaldevice()
{
    //Use an ordered map to automatically sort candidates by increasing score
    std::multimap<int, VKPhysicalDevice> candidates;


    for(const auto& device : devices)
    {
        int score = rateDeviceSuitability(device);
        candidates.insert(std::make_pair(scorem device));   
    }

    //Check if the best candidate is suitable at all
    if(condidates.rbegin()->first > 0)
    {
        physicalDevice = candidates.rbegin()->second;
    }
    else
    {
        throw std::runtime_error("failed to find a suitable GPU!);
    }
}

int rateDeviceSuitability(vkPhysicalDevice device)
{
    ...

    int score = 0;

    //Discrete GPUs have a significant performance advantage
    if(deviceProperties.deviceType == VK+PHYICAL_DEVICE_TYPE_DISCRETE_GPU)
    {
        scpre += 1000;
    }

    //Maximum possible size of textures affects graphics quality
    score += deviceProperties.limits.maxImageDimension2D;

    //Application can't function without geomtry shaders
    if(!deviceFeatures.geometryShader)
    {
        return 0;
    }

    return score;
}


```

You don't need to implement all that for this tutorial, but it's to give you an idea of how you could design your device selection process. Of course you can also display the names of the choices and allow the user to select.

Because we're just starting out, Vulkan support is the only thing we need and therefore we'll settle 

## Queue Families
It has been briefly touched upon before that almost every operation in Vulkan, anything from drawing to uploading textures, requires commands to be submitted to a queue. There are different types of queue that priginate from different queue families and each family of queues allows on;y a subset of commands. For example, there could be a queue family that only allows processing of cimpute commands or one that only allows memory transfer related commands. 

We need to check which queue families are supported by the device and which one of these supports the commands that we want to use. For that purpose we add a new function findQueueFamilies that looks for all the queue families we need. Right now we'll only look for a queue that supports graphics commands, but we may extend this function to look for more at a later point in time. 

This function will return the indicies of the queue families that satisfy cetrain desired properties. The best way to do that is using a structure where we use std::optional to track if an index was found:
```cpp
struct QueueFamilyIndices
{
    std::optional<uint32_t> graphicslFamily;
    bool isComplete()
    {
        return graphicsFamily.has_value();
    }
}
```

Note that this also requires including <optional>. We can now implementing findQueueFamilies:
```cpp
QueueFamilyIndices findQueueFamilies(vkPhysicalDevicedevice)
{
    QueueFamilyIndices indices;
    return indices;
}
```

The process of retrieving the list of queue families is exactly what you expect and uses vkGetPhysicalDeviceQueueFamilyProperties:

```cpp
uint32_t queueFamilyCount = 0;
vkGetPhysicalDeviceQueueFamilyProperties(devicem &deviceFamilyCount, nullptr);

std::vector<vkQueueFamilyProperties> queueFamilies(queueFamilycount);
vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());
```
The vkQueueFamilyProperties struct contains some details about the queue family, including the type of operations that are supported and the number queues that can be created based on that family. We need to find at least one quque family that supports VL_QUEUE_GRAPHICS_BIT. 

```cpp
int i = 0;

for(const auto& queueFamily :queueFamilies)
{
    if(queueFamily.queueCount > 0 && queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT)
    {
        indices.graphicsFamily = i;
    }

    if(indices.isComplete())
    {
        break;
    }
    i++;
}
```

Now that we have thsi fancy queue family lookup functionm we can use it as a check in the isDeviceSuitable function to ensure taht the device can process the commands we wnat to use.
```cpp
bool isDeviceSuitable(vkPhysicalDevice device)
{
    QueueFamilyIndices indices = findQueueFamilies(device);
    return indices.isComplete();
}
```

# Logical device and queues
## Intoduction
After selecting a physical device to use we need to set up a logical device to interface with it. The logical device creation process is similar to the instance creation process and describes the features we want to use. 

We also need to specify which queues to create now that we've queried which queue families are available. You can even create multiple logical devices from the same physical device if you have varying requirements.

Start by adding a new class member to store the logical device handle in.
```cpp
VkDevice device;
```

Next add a createlogicalDevice function that is called from initVulkan.

```cpp
void initVulkan()
{
    createinstance();
    setupDebugCallback();
    pickPhysicalDevice();
    createLogicalDevice();
}

void createLogicalDevice()
{


}
```
## specify the queues to be created
The creation iof a logical device involves specifying a bunch of details ins struct again, of which the forst one will be VkDeviceQueueCreateInfo. This structure describes the number of queues we want for a single queue family. Right now we're only interested in a queue with graphics capabilities. 

```cpp
QueueFamilyIndices indices = findQueueFamilies(physicalDevice);
VkDeviceQueueCreateInfo queueCreateInfo = {};
queueCreateInfo,sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
queueCreateInfo.queueCount = 1;
```


Vulkan lets you assign priorities to queues to influence the schdeduling of command buffer execution using floating point numbers between 0.0 and 1.0. This is required even if there is only a single queue. 
```cpp
float queuePriority = 1.0f;
queueCreateInfo.pQueuePriorities = &queuePriority;
```
## Specifying used device features
The next information to specify is the set of device features that we'll be using. These are the features that we queried support for with vkGetPhysicalDeviceFeatures in the previous chapter, like geometry shaders. Right now we don't need anything special, so we can simply define it and leave everything to VK_FALSE. we'll cine vack to this structure once we're about to start doing more interesting things with Vulkan.

```cpp
VkPhysicalDeviceFeatures deviceFeatures = {};
```
## Creating the logical device
With the previous two structure in place, we can start filing in the main VkDeviceCreateInfo structure.

```cpp
VkDeviceCreateInfo createInfo = {};
createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
```

First add pointers to the queue creation info and device features structs:
```cpp
createInfo.pQueueCreateInfos = &queueCreateInfo;
createInfo.queueCreateInfoCount = 1;
createInfo.pEnabledFeatures = &deviceFeatures;
```

The remainder of the information bears a resemblance to the VkInstanceCreateInfo struct and requires you to specify extensions and validation layers. The difference is that these are device specific this time. 

An example of a device specific extension is VK_KHR_swapchain, which allos you to present rendered images from that device to windows. It is possible that there are Vulkan devices in the system that lack thsi ability, for example because they only support compute operations. We will come back to thsi extension in the swap chain chapeter. 

Previous implementation of Vulkan made a distinction between instance and device specific validation layers, but this is no longer the case. That means that the enabledLayerCount and ppEnabledLayerNames fields of VkDeviceCreateInfo are ignored by up to date implementations. However it is still a good idea to set them anyway to be compatible with older implementations. 

```cpp
createInfo.enabledExtensionCount = 0;

if(enabledValidationLayers)
{
    createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
    createInfo.ppEnabledLayerNames = validationLayers.data();
}
else
{
    createInfo.enabledLayerCount = 0;
}
```

We won;t need any device specific extensions for now.

That's it, we're now ready to instantiate the logical device with a call to the appropriately named vkCreateDevice function.

```cpp
if(vkCreateDevice(physicalDevice, &createInfo, nullptr, &device)!= VK_SUCCESS)
{
    throw std::runtime_error("failed to create logical device!");
}
```

The parameters are the physical device to interface with, the queue and usage info we just specified, the optional allocation callbacks pointer and a pointer to a variable to store the logical device handle in. Similarly to the instance creation function, this call can return erros based on enabling non existent extensions or specifying the desired usage of unsupported features.

The device should be destroyed in cleanup with vkDestoryedDevice function.
```cpp
void cleanup()
{
    vkDestroyDevice(device, nullptr);
}
```
Logical devices don't interact directly with instances, which is why it's not included as a parameter. 




## Retrieving queue handles
The queues are automatically created along with the logical device, but we don't have a hanlde to interface with them yet. First add a class member to store a handle to the graphics queue:
```cpp
VkQueue graphicsQueue;
```

Device queue are implicitly cleaned up when the device is destoryed, so we don't need to do anything in cleanup. 

We can use the vkGetDeviceQueue function to retrieve queue handles for each queue family. The parameters are the logical device, queue family, queue index and a pointer to the variable to store the queue handle in. Because we're only creating a single queue from this family, we'll simpky use index 0.
```cpp
vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);
```
With the logical device and queue handles we can now actually start using the graphics card to do things!In the next few chapters we'll set up the resources to present results top the window system. 





 

