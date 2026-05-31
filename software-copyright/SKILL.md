---
name: software-copyright
description: 生成中国软件著作权（软著）申请材料，包括申请信息汇总文档和源代码 PDF。当用户提到软著、软件著作权、版权登记、代码PDF、源代码提交材料时使用此 skill。也适用于用户说"帮我准备软著材料"、"生成源代码文档"、"申请软件著作权"等场景。
user_invocable: true
---

# 软件著作权申请材料生成器

从项目代码库自动生成软件著作权申请所需的全部材料。

## 参数

用户可通过自然语言指定以下参数，未指定时使用默认值：

- **version**: 申请版本号，默认 `V1.0`
- **software_name**: 软件全称，默认从项目配置中推断

## 工作流程

按以下 3 个阶段执行，每个阶段完成后向用户汇报进展。

### 阶段一：项目信息采集

使用 Explore 子agent 扫描项目，收集以下信息：

1. **项目名称和描述** — 从 `package.json`、`app.json`、`project.config.json`、`README.md` 等推断
2. **技术栈** — 编程语言、框架、依赖库、数据库
3. **代码行数** — 统计所有源代码文件（排除 `node_modules`、`dist`、`.d.ts` 等），分语言统计
4. **文件结构** — 主要目录和模块划分
5. **核心功能** — 从页面/路由/组件/API 推断功能模块
6. **版本信息** — 从配置文件或 git 历史获取

统计代码行数时，使用 bash 命令：
```bash
# 按语言统计（排除第三方代码）
find . -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.java" -o -name "*.go" \
  | grep -v node_modules | grep -v dist | grep -v .d.ts \
  | xargs wc -l 2>/dev/null | tail -1
```

### 阶段二：生成申请信息汇总文档

在 `软著申请材料/` 目录下生成 `软件著作权申请材料.md`，包含以下章节：

#### 一、软件基本信息（表格）
- 软件全称（格式："XX软件"或"XX系统"）
- 软件简称
- 版本号（使用用户指定版本）
- 软件分类（应用软件/系统软件/中间件等）
- 开发完成日期 → 标注 **[需填写]**
- 首次发表日期 → 标注 **[需填写]**
- 开发方式 → 标注 **[请确认]**
- 著作权人 → 标注 **[需填写]**

#### 二、软件开发和运行环境（表格）
根据项目实际情况填写：开发硬件环境、开发软件环境、开发语言、源程序量、运行硬件环境、运行软件环境、SDK、数据库、编程语言及版本。

#### 三、登记申请表关键字段
这些字段直接对应在线申请表单，用代码块包裹方便复制：

- **源程序量（行）** — 实际统计数字，不能为 0 或以 0 开头
- **开发目的（限50字）** — 根据项目功能撰写，控制字数
- **面向领域/行业（限50字）** — 根据项目领域撰写，控制字数
- **软件的主要功能（500～1300字）** — 详细描述所有核心功能模块，每个模块展开说明具体内容

#### 四、软件功能说明（约 800 字）
详细描述软件用途、核心功能模块（逐个展开）、技术架构。这部分用于软件说明书。

#### 五、源代码提交要求
说明前后各 30 页的代码文件选择建议。

#### 六、申请所需材料清单
必备材料和可选材料的 checklist。

#### 七、登记申请表填写参考
鉴别材料和功能技术特点的表格。

#### 八、申请流程
从注册到领证的步骤。

#### 九、待确认信息
列出需要用户补充的所有信息项。

### 阶段三：生成源代码 PDF

#### 前置检查

先检查 `reportlab` 是否已安装：
```bash
pip3 list 2>/dev/null | grep -i reportlab || pip3 install reportlab --quiet
```

#### 生成 PDF 的 Python 脚本

在 `软著申请材料/` 下创建 `generate_code_pdf.py`，脚本需要：

**配置参数：**
- `SOFTWARE_NAME` — 软件全称
- `VERSION` — 版本号
- `LINES_PER_PAGE = 50` — 每页行数
- `FRONT_PAGES = 30` — 前页数
- `BACK_PAGES = 30` — 后页数
- `SOURCE_FILES` — 按逻辑顺序排列的源文件路径列表

**源文件排列顺序（重要）：**
按以下逻辑顺序排列，确保代码具有可读性：
1. 应用入口文件
2. 通用基类/工具
3. 数据模型/类型定义
4. 状态管理
5. 业务服务/核心逻辑
6. 配置文件
7. 页面/路由/控制器
8. 组件
9. 工具函数
10. 后端/云函数/API

**排除以下文件：**
- `node_modules/`、`dist/`、`build/` 下的文件
- `.d.ts` 类型声明文件
- 纯配置文件（`tsconfig.json`、`.eslintrc` 等）
- 测试文件（`*.test.*`、`*.spec.*`）
- `package.json`、`package-lock.json`

**字体处理（macOS）：**
```python
# 中文字体
pdfmetrics.registerFont(TTFont('STHeiti', '/System/Library/Fonts/STHeiti Medium.ttc', subfontIndex=0))
# 等宽英文字体
pdfmetrics.registerFont(TTFont('Menlo', '/System/Library/Fonts/Menlo.ttc', subfontIndex=0))
```

如果在 Linux 上，回退到系统可用的中文字体：
```python
# Linux 回退方案
font_candidates = [
    '/usr/share/fonts/truetype/wqy/wqy-microhei.ttc',
    '/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc',
    '/usr/share/fonts/truetype/droid/DroidSansFallbackFull.ttf',
]
```

**PDF 排版规范：**

| 元素 | 规格 |
|------|------|
| 页面尺寸 | A4 |
| 页眉左侧 | `{软件全称}  {版本号}` — STHeiti 9pt, #333333 |
| 页眉右侧 | `源代码前30页` 或 `源代码后30页` |
| 页眉下划线 | #BBBBBB, 0.6pt |
| 页脚 | `- {页码} -` 居中, STHeiti 9pt, #888888 |
| 行号 | Menlo 7pt, #AAAAAA, 右对齐 |
| 代码文本 | Menlo 8pt（英文）或 STHeiti 8pt（含中文时）, #1C1C1C |
| 注释行 | `//` `/*` `*` 开头的行用 #5D6D7E |
| 文件分隔头 | 浅蓝背景条 #F0F4F8，文件路径 STHeiti 8pt #1A5276 |
| 左边距 | 18mm |
| 行号区域 | 12mm |
| 行高 | 4.2mm |
| 每行最大字符 | 85（中文算2） |

**中文宽度处理：**
使用 `unicodedata.east_asian_width()` 判断字符宽度，中文字符算 2 格，ASCII 算 1 格，按显示宽度截断。

**代码行字体选择逻辑：**
检测当前行是否包含 CJK 字符（`unicodedata.category(ch).startswith('Lo')`），包含则用 STHeiti，否则用 Menlo。

**输出文件：**
- `源代码_前30页.pdf` — 页码 1-30
- `源代码_后30页.pdf` — 页码 31-60
- `源代码_全60页.pdf` — 合并版（推荐提交），页码 1-60

#### 运行脚本
```bash
python3 软著申请材料/generate_code_pdf.py
```

运行后打开 PDF 供用户预览：
```bash
open 软著申请材料/源代码_全60页.pdf  # macOS
# xdg-open 软著申请材料/源代码_全60页.pdf  # Linux
```

### 阶段四：生成设计说明书 PDF

设计说明书是软著申请的鉴别材料之一，用于说明软件的功能、设计和技术架构。

#### 前置准备

1. **项目分析**：使用 Explore 子agent 深入分析项目，收集：
   - 完整页面/模块功能描述
   - 组件列表及用途
   - 核心业务逻辑和数据流
   - 配置数据（如本项目的呼吸法参数）
   - UI 界面特征和设计风格
   - 用户数据模型
   - 后端服务架构

2. **参考文档**：如果用户提供了参考文档（如 .docx），用 `python-docx` 提取其结构和格式风格作为参考。

3. **截图收集**：如果用户提供了软件截图，复制到 `软著申请材料/screenshots/` 目录并嵌入到对应章节。

#### 生成 PDF 的 Python 脚本

在 `软著申请材料/` 下创建 `generate_manual_pdf.py`，使用 `reportlab` 生成。

**文档结构（7 章）：**

| 章节 | 内容要点 |
|------|---------|
| 封面 | 软件名称 + 版本号 + "设计说明书" + 日期 |
| 目录 | 完整章节索引 |
| 一、项目背景 | 行业痛点、产品定位、解决方案概述（2-3 段） |
| 二、软件功能介绍 | 按模块逐一详述，每个模块用 H2 标题 + 正文 + 功能列表，**截图插入在对应模块末尾** |
| 三、软件特性介绍 | 技术亮点和用户体验特性列表 |
| 四、软件运行环境 | 客户端环境 + 服务端环境，分条列出 |
| 五、功能设计 | 核心系统的详细设计（含数据表格），如配置体系、用户系统、数据同步等 |
| 六、技术架构 | 技术栈表格 + 目录架构说明 + 后端架构说明 |
| 七、系统维护设计 | 更新机制、日志监控、用户反馈等 |

**PDF 排版规范：**

| 元素 | 规格 |
|------|------|
| 页面尺寸 | A4，上下左右边距 25mm |
| 封面标题 | 粗体 28pt，居中 |
| 封面版本号 | 18pt，居中，#333333 |
| 封面日期 | 14pt，居中，#555555 |
| 一级标题 (H1) | 粗体 18pt，上间距 24，下间距 12 |
| 二级标题 (H2) | 粗体 15pt，上间距 18，下间距 8 |
| 三级标题 (H3) | 粗体 13pt，上间距 12，下间距 6，左缩进 10 |
| 正文 | 11pt，行高 20，首行缩进 22，两端对齐 |
| 列表项 | 11pt，左缩进 20，以 `•` 开头 |
| 子列表项 | 11pt，左缩进 40，以 `-` 开头，#333333 |
| 图片说明 | 10pt，居中，#666666 |
| 表格表头 | 粗体 10pt，深蓝背景 #2C3E50，白色文字 |
| 表格单元格 | 10pt，居中，奇偶行交替背景 |

**字体处理（macOS 优先）：**
```python
# 普通字体 - 按优先级尝试
font_paths = {
    'PingFang': '/System/Library/Fonts/PingFang.ttc',
    'STSong': '/System/Library/Fonts/STHeiti Light.ttc',
    'Heiti': '/System/Library/Fonts/STHeiti Medium.ttc',
}
# 粗体字体
bold_paths = [
    ('/System/Library/Fonts/PingFang.ttc', 'PingFangBold', 1),
    ('/System/Library/Fonts/STHeiti Medium.ttc', 'HeitiBold', 0),
]
```

**截图嵌入规范：**
- 图片宽度默认 65mm，按原始宽高比缩放
- 最大高度限制 200mm，防止超出页面
- 居中对齐 (`hAlign = 'CENTER'`)
- 上方间距 4mm，下方紧跟图片说明文字（格式：`图N  标题 — 描述`）
- 使用 `ImageReader` 获取原始尺寸计算宽高比

```python
def add_screenshot(content_list, filename, caption, img_width=65*mm):
    from reportlab.lib.utils import ImageReader
    img_reader = ImageReader(img_path)
    iw, ih = img_reader.getSize()
    aspect = ih / iw
    img_height = img_width * aspect
    max_height = 200 * mm
    if img_height > max_height:
        img_height = max_height
        img_width = img_height / aspect
    img = Image(img_path, width=img_width, height=img_height)
    img.hAlign = 'CENTER'
```

**表格样式：**
```python
TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), HexColor('#2C3E50')),     # 表头深蓝
    ('TEXTCOLOR', (0, 0), (-1, 0), HexColor('#FFFFFF')),       # 表头白字
    ('GRID', (0, 0), (-1, -1), 0.5, HexColor('#CCCCCC')),     # 浅灰网格
    ('ROWBACKGROUNDS', (0, 1), (-1, -1), ['#FFFFFF', '#F5F5F5']), # 交替行背景
    ('TOPPADDING', (0, 0), (-1, -1), 6),
    ('BOTTOMPADDING', (0, 0), (-1, -1), 6),
])
```

**输出文件：**
- `软著申请材料/《{软件名称}》软件著作权登记设计说明书.pdf`

#### 运行脚本
```bash
python3 软著申请材料/generate_manual_pdf.py
```

### 最终交付

向用户展示 `软著申请材料/` 目录结构和文件列表，提醒用户：
1. 检查 PDF 中的代码可读性和中文显示
2. 检查设计说明书中的截图显示和章节内容准确性
3. 补充"待确认信息"中标记 **[需填写]** 的字段
4. `源代码_全60页.pdf` 可直接上传到申请系统
5. 设计说明书 PDF 可作为鉴别材料（文档）上传
