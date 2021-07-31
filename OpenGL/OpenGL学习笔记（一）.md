# OpenGL学习笔记(一)

## 你好窗口

### 初始化GLFW，配置GLFW

```c++
int main()
{
    //进行初始化
    glfwInit();
    /** glfwWindowHint函数,配置GLFW
    * 第一个参数代表选项的名称，我们可以从很多以GLFW_开头的枚举值中选择；
    * 第二个参数接受一个整形，用来设置这个选项的值。 */
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    return 0;
}
```

### 创建窗口

```c++
/**glfwCreateWindow函数用于创建窗口对象
*前两个参数：需要窗口的宽和高；
*第三个参数表示这个窗口的名称（标题）*/
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
if (window == NULL)
{
    std::cout << "Failed to create GLFW window" << std::endl;
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
```

### GLAD

GLAD 管理 OpenGL 的函数指针

因此我们希望在调用任何 OpenGL 函数之前初始化 GLAD

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
    std::cout << "Failed to initialize GLAD" << std::endl;
    return -1;
}  
```

### 视口(Viewport)

```c++
/** glViewport函数
*前两个参数:控制窗口左下角的位置。
*第三个和第四个参数控制渲染窗口的宽度和高度（像素）*/
int width, height;
glfwGetFramebufferSize(window, &width, &height);
glViewport(0, 0, width, height);
```

随窗口大小变化调整视口大小

```Cpp
//告诉 GLFW 要在每次窗口调整大小时调用这个函数
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

//每次调整窗口大小时调用的窗口上注册一个回调函数
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width and 
    // height will be significantly larger than specified on retina displays.
    glViewport(0, 0, width, height);
}
```

### 渲染循环

在程序中添加一个while循环，我们可以把它称之为游戏循环(Game Loop)，它能在我们让GLFW退出前一直保持运行。

```c++
while(!glfwWindowShouldClose(window))
{
    //交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上。
    glfwSwapBuffers(window);
    //检查有没有触发什么事件（比如键盘输入、鼠标移动等），然后调用对应的回调函数
    glfwPollEvents();    
}
```

### 双缓冲(Double Buffer)

应用程序使用单缓冲绘图时可能会存在图像闪烁的问题。这是因为生成的图像不是一下子被绘制出来的，而是按照从左到右，由上而下逐像素地绘制而成的。最终图像不是在瞬间显示给用户，而是通过一步一步生成的，这会导致渲染的结果很不真实。为了规避这些问题，我们应用双缓冲渲染窗口应用程序。前缓冲保存着最终输出的图像，它会在屏幕上显示；而所有的的渲染指令都会在后缓冲上绘制。当所有的渲染指令执行完毕后，我们交换(Swap)前缓冲和后缓冲，这样图像就立即呈显出来，之前提到的不真实感就消除了。

### 释放资源

```c++
/** glfwTerminate函数,释放GLFW分配的内存 */
glfwTerminate();
return 0;
```

### 输入

GLFW中的键盘控制可以通过使用GLFW的**回调函数(Callback Function)**来完成。回调函数事实上是一个函数指针，当我们设置好后，GLWF会在合适的时候调用它。**按键回调**(KeyCallback)是众多回调函数中的一种。当我们设置了按键回调之后，GLFW会在用户有键盘交互时调用它。

```cpp
/** 按键回调函数接受一个作为它的
*第一个参数:GLFWwindow指针
*第二个参数:用来表示按下的按键
*第三个参数:表示这个按键是被按下还是释放
*第四个参数:表示是否有Ctrl、Shift、Alt、Super等按钮的操作。*/
void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
{
    // 当用户按下ESC键,我们设置window窗口的WindowShouldClose属性为true
    // 关闭应用程序
    if(key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
        glfwSetWindowShouldClose(window, GL_TRUE);
}    
```

### 渲染

如果想要在渲染循环的每次迭代中执行所有的渲染指令，则需要将所有渲染命令放在渲染循环中。

```cpp
// render loop
while(!glfwWindowShouldClose(window))
{
    // input
    processInput(window);

    // rendering commands here
    //设置用于清除屏幕的颜色颜色
	glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    //清除屏幕的颜色缓冲区
	glClear(GL_COLOR_BUFFER_BIT);

    // check and call events and swap the buffers
    glfwPollEvents();
    glfwSwapBuffers(window);
}
```

### 完整代码

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow *window);

// settings
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

int main()
{
    // glfw: initialize and configure
    // ------------------------------
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    // glfw window creation
    // --------------------
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // glad: load all OpenGL function pointers
    // ---------------------------------------
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }    

    // render loop
    // -----------
    while (!glfwWindowShouldClose(window))
    {
        // input
        // -----
        processInput(window);

        // render
        // ------
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // glfw: swap buffers and poll IO events (keys pressed/released, mouse moved etc.)
        // -------------------------------------------------------------------------------
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // glfw: terminate, clearing all previously allocated GLFW resources.
    // ------------------------------------------------------------------
    glfwTerminate();
    return 0;
}

// process all input: query GLFW whether relevant keys are pressed/released this frame and react accordingly
// ---------------------------------------------------------------------------------------------------------
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

// glfw: whenever the window size changed (by OS or user resize) this callback function executes
// ---------------------------------------------------------------------------------------------
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width and 
    // height will be significantly larger than specified on retina displays.
    glViewport(0, 0, width, height);
}
```

