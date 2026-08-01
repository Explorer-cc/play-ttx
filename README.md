# ttx 工具详解笔记

`fontTools` 的 `ttx` 命令行工具详解文档（基于 fontTools 4.61.0 实际演示整理）。涵盖：字体文件结构（sfnt 容器、表）、Dump/Compile 操作、TTX (XML) 逐表解读（`head`/`name`/`cmap`/`glyf`/`CFF`/`GSUB`/`GPOS`/可变字体表 等）、字体特性与功能、常见工作流。

📖 完整文档见 [`ttx-note.md`](ttx-note.md)。

## ⚠️ 本仓库不提供字体文件

文档中的演示命令引用了大量字体文件，但**这些字体文件不包含在本仓库中**——它们体积庞大、且部分受版权或发行许可限制。请按下方清单自行获取，放到仓库根目录（或文档中对应路径）后即可复现演示。

## 演示字体清单与获取方式

| 文件 | 说明 | 获取方式 |
|------|------|----------|
| `times.ttf` | Times New Roman（西文 TrueType） | Windows 系统字体（`C:\Windows\Fonts\times.ttf`） |
| `msyh.ttc` | 微软雅黑（中文 TTC） | Windows 系统字体 |
| `cambria.ttc` | Cambria / Cambria Math | Windows Office 附带 |
| `Songti.ttc` | 宋体（macOS） | macOS 系统字体 |
| `SourceHanSansSC-Regular.otf` | 思源黑体 简体 | [adobe-fonts/source-han-sans](https://github.com/adobe-fonts/source-han-sans)（SIL OFL） |
| `SourceHanMono.ttc` | 思源等宽（Super TTC，70 子字体） | [adobe-fonts/source-han-mono](https://github.com/adobe-fonts/source-han-mono)（SIL OFL） |
| `NotoSerifCJKsc-VF.ttf` | Noto Serif CJK SC 可变字体 | [notofonts/noto-cjk](https://github.com/notofonts/noto-cjk)（SIL OFL） |
| `NotoSerifCJKsc-Regular.otf` | Noto Serif CJK SC | 同上 |
| `NotoSansCJKjp-Regular.otf` | Noto Sans CJK JP | 同上 |
| `Sarasa-SemiBold.ttc` | 更纱黑体 单字重 TTC（48 子字体） | [be5invis/sarasa-gothic](https://github.com/be5invis/sarasa-gothic)（SIL OFL） |
| `Sarasa-SuperTTC.ttc` | 更纱黑体 Super TTC（480 子字体） | 同上 |
| `texgyrepagella-math.otf` | TeX Gyre Pagella Math | [CTAN: tex-gyre-math-pagella](https://ctan.org/pkg/tex-gyre-math-pagella)（GUST 许可） |
| `times.woff2` | Times 的 WOFF2 | 用 `ttx --flavor woff2` 从 `times.ttf` 转换（见文档 5.1/8.5 节） |

> **版权提示**：系统字体（Times / 微软雅黑 / Cambria / 宋体）为各操作系统自带，受版权保护，仅用于本地学习演示，请勿再分发。开源字体（思源、Noto、Sarasa、TeX Gyre）按各自开源协议（多为 SIL OFL）使用。

## 复现演示

```bash
pip install fonttools brotli   # ttx + WOFF2 支持
# 按上表把所需字体文件放到仓库根目录
ttx -l times.ttf               # 即可复现文档中的命令
```

## 许可

文档本身（`ttx-note.md` 等）按本仓库 [LICENSE](LICENSE) 发布。字体文件见上表各自的许可，**本仓库不予再分发**。
