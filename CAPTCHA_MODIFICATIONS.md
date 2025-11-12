# NocoBase 登录验证码功能修改说明

## 修改概述

本次修改为 NocoBase 的登录功能增加了验证码验证，并修改了默认管理员密码，提高了系统的安全性。

## 主要修改内容

### 1. 后端修改

#### 1.1 创建验证码服务 (`packages/plugins/@nocobase/plugin-auth/src/server/captcha.ts`)
- 实现了验证码生成功能
- 生成 4 位随机字母数字组合的验证码
- 创建 SVG 格式的验证码图片
- 验证码存储在 session 中，有效期 5 分钟
- 提供验证码验证功能

#### 1.2 修改基础认证类 (`packages/plugins/@nocobase/plugin-auth/src/server/basic-auth.ts`)
- 在 `validate()` 方法中增加验证码验证逻辑
- 登录时必须提供验证码
- 验证码验证失败时返回相应错误信息

#### 1.3 添加验证码生成 API (`packages/plugins/@nocobase/plugin-auth/src/server/actions/auth.ts`)
- 新增 `generateCaptcha` action
- 提供验证码生成接口

#### 1.4 配置权限 (`packages/plugins/@nocobase/plugin-auth/src/server/plugin.ts`)
- 为验证码生成接口配置公开访问权限

#### 1.5 修改默认管理员密码 (`packages/plugins/@nocobase/plugin-users/src/server/server.ts`)
- 将默认密码从 `admin123` 修改为 `NocoBase@2024!`
- 提高了密码复杂度，符合安全要求

### 2. 前端修改

#### 2.1 修改登录表单 (`packages/plugins/@nocobase/plugin-auth/src/client/basic/SignInForm.tsx`)
- 添加验证码输入框
- 集成验证码图片显示组件
- 支持验证码刷新功能
- 添加验证码验证规则

#### 2.2 添加中文翻译 (`packages/plugins/@nocobase/plugin-auth/src/locale/zh-CN.json`)
- 添加验证码相关的错误提示信息
- 提供用户友好的中文界面

## 功能特性

### 验证码功能
- **生成方式**: 4 位随机字母数字组合
- **显示格式**: SVG 图片，包含干扰线和干扰点
- **有效期**: 5 分钟
- **验证逻辑**: 不区分大小写，验证成功后自动清除
- **刷新机制**: 点击验证码图片可重新生成

### 安全增强
- **强制验证**: 登录时必须提供验证码
- **密码强度**: 默认管理员密码复杂度提升
- **会话管理**: 验证码与用户会话绑定
- **防重放**: 验证码使用后立即失效

## 使用方法

### 1. 启动服务
```bash
# 安装依赖
yarn install

# 启动开发服务器
yarn dev
```

### 2. 访问登录页面
- 打开浏览器访问 `http://localhost:3000/signin`
- 输入用户名/邮箱和密码
- 输入验证码（点击图片可刷新）
- 点击登录按钮

### 3. 默认管理员账号
- **用户名**: `nocobase` 或 `admin@nocobase.com`
- **密码**: `NocoBase@2024!`

## API 接口

### 生成验证码
```http
POST /api/auth:generateCaptcha
```

响应示例：
```json
{
  "svg": "<svg>...</svg>",
  "expires": 300
}
```

### 登录接口
```http
POST /api/auth:signIn
Content-Type: application/json
X-Authenticator: basic

{
  "account": "admin@nocobase.com",
  "password": "NocoBase@2024!",
  "captcha": "ABCD"
}
```

## 测试

运行测试脚本验证功能：
```bash
node test-captcha.js
```

## 注意事项

1. **环境变量**: 可以通过 `INIT_ROOT_PASSWORD` 环境变量自定义默认管理员密码
2. **Session 配置**: 确保服务器正确配置了 session 中间件
3. **缓存清理**: 验证码使用后会自动从 session 中清除
4. **错误处理**: 验证码错误或过期时会返回相应的错误信息

## 兼容性

- 保持与现有 API 的兼容性
- 不影响其他认证方式（如 SMS 认证）
- 向后兼容现有的用户数据

## 安全建议

1. 定期更换管理员密码
2. 启用 HTTPS 协议
3. 配置适当的 session 超时时间
4. 监控登录失败次数
5. 考虑启用双因素认证

## 故障排除

### 常见问题

1. **验证码不显示**
   - 检查网络连接
   - 确认 API 接口正常
   - 查看浏览器控制台错误

2. **验证码验证失败**
   - 确认验证码输入正确
   - 检查验证码是否过期
   - 验证 session 配置

3. **登录失败**
   - 确认用户名和密码正确
   - 检查验证码是否正确
   - 查看服务器日志

### 调试方法

1. 查看浏览器开发者工具的网络请求
2. 检查服务器日志输出
3. 使用测试脚本验证 API 接口
4. 确认 session 配置正确 
