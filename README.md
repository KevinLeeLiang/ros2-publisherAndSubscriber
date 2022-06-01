# ros2-publisherAndSubscrible
## 编写一个简单的发布者和订阅者(C++)

## ros2 安装
[官网安装](https://docs.ros.org/en/foxy/Installation/Alternatives/Ubuntu-Development-Setup.html)

## 设置环境变量

```bash
source /opt/ros/foxy/setup.bash
```

## 创建功能包

[ROS2—创建工作区间和功能包](https://www.guyuehome.com/35371 "ROS2--创建工作区间和功能包")

```bash
ros2 pkg create --build-type ament_cmake cpp_package --dependencies rclcpp std_msgs
```

## 创建发布者

```bash
cd ~/ros2_ws/src/cpp_package/src
touch publisher_member_function.cpp
```

将下面c++代码复制publisher\_member\_function.cpp文件中

```cpp
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

/* This example creates a subclass of Node and uses std::bind() to register a
* member function as a callback from the timer. */

class MinimalPublisher : public rclcpp::Node
{
  public:
    MinimalPublisher()
    : Node("minimal_publisher"), count_(0)
    {
      publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
      timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
    }

  private:
    void timer_callback()
    {
      auto message = std_msgs::msg::String();
      message.data = "Hello, world! " + std::to_string(count_++);
      RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
      publisher_->publish(message);
    }
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    size_t count_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```

对上述代码进行简单分析：  
**头文件部分**

```cpp
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
```

**命名空间**

```cpp
using namespace std::chrono_literals;
```

**使用面向对象中的继承，类名为MinimalPublisher继承rclcpp::Node这个类，继承方式为public;**

```cpp
class MinimalPublisher : public rclcpp::Node
```

**私有成员有回调函数timer\_callback，计时器对象timer\_,发布者对象publisher\_,统计数量的count\_;**  
该timer\_callback函数是设置消息数据和实际发布消息的地方。该RCLCPP\_INFO宏确保每个发布的消息都打印到控制台。还有计时器、发布者和计数器字段的声明

```cpp
private:
    void timer_callback()
    {
      auto message = std_msgs::msg::String();
      message.data = "Hello, world! " + std::to_string(count_++);
      RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
      publisher_->publish(message);
    }
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    size_t count_;
```

**公有成员:类的构造函数用于对私有成员进行初始化；**  
公共构造函数命名节点minimal\_publisher并初始化count\_为 0。在构造函数内部，发布者使用String消息类型、主题名称topic和所需的队列大小进行初始化，以在发生备份时限制消息。接下来，timer\_被初始化，这会导致timer\_callback函数每秒执行两次。**this指的是该节点**

```cpp
public:
    MinimalPublisher()
    : Node("minimal_publisher"), count_(0)
    {
      publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
      timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
    }
```

**主函数部分**  
rclcpp::init初始化 ROS 2，并rclcpp::spin开始处理来自节点的数据，包括来自计时器的回调。

```cpp
int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```

**ROS2相比ROS1在语法上有了很大的改变，在编程思路还是一样。**

## 创建订阅者

```cpp
cd ~/dev_ws/src/cpp_package/src
touch publisher_member_function.cpp
```

将下面c++代码复制subscriber\_member\_function.cpp文件中

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
using std::placeholders::_1;

class MinimalSubscriber : public rclcpp::Node
{
  public:
    MinimalSubscriber()
    : Node("minimal_subscriber")
    {
      subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
    }

  private:
    void topic_callback(const std_msgs::msg::String::SharedPtr msg) const
    {
      RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
    }
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  rclcpp::shutdown();
  return 0;
}
```

对上述代码进行简单分析：  
头文件部分：

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
```

**和发布者代码一样，定义一个类MinimalSubscriber继承rclcpp::Node**

```cpp
class MinimalSubscriber : public rclcpp::Node
```

**私有成员：定义回调函数topic\_callback将接收的消息打印到终端，订阅者对象subscription\_；**

```cpp
private:
    void topic_callback(const std_msgs::msg::String::SharedPtr msg) const
    {
      RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
    }
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
```

**公有成员：与类名一样的构造函数对私有成员初始化，这里节点名是唯一标识，除非在不同的命名空间下可以有相同的节点名。this指的是该节点。**

```cpp
public:
    MinimalSubscriber()
    : Node("minimal_subscriber")
    {
      subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
    }
```

**主函数部分和发布者相差不大。**  
**修改CMakeLists.txt文件，将下面内容复制到文件中并保存。**

```bash
add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp std_msgs)

add_executable(listener src/subscriber_member_function.cpp)
ament_target_dependencies(listener rclcpp std_msgs)

install(TARGETS
  talker
  listener
  DESTINATION lib/${PROJECT_NAME})
```

## 编译

```bash
cd ~/ros2_ws/
colcon build --packages-select cpp_package
. install/setup.bash
```

## 运行

**启动发布者talker节点**

```bash
ros2 run cpp_package talker
```

结果：  
[![talker](https://www.guyuehome.com//Uploads/Editor/202109/20210910_66934.png "talker")](https://www.guyuehome.com//Uploads/Editor/202109/20210910_66934.png)

**启动订阅者listener节点**

```bash
ros2 run cpp_package listener
```

结果：  
[![listener](https://www.guyuehome.com//Uploads/Editor/202109/20210910_87501.png "listener")](https://www.guyuehome.com//Uploads/Editor/202109/20210910_87501.png)
