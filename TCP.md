# 图解TCP、IP
## 第一章：    
### 1.2:计算机发展阶段    
##### 概念：
- **分时系统**：多个终端与同一个计算机链接，允许多个用户同时使用一台计算机的系统。分时系统特性：多路性、独占性、交互性和及时性
- **ISO**:International Organization for Standards 国际标准化组织    
- **OSI**:Open System Interconnection 开放式通信系统互联参考模型    
- **OSI参考模型**：7层模型 7-应用层，6-表示层，5-会话层，4-传输层，3-网络层，2-数据链路层，1-物理层。    
- **应用层**：为应用程序提供服务并规定应用程序中通信相关的细节。例如：文件传输，电子邮件，远程登陆等协议。    
- **表示层**：将应用处理的信息转换为适合网络传输的格式，或将来自下一层的数据转换为上层能够出里的格式。因此它主要负责数据格式的转换。
- **会话层**：负责建立和断开通信链接（数据流动的逻辑通路），以及数据的分割等数据传输相关的管理工作。    
- **传输层**：管理两个节点之间的数据传输，负责可靠数据传输，只要在通信双方节点上进行处理，无需在路由器上处理。    
- **网络层**：地址管理与路由选择，将数据传输到目标地址。    
- **数据链路层**：互联设备之间传送和识别数据帧，负责无力层面上互连的，节点之间的通信传输。
- **物理层**：比特流与电子信号之间的切换。负责0、1比特流（0、1序列）与电压高低、光的闪灭之间的呼唤。