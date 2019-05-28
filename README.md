# 天猫精灵对接AliOS ESP32 设备

## 1. 介绍
  IoT物联网变得越来越火热, 智能家居已经走入家庭. 智能设备,如灯, 开关, 空调, 温湿度传感器, 风扇, 扫地机器人都可以通过智能音箱来控制. 
极大方便了我们日常的生活和智能家居控制管理. 针对IoT市场, Alibaba提供了完善的生态系统, 包括[AliOS Things](https://github.com/alibaba/AliOS-Things)针对终端设备, 含有Link Kit SDK
来连接阿里云IoT物联网平台-[智能生活开放平台](https://living.aliyun.com). 同时允许天猫精灵无缝连接, 通过语音来控制智能设备.
  在本文, 我们利用ESP32的平台来跑AliOS, 连接到阿里云IoT物联网平台, 通过阿里巴巴的云智能App来控制灯的开/关, 然后绑定天猫精灵,使得我
们能够通过语音来控制灯的开/关和了解灯的状态. 
     
## 2. 安装AliOS Things Studio
  阿里巴巴提供了详细的文档介绍如何安装开发环境, 下载AliOS Things的代码, 编译示例代码, 下载代码到目标板. ESP32 devkitc是AliOS Things
  支持一款开发板. 详细介绍可以参看此网址 [AliOS Things Studio](https://github.com/alibaba/AliOS-Things/wiki/AliOS-Things-Studio). 
  
## 3. Link Kit SDK介绍
  Link Kit SDK是阿里云提供给设备厂商并通过该SDK将设备安全接入到阿里云IoT物联网平台, 从而设备可以被阿里云IoT平台控制, 同样可以被阿里
云的App, 或者天猫精灵控制. 详细介绍可以参看此网址设备接入[Link Kit SDK](https://help.aliyun.com/document_detail/96596.html?spm=a2c4g.11186623.6.542.217265567uqowz)
  AliOS 已经包含Link Kit SDK, 我们可以直接生成示例代码. 
  
## 4. 编译运行Linkkit的示例代码
  打开Visual Studio Code, 点击下方工具栏红色方框中的编译选项, 输入linkkitapp选择示例代码按回车, 再输入esp32devkitc选择开发平台再回车. 
  点击编译按钮生成烧录文件, 再点击烧录. 在这里它会每次提示你选择串口, 如果觉得烦就直接修改\AliOS-Things\build\site_scons\upload\esp32.json, 
  将"@PORT@"修改成你的串口号比如"COM11". 
  点击串口工具图标观察输出, 如果有看到设备反复重启, 应该是没有烧录正确, 可以通过ESP32原厂的烧录工具来验证. 
  对此过程有疑问的可以参考网址[AliOS Things Studio](https://github.com/alibaba/AliOS-Things/wiki/AliOS-Things-Studio)

## 5. 在阿里云物联网平台建立设备
  我们需要在阿里云物联网平台建立设备, 生成product key, device key等信息, 并把信息加入到Linkkit的示例代码中, 双方才可以建立连接. 
阿里云IoT提供两个云服务平台:[生活物联网平台](https://living.aliyun.com/#/) 和 [物联网管理平台](https://www.aliyun.com/product/iot-devicemanagement?spm=ilop), 其中生活物联网平台, 提供设备端的SDK, 公版App, 开发门槛较低, 我们这里采用的就
是这个平台. 
  进入[生活物联网平台](https://living.aliyun.com/#/)
*  首先[创建产品](https://living.aliyun.com/doc?spm=a2c9o.12549863.0.0.4e9d38e48XXAqy#readygo.html), 我们选择灯的产品, 选项都默认, 这个灯只具有开关功能. 创建完后, 在右侧的基本信息中, 把Product Secret的值拷贝下来.
*  [设备调试](https://living.aliyun.com/doc?spm=a2c9o.12549863.0.0.4e9d38e48XXAqy#evgs8e.html), 选择认证模组/芯片里面没有ESP32, 我们选择最后一个品牌,型号不限. 在这里我们可以添加测试设备, 生成 Product Key, 
   Device Name, DeviceSecret等信息用于调试. 
* [人机交互](https://living.aliyun.com/doc?spm=a2c9o.12549863.0.0.4e9d38e48XXAqy#swz51p.html), 这一步可以下载公版App, 并可生成产品配网二维码, 用App扫描此二维码即可添加设备. 
* [产品发布](https://living.aliyun.com/doc?spm=a2c9o.12549863.0.0.4e9d38e48XXAqy#gs40p8.html), 在这一步可以发布产品, 购买激活码并分配给产品, 我们就可以在App里面找到我们自定义的产品.在这里点击量产管理, 选择量产记录, 
  找到你已经发布的设备,点击查看, 在弹出窗口下载激活码, 在一个excel文件里包含了Product key, Device Name 和 Device Secret. 

## 6. 定制linkkitapp的代码
### a. app\example\linkkitapp\linkkit_example_solo.c

i. 用生活物联网平台得到的产品信息更新如下宏定义. 
```c
#define PRODUCT_KEY "a1X2bEnP82z"
#define PRODUCT_SECRET "7jluWm1zql7bt8qK"
#define DEVICE_NAME "test_06"
#define DEVICE_SECRET "wQ1xOzFH3kLdjCTLfi8Xbw4otRz0lHoq"
```

ii. 添加GPIO18初始化设置
```c
gpio_dev_t led;
#define GPIO_LED_IO 18

void init_gpio(void)
{
  led.port = GPIO_LED_IO;
  led.config = OUTPUT_PUSH_PULL;
  hal_gpio_init(&led);
}

int linkkit_main(void *paras)
{
  /* Adding GPIO initialization at beginning of linkkit_main() */
  init_gpio(); 
  …
}
```

iii. 添加处理LED on/off的代码
	 在处理阿里云发送过来的JSON格式的命令处理函数添加对LightSwitch的处理
```c
static int thing_prop_changed(const void *thing_id, const char *property,void *ctx)
{
    if (strstr(property, "HSVColor") != 0) {
  …
  else if (strstr(property, "LightSwitch") != 0) {
    int sw_on = 0xFF;
    linkkit_get_value(linkkit_method_get_property_value, thing_id,
              property, &sw_on, &value_str);
    if (value_str) {
      free(value_str);
      value_str = NULL;
    }
    if (sw_on == 1) {
      hal_gpio_output_high(&led);
    }
    if (sw_on == 0) {
      hal_gpio_output_low(&led);
    }
  }
```

### b. app\example\linkkitapp\app_entry.c
  设备端和App有个配网过程, 首先App加入到一个WiFi网络内, 它会把WiFi的SSID和密码通过组播的UDP传输方式发送给设备, 设备加入WiFi后和App
建立匹配, 并注册信息到阿里云. 以后App可通过阿里云对灯设备进行控制.
  设备配网的起始函数是do_awss_active(), 原代码中要等一个按键事件才开始配网. 我们可以添加一个2秒定时器来自动运行配网. 
  
```c
static void app_delayed_action(void *arg)
{
  do_awss_active();
}

int application_start(int argc, char **argv)
{
  …
  aos_post_delayed_action(2000, app_delayed_action, NULL); 
  aos_loop_run();
  return 0;
}
```

### c. 编译linkkit app并下载到ESP板子上.

## 7. 云智能App和设备配网
* 打开云智能App, 选择扫描, 扫描生活物联网平台上人机交互阶段生成的二维码, 输入WiFi网络的SSID和密码,开始扫描设备. 
* 复位ESP32板子,2秒后开始接收App发送过来的SSID和密码. 
* 配网成功, 云智能App显现设备列表, 点击设备进入控制界面, 可以控制设备的灯的on/off

## 8. 天猫精灵语音控制灯
* 安装天猫精灵App, 并用淘宝账户登陆
* 在云智能App中选择"我的"->"第三方服务", 选择天猫精灵. 点击绑定账号, 登陆同一个淘宝账号. 灯设备出现在可控设备列表里面. 
* 在天猫精灵App里面, 找到你的灯, 分配好名字, 如书房的灯. 
* 对天猫精灵说"打开书房的灯", 灯开; "关闭书房的灯", 灯灭; "现在灯是开的么?", 回答"现在灯处于关闭状态"

