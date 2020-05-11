### **技术选型** 
### **前端项目技术栈**

- Vue
- Vue-router
- Element-UI
- Axios
- Echarts
 **项目初始化** 
 **前端项目初始化步骤** 

1. 安装 Vue 脚手架
2. 通过 Vue-Cli 创建项目
3. 配置 Vue-router
4. 配置 Element-UI 组件库
5. 配置 Axios 库
6. 初始化 git 远程仓库
 **登录概述** 
 **登录业务流程** 

1. 在登录页面输入用户名和密码
2. 调用后台接口进行验证
3. 通过验证之后,根据后台的响应状态跳转到项目主页

 **登录业务相关技术点** 

1. http是无状态的
2. 通过cookie在客户端记录状态
3. 通过sesion在服务器端记录状态
4. 通过token维持状态(不允许跨域使用)



 **登录业务流程** 
 **登录页面的布局** 
通过Element-UI组件实现布局


- el-form
- el-form-item
- el-input
- el-button
- 字体图标

