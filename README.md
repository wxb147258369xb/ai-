# Nexus 赛博朋克风格社区论坛

## 项目简介

Nexus 是一个具有未来感的赛博朋克风格社区论坛，融合了现代Web技术和视觉设计，打造出一个独特而沉浸式的用户体验。本项目使用PHP作为后端核心，结合Tailwind CSS实现炫酷的视觉效果，以及JavaScript提供流畅的交互体验。

## 技术栈

- **后端**: PHP 8.x
- **前端**: HTML5, CSS3, JavaScript, Tailwind CSS
- **数据库**: MySQL 5.7+
- **服务器环境**: 推荐使用phpStudy Pro进行本地开发

## 系统功能

- **用户系统**: 注册登录、个人资料、头像上传
- **论坛核心**: 版块管理、发帖回帖、富文本编辑器
- **积分系统**: 用户积分获取与消费
- **积分商城**: 虚拟商品兑换
- **排行榜**: 用户积分排行、热门话题排行
- **后台管理**: 用户管理、内容审核、系统设置

## 使用 phpStudy Pro 搭建环境

### 1. 安装 phpStudy Pro

1. 访问 phpStudy Pro 官网下载最新版本：[https://www.xp.cn/](https://www.xp.cn/)
2. 按照安装向导完成安装
3. 启动 phpStudy Pro

### 2. 创建网站环境

1. 在 phpStudy Pro 主界面，点击左侧的「网站」选项卡
2. 点击「创建网站」按钮
3. 填写以下信息：
   - **网站名称**: Nexus 论坛
   - **域名**: 可使用默认的 `localhost` 或设置为 `nexus.test`
   - **端口**: 默认 80
   - **根目录**: 选择本项目的 `public` 文件夹路径 (例如：`D:\php论坛\public`)
   - **PHP版本**: 选择 PHP 8.0 或更高版本
   - **数据库**: MySQL 5.7+ 或 MariaDB
   - **数据库名**: `php_forum`
   - **用户名**: `root`
   - **密码**: 设置数据库密码

### 3. 导入数据库

1. 在 phpStudy Pro 主界面，点击左侧的「数据库」选项卡
2. 找到刚刚创建的数据库 `php_forum`
3. 点击「管理」按钮，进入 phpMyAdmin
4. 点击「导入」选项卡
5. 选择本项目中的 `database/init.sql` 文件
6. 点击「执行」按钮，等待导入完成

### 4. 配置文件

1. 找到项目中的 `app/config/database.php` 文件
2. 根据您的数据库设置修改以下参数：
   ```php
   'host' => 'localhost',     // 数据库主机
   'dbname' => 'php_forum',   // 数据库名称
   'username' => 'root',      // 数据库用户名
   'password' => 'your_password', // 数据库密码
   ```

3. 修改 `app/config/app.php` 文件中的基本配置：
   ```php
   'app_name' => 'Nexus 论坛', // 应用名称
   'app_version' => '1.0.0',   // 应用版本
   'debug' => true,           // 调试模式 (生产环境请设置为 false)
   ```

### 5. 启动网站

1. 返回 phpStudy Pro 主界面
2. 确保网站和数据库服务都已启动（状态为绿色）
3. 打开浏览器，访问您设置的域名（例如：`http://localhost` 或 `http://nexus.test`）
4. 如果一切配置正确，您将看到 Nexus 论坛的首页

## 管理员账户

导入数据库后，系统会自动创建一个默认的管理员账户：

- **用户名**: `admin`
- **密码**: `admin123`

首次登录后，请立即修改管理员密码以确保安全。

## 目录结构

```
php论坛/
├── app/                     # 应用核心代码
│   ├── controllers/         # 控制器
│   ├── models/              # 数据模型
│   ├── views/               # 视图模板
│   │   ├── layouts/         # 布局文件
│   │   └── ...              # 其他视图文件
│   ├── helpers/             # 辅助函数
│   ├── middleware/          # 中间件
│   └── config/              # 配置文件
├── public/                  # 公共资源目录
│   ├── css/                 # CSS样式文件
│   ├── js/                  # JavaScript文件
│   ├── images/              # 图片资源
│   ├── uploads/             # 上传文件目录
│   └── index.php            # 网站入口文件
├── database/                # 数据库相关
│   └── init.sql             # 数据库初始化脚本
├── docs/                    # 文档
│   └── Nexus论坛系统设计文档.md  # 系统设计文档
└── README.md                # 项目说明文档
```

## 开发说明

### 添加新页面

1. 在 `app/controllers/` 目录下创建新的控制器
2. 在 `app/views/` 目录下创建对应的视图文件
3. 在 `public/index.php` 中注册新的路由

### 自定义样式

- 主样式文件位于 `public/css/cyberpunk.css`
- 可以在 `views/layouts/header.php` 中修改全局样式设置

### 添加交互功能

- JavaScript功能在 `public/js/cyberpunk.js` 中实现
- 可以通过在 `DOMContentLoaded` 事件中添加新的初始化函数来扩展功能

## 常见问题

### 1. 无法连接数据库

- 检查 `app/config/database.php` 中的数据库配置是否正确
- 确认 phpStudy Pro 中的数据库服务是否已启动
- 验证数据库用户名和密码是否正确

### 2. 页面显示500错误

- 确保PHP版本至少为8.0
- 检查是否有语法错误或缺少必要的PHP扩展
- 打开 `app/config/app.php` 中的调试模式 `debug => true` 查看详细错误信息

### 3. 图片上传失败

- 检查 `public/uploads/` 目录是否存在且具有写入权限
- 确认PHP配置中的 `upload_max_filesize` 和 `post_max_size` 设置是否足够大

### 4. 路由无法访问

- 确保在 `public/index.php` 中正确注册了路由
- 检查控制器和方法名称是否正确

## 安全建议

1. 生产环境中禁用调试模式 (`debug => false`)
2. 定期备份数据库
3. 使用HTTPS协议
4. 定期更新PHP和相关依赖库
5. 实施输入验证，防止SQL注入和XSS攻击

## 许可证

本项目仅供学习和参考，未经授权不得用于商业用途。

## 致谢

感谢所有为这个项目提供灵感和帮助的开发者！

---

如有任何问题或建议，请随时联系项目维护者。祝您使用愉快！