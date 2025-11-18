[根目录](../../CLAUDE.md) > **openhands-ui**

# OpenHands UI 组件库模块

## 模块职责

openhands-ui 模块提供 OpenHands 平台的可复用 UI 组件库，基于 TypeScript 和 Vite 构建，为 frontend 模块和其他 UI 应用提供一致的设计系统和组件。

## 入口与使用

### 主要入口文件
- `index.ts`: 组件库主入口
- `tokens.css`: 设计令牌和 CSS 变量
- `package.json`: 项目配置和依赖

### 构建和开发
```bash
# 安装依赖
bun install

# 开发模式
bun run dev

# 构建
bun run build

# 测试
bun run test

# 类型检查
bun run typecheck
```

### 使用方式
```typescript
// 导入组件
import { Button, Input, Modal } from 'openhands-ui';

// 导入样式
import 'openhands-ui/tokens.css';
```

## 对外接口

### 核心组件

#### 基础组件
- **Button**: 多样化按钮组件，支持不同样式和状态
- **Input**: 输入框组件，支持验证和格式化
- **Modal**: 模态框组件，支持自定义内容和动画
- **Select**: 下拉选择组件，支持搜索和多选

#### 布局组件
- **Container**: 容器组件，支持响应式布局
- **Grid**: 网格布局组件
- **Flex**: 弹性布局组件
- **Card**: 卡片容器组件

#### 反馈组件
- **Toast**: 消息提示组件
- **Loading**: 加载状态组件
- **Progress**: 进度条组件
- **Alert**: 警告和提示组件

#### 数据展示
- **Table**: 表格组件，支持排序和分页
- **List**: 列表展示组件
- **Badge**: 标签和徽章组件
- **Avatar**: 头像组件

### 设计系统
- **设计令牌**: `tokens.css` 中的颜色、字体、间距等定义
- **主题系统**: 支持明暗主题切换
- **响应式设计**: 移动端友好的响应式布局
- **无障碍支持**: ARIA 标准和键盘导航

## 关键依赖与配置

### 核心框架
- **TypeScript**: 类型安全的 JavaScript
- **Vite**: 快速构建工具
- **Vitest**: 测试框架

### 开发工具
- **Bun**: 快速的 JavaScript 运行时和包管理器
- **TypeScript Compiler**: TypeScript 编译器
- **ESLint**: 代码质量检查
- **Prettier**: 代码格式化

### 构建配置
- **Vite Config**: `vite.config.ts` 构建配置
- **TypeScript Config**: `tsconfig.json` 类型配置
- **Vitest Config**: `vitest.config.ts` 测试配置

### 样式处理
- **PostCSS**: CSS 处理工具
- **CSS Variables**: 自定义 CSS 变量
- **CSS Modules**: 模块化 CSS

## 数据模型

### 组件 Props 定义
```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}
```

### 设计令牌
```css
:root {
  --color-primary: #0066cc;
  --color-secondary: #6c757d;
  --font-family-base: 'Inter', sans-serif;
  --font-size-base: 16px;
  --spacing-md: 1rem;
}
```

## 测试与质量

### 测试结构
```
openhands-ui/
├── src/
│   ├── components/     # 组件源码
│   ├── styles/        # 样式文件
│   └── __tests__/     # 测试文件
```

### 测试策略
- **单元测试**: 每个组件的独立测试
- **集成测试**: 组件间协作测试
- **视觉回归测试**: UI 一致性测试
- **无障碍测试**: 可访问性验证

### 代码质量
- **TypeScript**: 严格的类型检查
- **ESLint**: 代码规范检查
- **Prettier**: 统一代码格式
- **Husky**: Git 预提交钩子

### 性能优化
- **Tree Shaking**: 按需导入支持
- **Bundle 分析**: 打包体积优化
- **运行时性能**: 组件渲染性能优化

## 版本管理

### 语义化版本
- **Major**: 破坏性变更
- **Minor**: 新功能添加
- **Patch**: 问题修复

### 发布流程
- **自动化发布**: 通过 CI/CD 自动发布
- **变更日志**: 自动生成变更记录
- **标签管理**: Git 标签管理版本

### 分发方式
- **NPM**: 通过 NPM 包管理器分发
- **CDN**: 支持 CDN 直接引用
- **源码**: 支持源码直接使用

## 常见问题 (FAQ)

### Q: 如何自定义主题？
A: 修改 `tokens.css` 中的 CSS 变量，或覆盖设计令牌。

### Q: 如何添加新组件？
A: 在 `src/components/` 下创建组件文件，导出并在主入口中注册。

### Q: 如何处理样式冲突？
A: 使用 CSS Modules 或 CSS-in-JS 方案，确保样式隔离。

### Q: 如何优化打包体积？
A: 启用 Tree Shaking，按需导入组件，使用外部依赖。

## 相关文件清单

### 核心文件
- `package.json` - 项目配置和依赖
- `index.ts` - 组件库主入口
- `tokens.css` - 设计令牌和变量
- `index.css` - 全局样式

### 配置文件
- `tsconfig.json` - TypeScript 配置
- `vite.config.ts` - Vite 构建配置
- `vitest.config.ts` - Vitest 测试配置
- `vitest.shims.d.ts` - Vitest 类型定义

### 样式文件
- `tokens.css` - 设计令牌
- `index.css` - 主样式文件

### 工具文件
- `vite.config.ts` - Vite 配置
- `vitest.config.ts` - 测试配置

### 文档
- `README.md` - 组件库使用说明
- `PUBLISHING.md` - 发布指南

### 锁定文件
- `bun.lock` - Bun 依赖锁定

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 openhands-ui 模块文档
- 添加导航面包屑和模块结构说明
- 完善组件库功能和接口描述
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成维护，最后更新时间：2025-11-18 17:14:39*