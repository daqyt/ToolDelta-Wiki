## 插件开发

#### 安装 ToolDelta 库

你需要使用 Python 3.10+ 配置 ToolDelta 的插件开发环境。
使用 pip 安装 ToolDelta：
```sh
pip install tooldelta
```

进入 ToolDelta 工作目录， 即可进入插件开发环境！
如果想编写需求功能较多的插件， 可以开发**ToolDelta类式插件**，其使用同步模式进行开发（推荐）。  
如果插件功能轻量较少， 可以开发**ToolDelta注入式插件**，其使用异步模式进行开发。  

| [类式插件开发教程](./class-plugin/创建插件) | [注入式插件开发教程](./injected-plugin/编写教程) |
| -- | -- |