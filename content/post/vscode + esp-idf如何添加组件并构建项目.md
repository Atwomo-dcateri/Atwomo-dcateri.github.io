+++
date = '2026-04-03T15:59:59+08:00'
draft = false
title = 'Vscode + Esp-Idf如何添加组件并构建项目'
+++

注意，前提是已经部署好vscode+esp-idf的开发环境，并在vscode中安装好插件esp-idf，同时已经创建了第一个项目

在第一次使用vscode+esp-idf你会发现，如果你想在创建的组件中引入一些（头文件）组件会报错，vscode会直接出现小红波浪线，这是因为没用注册该组件导致，就像keil5中创建文件而不导入包含该文件的目录一样，编译器这个不太聪明的亚子，是找不到你创建的头文件在哪里，编译直接报错，写代码时一个提示也不会出现，真的很烦( •̀ ω •́ )y，应对之道就在下文

1. 注册组件

打开左边侧栏，点击

![alt text](/images/vscode-esp-idf/image.png)
再点击Advanced
![alt text](/images/vscode-esp-idf/image-1.png)
点击创建ESP-IDF组件
![alt text](/images/vscode-esp-idf/image-2.png)
输入你想要创建的组件名称or头文件名称（驱动的名称就是stm32里面添加头文件是一样的）
![alt text](/images/vscode-esp-idf/image-3.png)

然后点击左边侧栏的资源管理器
![alt text](/images/vscode-esp-idf/image-4.png)
你会发现这里并没有你注册的组件，这些新创建的组件必须得构建后才能引入和使用其他组件。但是项目components文件夹中已经出现创建组件（以及创建编构建后所以不是绿色的）
![alt text](/images/vscode-esp-idf/image-5.png)
![alt text](/images/vscode-esp-idf/image-6.png)

但是会发现如果现在你去引入freertos/FreeRTOS.h还是不行，你注册了该组件但是在编译器并不认识你，因为你并没被引入该项目。所以接下来的步骤也是必须的
2. fullclean清理之前的cmakeLists构建项目组件

在此之前一般要先清理之前项目组件，点击
![alt text](/images/vscode-esp-idf/image-7.png)
再点击fullclean，你发现所以文件包含头文件的地方都有报错，找不到头文件，这是因为项目组件被清空了，大家都找不到引入的头文件了。重新构建后就会消失，和代码对错没有关系，
![alt text](/images/vscode-esp-idf/image-8.png)

3. 打开创建组件的CMakeLists.txt文件
填入需要引入的组件（头文件）注意只需要填写引入组件的所在文件夹的名称（创建组件时使用的名称）就行，不用填写完整的源文件名称，也不用填写路径，cmake在构建会重新扫描文件目录，找到组件

注意，CMakeLists.txt的格式，缩进保持一致
```
idf_component_register( # 这是 ESP-IDF 专属的 CMake 宏，本质是对 CMake 的封装，专门用来注册一个组件
    SRCS bsp_button.c  # 表示包含源文件
    INCLUDE_DIRS include  # 头文件目录
    REQUIRES # 公开依赖，你能使用的其他组件中的头文件所在目录
        bsp_button.h  # 是错误的
        ./components/bsp_button # 是错误的
        bsp_button # 是正确的
           
)
```

4. build项目

点击
![alt text](/images/vscode-esp-idf/image-9.png)
再点击，构建项目
![alt text](/images/vscode-esp-idf/image-10.png)

5. 验证注册组件是否能引入CMakeList文件中的加入组件

这是没有加入前，是这样

![alt text](/images/vscode-esp-idf/image-11.png)

这是重新构建后，现在就不会报错找不到头文件了

![alt text](/images/vscode-esp-idf/image-12.png)

基本就解决引入组件这个问题了

### 总结
1. 原理和使用keil5创建组件一样，但是由于引入Make，得自己管理各个头文件之间依赖的关系，编译时间长，不如keil5快速好用

2. 这个操作顺序得记好，必须创建好组件，编辑 CMakeLists.txt,清理项目组件，然后构建项目，最后才能引入组件