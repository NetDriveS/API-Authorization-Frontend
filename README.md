# API Authorization Frontend

现代化 API 授权管理前端应用，采用 macOS/iOS 设计风格。

## 项目概述

项目形态：Web 前端应用
技术栈：HTML5 + CSS3 + JavaScript (原生) + Font Awesome + Google Fonts

## 项目简介

这是一个面向 API 授权管理的前端应用。它不只是简单的登录表单，而是将多个管理功能整合在一起：

- 用户认证功能
- 应用管理功能
- API 密钥管理
- 操作日志追踪
- 主题切换功能

## 开发初衷

API 授权管理在实际应用中常常面临这些问题：

- 密钥生成散落在各处
- 权限配置不直观
- 缺少统一的操作日志
- 界面风格不统一

基于这些痛点，我们希望做一个工具，把"认证 → 创建应用 → 配置权限 → 密钥管理"这个流程串联起来。

## 已实现功能

### 🔐 认证系统

- 用户名/密码登录
- 双因素认证 (2FA) 支持
- 社交登录 (Google, GitHub, Apple)
- 记住登录状态

### 📱 用户面板

- 应用管理 (创建、查看、删除)
- 安全设置 (修改密码)
- 操作日志 (登录、创建、删除记录)

### 🔑 API 密钥管理

- 自动生成 API 密钥和密钥
- 权限配置 (读取、写入、删除、管理员、邮件通知)
- Token 有效期设置
- 密钥显示/复制功能

### 🎨 界面特色

- macOS/iOS 风格设计
- 玻璃拟态效果
- 亮色/暗色主题切换
- 响应式布局

## 技术架构

### 文件结构

```
auth-portal/
├── index.html              # 主入口页面
├── styles.css             # 样式文件
├── app.js                # 主应用逻辑
├── component-loader.js   # 组件加载器
├── components/           # UI 组件目录
│   ├── header.html          # 主题头部
│   ├── login-form.html      # 登录表单
│   ├── forgot-password-form.html  # 忘记密码
│   ├── register-form.html   # 注册表单
│   ├── social-login.html    # 社交登录
│   ├── user-panel.html      # 用户面板
│   ├── app-detail.html      # 应用详情
│   ├── permissions.html      # 权限配置
│   ├── success.html         # 授权成功页
│   ├── modals.html          # 模态框集合
│   └── footer.html          # 页脚
└── README.md
```

### 组件化架构

`component-loader.js` 负责动态加载 HTML 组件：

1. **缓存机制**: 组件只加载一次，后续从缓存获取
2. **data-component 属性**: HTML 中使用 `data-component="组件名"` 标记
3. **componentsLoaded 事件**: 所有组件加载完成后派发，触发初始化

### 组件列表

| 组件                   | 文件                        | 职责                       |
| -------------------- | ------------------------- | ------------------------ |
| header               | header.html               | 主题头部，Logo，主题切换按钮         |
| login-form           | login-form.html           | 用户名/密码登录，2FA 输入          |
| forgot-password-form | forgot-password-form.html | 忘记密码表单                   |
| register-form        | register-form.html        | 用户注册表单                   |
| social-login         | social-login.html         | Google/GitHub/Apple 登录按钮 |
| user-panel           | user-panel.html           | 用户仪表盘，应用列表，标签页           |
| app-detail           | app-detail.html           | 应用详情，API 密钥显示            |
| permissions          | permissions.html          | 权限配置，Token 有效期           |
| success              | success.html              | 授权成功确认页                  |
| modals               | modals.html               | 所有模态框（删除确认、2FA设置等）       |
| footer               | footer.html               | 页脚                       |

### 代码架构

#### app.js 模块结构

```
app.js
├── APIService          # API 服务层
│   ├── _tryRefreshToken()   # Token 自动刷新
│   ├── request()            # 统一请求方法
│   └── login/logout/register # 认证 API
│
├── StateManager       # 状态管理
│   ├── state              # 应用状态
│   ├── getToken/setToken   # Token 管理
│   └── subscribers        # 状态订阅
│
├── UI Controllers
│   ├── setupTheme()        # 主题切换
│   ├── setupModals()      # 模态框管理
│   ├── setupDeleteModal() # 删除确认弹窗
│   ├── setupForms()       # 表单处理
│   ├── setupUserPanel()   # 用户面板
│   ├── setupAppDetail()   # 应用详情
│   └── setupSocialLogin() # 社交登录
│
├── 工具函数
│   ├── maskKey()          # 密钥脱敏
│   ├── formatDateTime()   # 日期格式化
│   ├── showToast()        # 提示消息
│   └── validateEmail()    # 邮箱验证
│
└── Initialize          # 初始化入口
```

#### StateManager 状态管理

```javascript
StateManager.state = {
    currentView: 'login',      // 当前视图
    user: null,                // 用户信息
    token: null,               // 认证令牌
    rememberMe: false,         // 记住登录
    permissions: {             // 权限配置
        read: true,
        write: false,
        delete: false,
        admin: false,
        webhooks: false
    },
    expiry: '24h',             // Token有效期
    apiKey: '',                // API密钥
    secretKey: '',             // 密钥
    apps: [],                  // 应用列表
    activities: [],            // 活动记录
    lastLoginTime: '',         // 最后登录时间
    currentEditingAppId: null, // 当前编辑的应用ID
    isCreatingApp: false       // 创建应用防重复
}
```

#### StateManager 方法

| 方法                    | 说明         |
| --------------------- | ---------- |
| `getToken()`          | 获取当前认证令牌   |
| `setToken(token)`     | 设置认证令牌     |
| `clearAuth()`         | 清空认证状态     |
| `setState(updates)`   | 更新状态并通知监听器 |
| `subscribe(listener)` | 添加状态变更监听器  |

## API 集成说明

应用采用 `APIService` 模块化架构，对接 auth-core 后端。

### API 接口清单

| 方法     | 端点                         | 说明     |
| ------ | -------------------------- | ------ |
| POST   | /auth/login                | 用户登录   |
| POST   | /auth/logout               | 用户登出   |
| POST   | /auth/register             | 用户注册   |
| POST   | /auth/refresh              | 刷新令牌   |
| GET    | /auth/profile              | 获取用户信息 |
| PUT    | /auth/password             | 更新密码   |
| GET    | /apps                      | 获取应用列表 |
| POST   | /apps                      | 创建应用   |
| DELETE | /apps/{id}                 | 删除应用   |
| POST   | /apps/{id}/keys/regenerate | 重新生成密钥 |
| GET    | /activities                | 获取活动记录 |

## 权限说明

### 权限类型

| 权限   | 标识       | 说明       |
| ---- | -------- | -------- |
| 读取   | read     | 读取数据权限   |
| 写入   | write    | 写入数据权限   |
| 删除   | delete   | 删除数据权限   |
| 管理员  | admin    | 完全管理控制权  |
| 邮件通知 | webhooks | 接收事件实时通知 |

### Token 有效期

| 值     | 说明    |
| ----- | ----- |
| 1h    | 1 小时  |
| 24h   | 24 小时 |
| 7d    | 7 天   |
| 30d   | 30 天  |
| 90d   | 90 天  |
| never | 永不过期  |

## 工具函数

| 函数                                | 说明                           |
| --------------------------------- | ---------------------------- |
| `generateAPIKey()`                | 生成 API 密钥 (sk\_live\_ 前缀)    |
| `generateSecretKey()`             | 生成 40 位随机密钥                  |
| `formatDate(isoString)`           | 格式化日期为 YYYY-MM-DD            |
| `formatDateTime(isoString)`       | 格式化日期时间为 YYYY-MM-DD HH:mm:ss |
| `formatTimeAgo(isoString)`        | 格式化相对时间 (刚刚、分钟前等)            |
| `maskKey(key)`                    | 隐藏密钥中间部分                     |
| `getPermissionLabel(permissions)` | 获取权限标签文本                     |

## 使用流程

1. **登录/注册**
   - 输入用户名和密码
   - 可选择启用 2FA 认证
   - 点击"临时登录"进入
2. **创建应用**
   - 点击"创建新应用"
   - 配置 API 权限
   - 设置 Token 有效期
   - 确认授权
3. **管理应用**
   - 查看已授权应用列表
   - 点击"重新进入"修改配置
   - 删除不需要的应用

## 项目成果

这个项目现在已经不只是"登录表单"，而是一个：

- API 授权管理平台
- 密钥生成与管理工具
- 操作日志追踪系统
- 主题切换管理界面

最值得称赞的是，它已经把原本分散在多个工具里的授权流程，收进了一个应用里。

## 浏览器兼容性

- Chrome (最新版本)
- Firefox (最新版本)
- Safari (最新版本)
- Edge (最新版本)

## 许可证

MIT License
