# Nexus论坛系统设计文档

## 项目概述

**Nexus** 是一个融合赛博朋克风格与现代化功能的下一代社区论坛系统。它不仅提供丰富的交流功能，还拥有令人印象深刻的视觉体验和流畅的用户交互。本系统采用PHP 8.x + Laravel 10.x作为核心技术栈，结合现代前端技术，打造一个高性能、高颜值的社区平台。

## 技术栈

### 后端
- **PHP 8.0+**：作为主要开发语言
- **Laravel 10.x**：提供MVC架构、Eloquent ORM、路由和中间件等功能
- **MySQL 8.0/MariaDB**：关系型数据库存储
- **Redis**：缓存、会话管理和排行榜数据
- **Elasticsearch**：全文搜索功能
- **WebSocket**：使用Swoole或Pusher实现实时通信

### 前端
- **Tailwind CSS**：快速构建炫酷、响应式UI
- **Alpine.js**：轻量级客户端交互
- **Animate.css/GSAP**：实现流畅的动画效果
- **Quill/TinyMCE**：富文本编辑器

## 系统架构

### 目录结构建议

```
app/
├── Console/              # 命令行工具
├── Exceptions/           # 异常处理
├── Http/
│   ├── Controllers/      # 控制器
│   │   ├── Admin/        # 管理员控制器
│   │   ├── API/          # API控制器
│   │   └── Frontend/     # 前端控制器
│   ├── Middleware/       # 中间件
│   ├── Requests/         # 表单请求验证
│   └── Resources/        # API资源
├── Models/               # 数据模型
├── Policies/             # 权限策略
├── Providers/            # 服务提供者
├── Services/             # 业务逻辑服务
└── Traits/               # 可复用特性
bootstrap/               # 应用启动文件
config/                  # 配置文件
database/
├── migrations/           # 数据库迁移
├── seeders/              # 数据填充
└── factories/            # 模型工厂
public/                  # 公共资源目录
resources/
├── css/                  # 样式文件
├── js/                   # JavaScript文件
└── views/                # 视图文件
    ├── admin/            # 后台视图
    ├── frontend/         # 前台视图
    └── components/       # 组件
routes/                  # 路由定义
storage/                 # 存储目录
vendor/                  # 第三方依赖
```

### 数据库设计

#### ER图关系

1. **用户(users)** 一对多 **帖子(threads)**
2. **用户(users)** 一对多 **评论(replies)**
3. **用户(users)** 一对多 **积分记录(point_records)**
4. **用户(users)** 一对多 **订单(orders)**
5. **用户(users)** 多对多 **点赞(likes)**
6. **版块(forums)** 一对多 **帖子(threads)**
7. **帖子(threads)** 一对多 **评论(replies)**
8. **虚拟商品(virtual_products)** 一对多 **订单(orders)**
9. **用户(users)** 多对多 **举报(reports)**

#### 详细表结构

```sql
-- 用户表
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    avatar VARCHAR(255) DEFAULT NULL,
    banner VARCHAR(255) DEFAULT NULL,
    signature TEXT,
    cyber_id VARCHAR(30) DEFAULT NULL UNIQUE,
    status ENUM('active', 'inactive', 'banned') DEFAULT 'inactive',
    role ENUM('user', 'moderator', 'admin') DEFAULT 'user',
    points INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login TIMESTAMP DEFAULT NULL,
    email_verified_at TIMESTAMP DEFAULT NULL,
    verification_token VARCHAR(255) DEFAULT NULL,
    two_factor_enabled BOOLEAN DEFAULT FALSE,
    two_factor_secret VARCHAR(255) DEFAULT NULL
);

-- OAuth社交账户表
CREATE TABLE social_accounts (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    provider VARCHAR(50) NOT NULL,
    provider_user_id VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY unique_social (provider, provider_user_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 用户徽章表
CREATE TABLE user_badges (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    badge_name VARCHAR(50) NOT NULL,
    badge_icon VARCHAR(255) NOT NULL,
    description TEXT,
    earned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 版块表
CREATE TABLE forums (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_id BIGINT DEFAULT NULL,
    icon VARCHAR(255) DEFAULT NULL,
    color VARCHAR(20) DEFAULT '#007bff',
    sort_order INT DEFAULT 0,
    is_archived BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_id) REFERENCES forums(id) ON DELETE CASCADE
);

-- 帖子表
CREATE TABLE threads (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    forum_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    content LONGTEXT NOT NULL,
    view_count INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    like_count INT DEFAULT 0,
    last_reply_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('normal', 'sticky', 'locked', '精华') DEFAULT 'normal',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (forum_id) REFERENCES forums(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 评论表
CREATE TABLE replies (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    thread_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    parent_id BIGINT DEFAULT NULL,
    content LONGTEXT NOT NULL,
    likes_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (thread_id) REFERENCES threads(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (parent_id) REFERENCES replies(id) ON DELETE CASCADE
);

-- 点赞表
CREATE TABLE likes (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    target_type ENUM('thread', 'reply') NOT NULL,
    target_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY unique_like (user_id, target_type, target_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 积分记录表
CREATE TABLE point_records (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    points INT NOT NULL,
    type ENUM('gain', 'lose') NOT NULL,
    reason VARCHAR(255),
    related_model VARCHAR(50) DEFAULT NULL,
    related_id BIGINT DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 虚拟商品表
CREATE TABLE virtual_products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price INT NOT NULL,
    stock INT DEFAULT 0,
    image VARCHAR(255),
    type ENUM('badge', 'title', 'theme', 'activation_code', 'ebook', 'physical') DEFAULT 'badge',
    delivery_type ENUM('auto', 'manual') DEFAULT 'auto',
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 订单表
CREATE TABLE orders (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT DEFAULT 1,
    total_points INT NOT NULL,
    status ENUM('pending', 'completed', 'cancelled') DEFAULT 'pending',
    shipping_address TEXT DEFAULT NULL,
    delivery_note TEXT DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES virtual_products(id) ON DELETE CASCADE
);

-- 举报表
CREATE TABLE reports (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    reporter_id BIGINT NOT NULL,
    target_type ENUM('thread', 'reply', 'user') NOT NULL,
    target_id BIGINT NOT NULL,
    reason TEXT NOT NULL,
    status ENUM('pending', 'processed', 'dismissed') DEFAULT 'pending',
    processed_by BIGINT DEFAULT NULL,
    processing_notes TEXT DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (reporter_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (processed_by) REFERENCES users(id) ON DELETE SET NULL
);

-- 系统日志表
CREATE TABLE logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT DEFAULT NULL,
    action VARCHAR(100) NOT NULL,
    description TEXT,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);

-- 版主版块关联表
CREATE TABLE moderators (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    forum_id BIGINT NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    assigned_by BIGINT DEFAULT NULL,
    UNIQUE KEY unique_moderator (user_id, forum_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (forum_id) REFERENCES forums(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_by) REFERENCES users(id) ON DELETE SET NULL
);
```

## 核心功能模块

### 1. 用户系统 - "身份矩阵"

#### 功能设计
- **多方式注册/登录**
  - 邮箱/密码注册（带邮箱验证）
  - 社交账号登录（微信、QQ、GitHub等）
  - 双因素认证（2FA）
  
- **个人资料管理**
  - 自定义头像、个人横幅
  - 个人签名、赛博ID
  - 成就徽章展示
  
- **安全设置**
  - 密码修改
  - 绑定/解绑社交账号
  - 2FA开关设置

#### API接口
```
POST /api/auth/register           - 用户注册
POST /api/auth/login              - 用户登录
POST /api/auth/logout             - 用户登出
POST /api/auth/social/{provider}  - 社交登录
POST /api/auth/verify-email       - 邮箱验证
GET  /api/users/me                - 获取当前用户信息
PUT  /api/users/me                - 更新个人资料
POST /api/users/avatar            - 上传头像
PUT  /api/users/password          - 修改密码
```

### 2. 论坛核心 - "话题枢纽"

#### 功能设计
- **版块管理**
  - 无限层级版块创建
  - 自定义图标和颜色主题
  - 访问权限控制
  
- **帖子管理**
  - 富文本编辑器发布帖子
  - 支持Markdown、代码高亮
  - 图片/视频上传
  - 帖子置顶、加精、锁定
  
- **评论系统**
  - 多级评论嵌套
  - @用户提及功能
  - 实时评论更新
  - 评论点赞

#### API接口
```
GET    /api/forums                - 获取版块列表
GET    /api/forums/{id}           - 获取版块详情
GET    /api/threads               - 获取帖子列表
POST   /api/threads               - 创建帖子
GET    /api/threads/{id}          - 获取帖子详情
PUT    /api/threads/{id}          - 更新帖子
DELETE /api/threads/{id}          - 删除帖子
GET    /api/threads/{id}/replies  - 获取帖子评论
POST   /api/threads/{id}/replies  - 发布评论
POST   /api/likes                 - 点赞
DELETE /api/likes                 - 取消点赞
```

### 3. 管理后台系统 - "神之视角"

#### 功能设计
- **仪表盘**
  - 实时数据统计图表
  - 最新活动监控
  - 系统状态显示
  
- **用户管理**
  - 用户列表查询
  - 编辑/封禁用户
  - 用户行为日志查看
  
- **内容管理**
  - 帖子审核
  - 评论管理
  - 举报处理
  
- **版块管理**
  - 拖拽排序
  - 设置版块属性
  
- **系统设置**
  - 基本信息配置
  - 积分规则设置
  - 邮件服务配置

#### API接口
```
GET    /api/admin/dashboard       - 获取仪表盘数据
GET    /api/admin/users           - 获取用户列表
PUT    /api/admin/users/{id}      - 更新用户状态
GET    /api/admin/reports         - 获取举报列表
PUT    /api/admin/reports/{id}    - 处理举报
GET    /api/admin/forums          - 获取版块列表
POST   /api/admin/forums          - 创建版块
PUT    /api/admin/forums/{id}     - 更新版块
DELETE /api/admin/forums/{id}     - 删除版块
```

### 4. 子管理员系统 - "区域守护者"

#### 功能设计
- **版主分配**
  - 指定用户为特定版块版主
  - 设置版主权限范围
  
- **版主管理功能**
  - 版块内帖子管理（置顶、锁定等）
  - 评论管理（删除违规评论）
  - 用户禁言（限制在特定版块发言）
  
- **简化管理界面**
  - 专注于所管辖版块的管理面板
  - 快速处理工具

#### API接口
```
GET    /api/moderator/forums              - 获取版主管理的版块
PUT    /api/moderator/threads/{id}        - 管理帖子状态
DELETE /api/moderator/replies/{id}        - 删除违规评论
POST   /api/moderator/mute                - 禁言用户
```

### 5. 热门排行榜 - "荣耀殿堂"

#### 功能设计
- **多维榜单**
  - 用户积分榜（日/周/月/总）
  - 活跃度榜
  - 获赞榜
  - 热门话题榜
  - 最热评论榜
  
- **动态更新**
  - 定时更新机制
  - 更新动画效果
  
- **特殊标识**
  - 前三名特殊标识
  - 趋势变化显示

#### API接口
```
GET /api/rankings/users/points        - 用户积分榜
GET /api/rankings/users/activity      - 用户活跃度榜
GET /api/rankings/threads/popular     - 热门话题榜
GET /api/rankings/replies/likes       - 热门评论榜
```

### 6. 积分与商城系统 - "价值交易所"

#### 功能设计
- **积分机制**
  - 发帖/评论获得积分
  - 每日登录奖励
  - 帖子被点赞/加精奖励
  - 积分消费记录
  
- **虚拟商品**
  - 独家徽章
  - 自定义头衔
  - 主题皮肤
  - 激活码
  - 电子书
  - 实体商品（可选）
  
- **订单管理**
  - 自动发货（虚拟商品）
  - 手动处理（实体商品）
  - 订单状态跟踪

#### API接口
```
GET    /api/shop/products          - 获取商品列表
GET    /api/shop/products/{id}     - 获取商品详情
POST   /api/shop/buy/{product_id}  - 购买商品
GET    /api/shop/orders            - 获取订单列表
GET    /api/users/points/history   - 获取积分历史
```

## 关键功能实现思路

### 1. Redis实现排行榜

排行榜是高并发场景下的常见需求，使用Redis的有序集合(Sorted Set)可以高效实现：

```php
// 记录用户积分变化
public function updateUserPoints($userId, $points, $reason)
{
    // 1. 更新数据库用户积分
    DB::transaction(function () use ($userId, $points, $reason) {
        $user = User::findOrFail($userId);
        $user->points += $points;
        $user->save();
        
        // 2. 记录积分变动历史
        PointRecord::create([
            'user_id' => $userId,
            'points' => $points,
            'type' => $points > 0 ? 'gain' : 'lose',
            'reason' => $reason
        ]);
    });
    
    // 3. 更新Redis排行榜
    $redis = Redis::connection();
    // 更新总积分排行榜
    $redis->zadd('rankings:points:all', $user->points, $userId);
    // 更新周积分排行榜（使用带过期时间的键）
    $redis->zadd('rankings:points:week', $points, $userId);
    
    // 设置周排行榜过期时间为7天
    $redis->expire('rankings:points:week', 7 * 24 * 60 * 60);
}

// 获取排行榜数据
public function getRanking($type = 'points', $period = 'all', $limit = 10)
{
    $redis = Redis::connection();
    $key = "rankings:{$type}:{$period}";
    
    // 获取排名数据（分数从高到低）
    $rankingData = $redis->zrevrange($key, 0, $limit - 1, 'WITHSCORES');
    
    // 转换为用户数据
    $userIds = array_keys($rankingData);
    $users = User::whereIn('id', $userIds)->get()->keyBy('id');
    
    $result = [];
    $rank = 1;
    foreach ($rankingData as $userId => $score) {
        if (isset($users[$userId])) {
            $result[] = [
                'rank' => $rank++,
                'user' => $users[$userId],
                'score' => $score
            ];
        }
    }
    
    return $result;
}
```

### 2. 虚拟商品自动发货逻辑

虚拟商品的自动发货是提升用户体验的关键功能：

```php
// 处理订单和自动发货
public function processOrder($userId, $productId, $quantity = 1)
{
    return DB::transaction(function () use ($userId, $productId, $quantity) {
        // 1. 获取用户和商品信息
        $user = User::lockForUpdate()->findOrFail($userId);
        $product = VirtualProduct::lockForUpdate()->findOrFail($productId);
        
        // 2. 验证库存和积分
        if ($product->stock < $quantity) {
            throw new Exception('商品库存不足');
        }
        
        $totalPoints = $product->price * $quantity;
        if ($user->points < $totalPoints) {
            throw new Exception('积分不足');
        }
        
        // 3. 创建订单
        $order = Order::create([
            'user_id' => $userId,
            'product_id' => $productId,
            'quantity' => $quantity,
            'total_points' => $totalPoints,
            'status' => 'pending'
        ]);
        
        // 4. 扣除用户积分
        $user->points -= $totalPoints;
        $user->save();
        
        // 记录积分变动
        PointRecord::create([
            'user_id' => $userId,
            'points' => -$totalPoints,
            'type' => 'lose',
            'reason' => "购买商品：{$product->name}",
            'related_model' => 'order',
            'related_id' => $order->id
        ]);
        
        // 5. 减少库存
        $product->stock -= $quantity;
        $product->save();
        
        // 6. 自动发货（如果是自动发货类型）
        if ($product->delivery_type === 'auto') {
            $this->deliverVirtualProduct($order, $user, $product, $quantity);
            $order->status = 'completed';
            $order->save();
        }
        
        return $order;
    });
}

// 虚拟商品发货逻辑
private function deliverVirtualProduct($order, $user, $product, $quantity)
{
    switch ($product->type) {
        case 'badge':
            // 发放徽章
            UserBadge::create([
                'user_id' => $user->id,
                'badge_name' => $product->name,
                'badge_icon' => $product->image,
                'description' => $product->description
            ]);
            break;
            
        case 'title':
            // 设置自定义头衔
            $user->custom_title = $product->name;
            $user->save();
            break;
            
        case 'activation_code':
            // 生成并发送激活码（这里简化处理，实际应从激活码池获取）
            $activationCode = $this->generateActivationCode();
            // 保存激活码并发送给用户
            ActivationCode::create([
                'user_id' => $user->id,
                'order_id' => $order->id,
                'code' => $activationCode,
                'product_name' => $product->name
            ]);
            // 发送邮件通知用户
            Mail::to($user->email)->send(new ActivationCodeMail($activationCode, $product));
            break;
            
        // 其他类型的虚拟商品处理...
    }
}
```

### 3. 实时通知系统

使用WebSocket实现实时通知和在线状态：

```php
// Laravel Event 定义
class NewReplyNotification implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
    
    public $reply;
    public $thread;
    
    public function __construct(Reply $reply)
    {
        $this->reply = $reply;
        $this->thread = $reply->thread;
    }
    
    public function broadcastOn()
    {
        // 向帖子作者和被@的用户发送通知
        $channels = [new PrivateChannel('user.' . $this->thread->user_id)];
        
        // 从评论内容中提取被@的用户
        preg_match_all('/@(\w+)/', $this->reply->content, $matches);
        if (!empty($matches[1])) {
            $usernames = $matches[1];
            $users = User::whereIn('username', $usernames)->get();
            
            foreach ($users as $user) {
                $channels[] = new PrivateChannel('user.' . $user->id);
            }
        }
        
        return $channels;
    }
    
    public function broadcastWith()
    {
        return [
            'id' => $this->reply->id,
            'thread_id' => $this->thread->id,
            'thread_title' => $this->thread->title,
            'content' => $this->reply->content,
            'user' => [
                'id' => $this->reply->user->id,
                'username' => $this->reply->user->username,
                'avatar' => $this->reply->user->avatar
            ],
            'created_at' => $this->reply->created_at->format('Y-m-d H:i:s')
        ];
    }
}
```

## 前端设计概念

### 首页视觉概念

**Nexus** 首页采用赛博朋克风格与极简设计的完美融合：

- **整体布局**：采用分层设计，深色背景上的霓虹元素创造深度感
- **导航栏**：固定在顶部，半透明效果，滚动时变化透明度
- **顶部横幅**：动态粒子效果背景，配以霓虹色品牌标志
- **版块展示区**：
  - 磁贴式布局，每个版块卡片具有独特颜色
  - 鼠标悬停时卡片轻微放大，显示版块内最新话题预览
  - 版块图标使用发光效果
- **热门内容区**：
  - 滚动展示热门话题，带有阅读量和评论数统计
  - 支持左右滑动或自动轮播
- **实时动态流**：
  - 展示最新发帖和评论
  - 新内容出现时带有优雅的滑入动画
- **排行榜区域**：
  - 游戏风格的排行榜设计
  - 前三名有特殊的皇冠/奖牌动画
- **页脚**：带有社交媒体链接和版权信息

### 商城页面视觉概念

商城页面设计为一个未来感十足的数字市场：

- **背景**：深色渐变背景，点缀网格线条和粒子效果
- **商品展示**：
  - 3D卡片式布局，商品图片占据主要位置
  - 价格使用霓虹色调突出显示
  - 鼠标悬停时卡片轻微旋转，显示更多详情
- **商品分类**：
  - 左侧边栏提供分类导航
  - 支持标签筛选和排序
- **商品详情**：
  - 点击商品卡片弹出模态框
  - 模态框使用半透明背景，内容区域有发光边框
  - 购买按钮带有脉动动画效果
- **用户积分显示**：
  - 顶部显示用户当前积分，带有数字变化动画
  - 积分不足时购买按钮变为红色并禁用
- **订单历史**：
  - 简洁的时间线布局
  - 不同状态的订单使用不同颜色标识

## 使用phpStudy Pro搭建项目

### 安装phpStudy Pro

1. **下载安装**
   - 访问[phpStudy Pro官网](https://www.xp.cn/)
   - 下载最新版本的phpStudy Pro
   - 按照安装向导完成安装

2. **启动服务**
   - 打开phpStudy Pro
   - 启动Apache/Nginx和MySQL服务
   - 确保PHP版本为8.0或更高

### 配置项目环境

1. **创建数据库**
   - 在phpStudy Pro面板中，点击「数据库」选项卡
   - 点击「创建数据库」按钮
   - 输入数据库名称：`php_forum`
   - 输入用户名：`root`
   - 输入密码：`password`（或自定义）
   - 点击「确定」创建数据库

2. **导入数据库结构**
   - 点击刚创建的数据库右侧的「管理」按钮
   - 进入phpMyAdmin界面
   - 选择「导入」选项卡
   - 上传`database/init.sql`文件
   - 点击「执行」完成数据库导入

3. **配置网站**
   - 在phpStudy Pro面板中，点击「网站」选项卡
   - 点击「创建网站」按钮
   - 域名：`php-forum.test`（可自定义）
   - 端口：默认80
   - 根目录：选择项目的`public`目录
   - PHP版本：选择8.0或更高
   - 数据库：选择刚创建的`php_forum`
   - 点击「确定」完成网站创建

4. **配置hosts文件**
   - 点击phpStudy Pro右上角的「其他选项菜单」→「工具」→「hosts文件」
   - 在hosts文件中添加一行：`127.0.0.1 php-forum.test`
   - 保存并关闭

### 安装依赖

1. **安装Composer依赖**
   - 确保phpStudy Pro中已安装Composer
   - 打开项目根目录的命令行工具
   - 执行命令：`composer install`

2. **配置环境变量**
   - 复制`.env.example`文件并重命名为`.env`
   - 编辑`.env`文件，配置数据库连接：
     ```
     DB_CONNECTION=mysql
     DB_HOST=127.0.0.1
     DB_PORT=3306
     DB_DATABASE=php_forum
     DB_USERNAME=root
     DB_PASSWORD=password
     ```

3. **生成应用密钥**
   - 执行命令：`php artisan key:generate`

### 启动项目

1. **运行迁移和填充（如需要）**
   - `php artisan migrate`
   - `php artisan db:seed`

2. **启动本地开发服务器**
   - 执行命令：`php artisan serve`
   - 或直接通过配置的域名访问：`http://php-forum.test`

3. **配置队列和定时任务（生产环境）**
   - 队列：`php artisan queue:work`
   - 定时任务：配置系统crontab
     ```
     * * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
     ```

### 常见问题排查

1. **数据库连接失败**
   - 检查`.env`文件中的数据库配置是否正确
   - 确认MySQL服务已启动
   - 验证数据库用户权限

2. **网站无法访问**
   - 检查Apache/Nginx服务是否正常运行
   - 确认hosts文件配置正确
   - 验证网站根目录指向是否正确

3. **权限错误**
   - 确保storage和bootstrap/cache目录有写入权限
   - 执行：`chmod -R 775 storage bootstrap/cache`

4. **Composer依赖安装失败**
   - 检查PHP版本是否满足要求
   - 确认网络连接正常
   - 尝试使用镜像源：`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`

## 总结

**Nexus** 论坛系统通过融合赛博朋克风格的视觉设计与现代化的技术实现，为用户提供一个既炫酷又实用的社区平台。系统采用PHP 8.x + Laravel 10.x作为核心技术栈，配合Redis、Elasticsearch等组件，确保高性能和良好的用户体验。

通过本设计文档，您可以了解系统的整体架构、数据库设计、核心功能模块以及实现思路。使用phpStudy Pro可以快速搭建开发环境，加速项目的开发和部署过程。

随着社区的发展，系统可以进一步扩展更多功能，如实时聊天、富媒体内容支持、AI内容审核等，为用户提供更加丰富的社区体验。