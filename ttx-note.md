# ttx 工具详解笔记

> 本文档基于 **fontTools 4.61.0**（`ttx` 命令行工具）实际演示整理。
> 所有终端输出均为**真实运行结果**逐字拷贝，演示字体见[附录 B](#附录-b演示字体清单)。

---

## 目录

- [第一部分　认识 ttx](#第一部分认识-ttx)
- [第二部分　ttx --help 手册全文 + 逐条中文详解](#第二部分ttx---help-手册全文--逐条中文详解)
- [第三部分　字体文件结构基础](#第三部分字体文件结构基础)
- [第四部分　Dump 操作实战](#第四部分dump-操作实战)
- [第五部分　Compile 操作实战](#第五部分compile-操作实战)
- [第六部分　TTX (XML) 格式逐表解读](#第六部分ttx-xml-格式逐表解读)
- [第七部分　字体特性与功能](#第七部分字体特性与功能)
- [第八部分　常见工作流](#第八部分常见工作流)
- [附录 A　命令行选项速查表](#附录-a命令行选项速查表)
- [附录 B　演示字体清单](#附录-b演示字体清单)
- [附录 C　常见表标签速查](#附录-c常见表标签速查)

---

# 第一部分　认识 ttx

## 1.1 ttx 是什么

`ttx` 是开源字体工具包 **fontTools** 提供的命令行工具，用于在 **二进制字体文件**（OpenType / TrueType）和 **TTX**（一种基于 XML 的文本格式）之间做**双向转换**：

```
                                反编译 (decompile / dump)             
字体文件(.ttf/.otf/.ttc/.woff)  ───────────────────────────► TTX 文件 (.ttx)
字体文件(.ttf/.otf/.ttc/.woff)  ◄─────────────────────────── TTX 文件 (.ttx)
                                      编译 (compile)
```

- **输入是字体**（`.ttf`/`.otf`/`.ttc`/`.otc`/`.woff`/`.woff2`）→ 反编译成 `.ttx`（XML 文本）。
- **输入是 `.ttx`** → 编译回二进制字体（TrueType 或 OpenType/CFF，取决于数据本身）。
- 特殊输入值 `-` 表示从**标准输入**读取。

**为什么有用？** 字体二进制是紧凑但不可读的结构化格式，*TTX 把每张"表"(table) 展开成人类可读、可用文本编辑器/diff/Git 管理的 XML*，使得**检视、修改、比对、版本控制**字体成为可能。修改 XML 后再用 ttx 编译回去，即可得到新字体。

## 1.2 安装与环境

`ttx` 随 `fontTools` 一起安装：

```bash
pip install fonttools          # 基础安装
pip install fonttools[woff]    # 额外装 zopfli，支持 --with-zopfli
pip install brotli             # 额外装 brotli，支持 WOFF2
```

查看版本：

```bash
$ ttx --version
4.61.0
```

本文档对应 `fontTools 4.61.0`。

## 1.3 两种工作模式

| 模式 | 触发条件 | 作用 |
|------|----------|------|
| **Dump（反编译）** | 输入是字体文件 | 字体 → TTX (XML) |
| **Compile（编译）** | 输入是 `.ttx` 文件 | TTX (XML) → 字体 |

> **安全设计**：ttx 生成的输出文件**永不覆盖同名已有文件**，而是自动追加序号（如 `times#1.ttx`）。需要覆盖时加 `-f`。

---

# 第二部分　ttx --help 手册全文 + 逐条中文详解

## 2.1 帮助原文（逐字拷贝）

> 注：`ttx --help` 并非有效选项（实际触发帮助的是 `-h`），ttx 会先打印帮助再报 `option --help not recognized` 退出。以下为完整输出：

```
usage: ttx [options] inputfile1 [... inputfileN]

TTX -- From OpenType To XML And Back

If an input file is a TrueType or OpenType font file, it will be
decompiled to a TTX file (an XML-based text format).
If an input file is a TTX file, it will be compiled to whatever
format the data is in, a TrueType or OpenType/CFF font file.
A special input value of - means read from the standard input.

Output files are created so they are unique: an existing file is
never overwritten.

General options
===============

-h Help            print this message.
--version          show version and exit.
-d <outputfolder>  Specify a directory where the output files are
                   to be created.
-o <outputfile>    Specify a file to write the output to. A special
                   value of - would use the standard output.
-f                 Overwrite existing output file(s), ie. don't append
                   numbers.
-v                 Verbose: more messages will be written to stdout
                   about what is being done.
-q                 Quiet: No messages will be written to stdout about
                   what is being done.
-a                 allow virtual glyphs ID's on compile or decompile.

Dump options
============

-l           List table info: instead of dumping to a TTX file, list
             some minimal info about each table.
-t <table>   Specify a table to dump. Multiple -t options
             are allowed. When no -t option is specified, all tables
             will be dumped.
-x <table>   Specify a table to exclude from the dump. Multiple -x
             options are allowed. -t and -x are mutually exclusive.
-s           Split tables: save the TTX data into separate TTX files per
             table and write one small TTX file that contains references
             to the individual table dumps. This file can be used as
             input to ttx, as long as the table files are in the
             same directory.
-g           Split glyf table: Save the glyf data into separate TTX files
             per glyph and write a small TTX for the glyf table which
             contains references to the individual TTGlyph elements.
             NOTE: specifying -g implies -s (no need for -s together
             with -g)
-i           Do NOT disassemble TT instructions: when this option is
             given, all TrueType programs (glyph programs, the font
             program and the pre-program) will be written to the TTX
             file as hex data instead of assembly. This saves some time
             and makes the TTX file smaller.
-z <format>  Specify a bitmap data export option for EBDT:
             {'raw', 'row', 'bitwise', 'extfile'} or for the CBDT:
             {'raw', 'extfile'} Each option does one of the following:

             -z raw
               export the bitmap data as a hex dump
             -z row
               export each row as hex data
             -z bitwise
               export each row as binary in an ASCII art style
             -z extfile
               export the data as external files with XML references

             If no export format is specified 'raw' format is used.
-e           Don't ignore decompilation errors, but show a full traceback
             and abort.
-y <number>  Select font number for TrueType Collection (.ttc/.otc),
             starting from 0.
--unicodedata <UnicodeData.txt>
             Use custom database file to write character names in the
             comments of the cmap TTX output.
--newline <value>
             Control how line endings are written in the XML file. It
             can be 'LF', 'CR', or 'CRLF'. If not specified, the
             default platform-specific line endings are used.

Compile options
===============

-m           Merge with TrueType-input-file: specify a TrueType or
             OpenType font file to be merged with the TTX file. This
             option is only valid when at most one TTX file is specified.
-b           Don't recalc glyph bounding boxes: use the values in the
             TTX file as-is.
--recalc-timestamp
             Set font 'modified' timestamp to current time.
             By default, the modification time of the TTX file will be
             used.
--no-recalc-timestamp
             Keep the original font 'modified' timestamp.
--flavor <type>
             Specify flavor of output font file. May be 'woff' or 'woff2'.
             Note that WOFF2 requires the Brotli Python extension,
             available at https://github.com/google/brotli
--with-zopfli
             Use Zopfli instead of Zlib to compress WOFF. The Python
             extension is available at https://pypi.python.org/pypi/zopfli
--optimize-font-speed
             Enable optimizations that prioritize speed over file size.
             This mainly affects how glyf table and gvar / VARC tables are
             compiled. The produced fonts will be larger, but rendering
             performance will be improved with HarfBuzz and other text
             layout engines.
```

## 2.2 通用选项（General options）逐条详解

| 选项 | 中文释义 | 说明 |
|------|----------|------|
| `-h` | 打印帮助信息 | 真正的帮助触发符；`--help` 反而会报错 |
| `--version` | 显示版本号并退出 | 例：`4.61.0` |
| `-d <目录>` | 指定输出文件**目录** | ⚠️ **该目录必须事先存在**，否则报错 `The -d option value must be an existing directory` |
| `-o <文件>` | 指定输出**文件名**；`-o -` 表示输出到**标准输出** | ⚠️ **`-o` 与 `-d` 同时出现时，`-o` 生效、`-d` 被忽略**，文件按 `-o` 的值（相对当前目录）写出 |
| `-f` | 覆盖已有输出文件（不追加序号） | 默认行为是永不覆盖、自动加 `#1`/`#2`… |
| `-v` | 详细模式：打印更多进度信息 | 如 `Reading 'head' table from disk` / `Decompiling ...` |
| `-q` | 静默模式：不打印任何进度信息 | 适合脚本化批量处理 |
| `-a` | 允许编译/反编译时使用**虚拟字形 ID** | 边缘场景，处理引用了不存在字形 ID 的字体时使用 |

## 2.3 Dump 选项（反编译时）逐条详解

| 选项 | 中文释义 | 说明 |
|------|----------|------|
| `-l` | **列表模式**：不导出 TTX，只列出每张表的标签/校验和/长度/偏移 | 快速体检字体的第一手段 |
| `-t <表>` | 只导出指定的表（可多次 `-t`） | 不带 `-t` 时导出全部表 |
| `-x <表>` | 排除指定的表（可多次 `-x`） | `-t` 与 `-x` **互斥**，不能同时用 |
| `-s` | **按表拆分**：每张表单独一个 `.ttx` 文件，另写一个引用它们的小主文件 | 便于逐表编辑、Git 提交 |
| `-g` | **按字形拆分 glyf**：每个字形单独一个文件 | `-g` 隐含 `-s`，无需再写 `-s` |
| `-i` | **不反汇编** TrueType 指令，改为十六进制输出 | 更快、文件更小；默认会把指令反汇编成可读汇编 |
| `-z <格式>` | 位图（EBDT/CBDT）导出格式：`raw`/`row`/`bitwise`/`extfile` | 见 [4.9 节](#49--z-位图导出格式) |
| `-e` | 不忽略反编译错误，打印完整 traceback 并中止 | 调试损坏字体时使用 |
| `-y <编号>` | 选择 TTC/OTC 字体集合中的字体编号（从 0 起） | 字体集合 `.ttc` 含多个字体，必须用 `-y` 指定 |
| `--unicodedata <文件>` | 用自定义 Unicode 数据库，在 cmap 输出注释里写字符名 | 自定义/扩展 Unicode 版本时使用 |
| `--newline <值>` | 控制 XML 行尾：`LF` / `CR` / `CRLF` | 不指定则用平台默认 |

## 2.4 Compile 选项（编译时）逐条详解

| 选项 | 中文释义 | 说明 |
|------|----------|------|
| `-m <字体>` | 把 TTX 文件**合并**进一个已有的字体文件 | 最多只能指定一个 TTX 文件；用于"只改几张表"的工作流 |
| `-b` | 不重新计算字形包围盒（bbox），直接用 TTX 里的值 | 默认会重算 |
| `--recalc-timestamp` | 把字体 `modified` 时间戳设为**当前时间** | 默认用 TTX 文件的修改时间 |
| `--no-recalc-timestamp` | 保留字体**原始** `modified` 时间戳 | |
| `--flavor <类型>` | 输出字体风味：`woff` 或 `woff2` | WOFF2 需额外安装 brotli |
| `--with-zopfli` | 用 Zopfli（而非 Zlib）压缩 WOFF | 压缩率更高但更慢；需装 zopfli |
| `--optimize-font-speed` | 优化渲染速度优先于体积 | 主要影响 glyf / gvar / VARC 的编译方式；产物更大但 HarfBuzz 等渲染更快 |

---

# 第三部分　字体文件结构基础

## 3.1 sfnt 容器与"表"(table)

OpenType/TrueType 字体本质是一个 **sfnt 容器**，里面装着一堆带 4 字符标签（tag）的**表**(table)。每张表负责字体的一个方面：

- `head`——字体头（版本、unitsPerEm、bbox、时间戳…）
- `name`——命名（字体名、版权、版本字符串…）
- `cmap`——字符到字形的映射
- `glyf` / `CFF `——字形轮廓数据
- `hhea` / `hmtx`——水平度量
- `GSUB` / `GPOS`——OpenType 布局（替换 / 定位）
- ……

`ttx -l` 是查看一个字体"装了哪些表"的最快方式。

## 3.2 ttx -l 列表实战（4 个字体对比）

### Times New Roman（`times.ttf`，西文 TrueType）

```
$ ttx -l times.ttf
Listing table info for "times.ttf":
    tag     checksum    length    offset
    ----  ----------  --------  --------
    DSIG  0xAC40E597      9644   1191976
    GDEF  0x240E2921       832   1032208
    GPOS  0x0340B599    126438   1033040
    GSUB  0x2267ED52     32366   1159480
    JSTF  0x6D2A6906        30   1191848
    LTSH  0xFAFD0F81      4733     19548
    OS/2  0x176F5876        96       536
    PCLT  0x59D381E3        54   1032152
    VDMX  0x4E236882      4500     24284
    cmap  0xE969B938      4510    142360
    cvt   0x1D6402D7      4228    153512
    fpgm  0x6D591B53      2649    146872
    gasp  0x00190021        16   1032136
    glyf  0x52EA912B    838298    176660
    hdmx  0x03D7905C    113576     28784
    head  0xF4B9D98E        54       412
    hhea  0x134D1A91        36       468
    hmtx  0x2005A771     18916       632
    kern  0xA677ACD1      5220   1014960
    loca  0x7B4F9A90     18920    157740
    maxp  0x1941135D        32       504
    meta  0x03A00561        96   1191880
    name  0x1A5574B8     11924   1020180
    post  0xFF240064        32   1032104
    prep  0xB3329291      3987    149524
```

四列分别是：**表标签**、**校验和**、**字节长度**、**在文件中的偏移**。可以看到这是一个内容丰富的 TrueType 字体：含 `glyf`(轮廓) + `loca`(轮廓索引)、`fpgm`/`prep`/`cvt`(hinting 程序)、`GSUB`/`GPOS`/`GDEF`/`JSTF`(布局)、`hdmx`/`VDMX`/`LTSH`(各种度量优化)。

### 微软雅黑（`msyh.ttc`，中文 TTC 集合）

`.ttc` 是字体集合，必须用 `-y` 指定字体编号：

```
$ ttx -l msyh.ttc
ERROR: specify a font number between 0 and 1 (inclusive)
```

该集合含 **2 个字体**（编号 0、1）。查看编号 0：

```
$ ttx -l -y 0 msyh.ttc
    ----  ----------  --------  --------
    GDEF  0xE09B5667        42       856
    GPOS  0xEAA2F7BC      1506       900
    GSUB  0x3161C84A      2436      2408
    LTSH  0xCCDD6463     30213      4844
    MERG  0x00160001        12     35060
    OS/2  0x85B823AD        96     35072
    VDMX  0x73EC7B66      1504     35168
    cmap  0x87CA4F2F    103666     36672
    cvt   0x4C086ADE      1014    140340
    fpgm  0x2C98B72C      2566    141356
    gasp  0x001D0023        16    143924
    glyf  0x6807FEF9  18617410    143940
    hdmx  0xD4172CA5    513612  18761352
    head  0x1A075ABF        54  19274964
    hhea  0x115E7599        36  19275020
    hmtx  0x8074F0B4    120164  19275056
    kern  0xCDB4CE62      3516  19395220
    loca  0x52561FE6    120840  19398736
    maxp  0x7F510D70        32  19519576
    meta  0x689BBA3B        48  19519608
    name  0xE8CA67D9      3150  19519656
    post  0xFF517639        32  19522808
    prep  0x44BD05CF       754  19522840
    vhea  0x0ED005F4        36  19523596
    vmtx  0xD74A0E0F     60420  19523632
```

注意几个 CJK 字体特征：`glyf` 高达 **18.6 MB**（几万个汉字轮廓）、`cmap` **103 KB**（超大字符映射）、含 **`vhea`/`vmtx`**（纵向排版度量）、`MERG`（字形合并提示）。

### Cambria（`cambria.ttc`，含数学字体）

编号 1 是 Cambria Math，独有 `MATH` 表：

```
$ ttx -l -y 1 cambria.ttc
    ...
    MATH  0x4F508D8B     25722   1726288
    ...
    EBDT  0x0E304C68     22252       744
    EBLC  0xF2D855C0      5600     22996
    ...
```

`MATH` 表（25 KB）是 OpenType 数学排版的核心；`EBDT`/`EBLC` 是嵌入式点阵位图（小字号下的预渲染字形）。

### 思源黑体（`SourceHanSansSC-Regular.otf`，CFF 轮廓）

```
$ ttx -l SourceHanSansSC-Regular.otf
    ----  ----------  --------  --------
    BASE  0xEDFAF516       240  15791504
    CFF   0xC9F11CC0  15551854    239648
    DSIG  0x00000001         8  15791744
    GDEF  0x020E0201        28  15791752
    GPOS  0xF2C8233F     46568  15791780
    GSUB  0x6C9FFF0C    167048  15838348
    OS/2  0x92EC11E3        96       384
    VORG  0xD86AF415       920  16005396
    cmap  0x57B9899D    236797      2816
    head  0x2F8E7185        54       284
    hhea  0x0C12084D        36       340
    hmtx  0xAB3609B4    262068  16006316
    maxp  0xFFFF5000         6       376
    name  0xFF97DD0C      2336       480
    post  0xFF860074        32    239616
    vhea  0x0CB215B2        36  16268384
    vmtx  0xF9498662    261412  16268420
```

这是 **CID-keyed CFF** 字体（PostScript 轮廓）：用 `CFF ` 表（15.5 MB）而非 `glyf`；`maxp` 只有 **6 字节**（CFF 字体的 maxp 极简）；含 `VORG`（纵向原点）、`BASE`（基线表）、`vhea`/`vmtx`（CJK 纵向）、超大 `GSUB`(167 KB)。

## 3.3 TTC/OTC 字体集合：枚举各 index

### 3.3.1 什么是 TTC/OTC 字体集合

**TTC**（TrueType Collection）和 **OTC**（OpenType Collection）是"字体集合"容器：一个文件里打包**多个字体**。两者容器格式相同（文件头 4 字节都是 `ttcf`），区别只在内部轮廓——`.ttc` 装 TrueType（`glyf`）字体，`.otc` 装 CFF（`CFF `）字体。

集合最大的价值是**表去重**：多个字体若共用同一张表（如同一套 `glyf` 轮廓），文件里只存一份、靠偏移引用。例如 `msyh.ttc`（微软雅黑 + 雅黑 UI）的 25 张表里有 **19 张完全共享**；`cambria.ttc`（Cambria + Cambria Math）共享 13 张、含 `glyf`/`loca`/`hmtx`，只有 Math 字体独占一张 `MATH`。

> **对 ttx 而言**：TTC/OTC 必须用 `-y <编号>` 选定其中一个字体再操作（见第 2.3 节的 `-y`）。`-y` 从 **0** 起。

### 3.3.2 发现集合里有几个字体（index 数量）

`ttx -l` 不带 `-y` 直接打 TTC 会报错——但报错信息**透露了 index 范围**，这是最快的"数数"方法：

```
$ ttx -l Songti.ttc
ERROR: specify a font number between 0 and 7 (inclusive)
```

→ 0 到 7，共 **8 个字体**。或者用 fontTools 一行直接拿到（无需猜测）：

```
$ python -c "from fontTools.ttLib import TTCollection; print(len(TTCollection('Songti.ttc').fonts))"
8
```

### 3.3.3 逐个 index 列出表信息（-y 迭代）

知道数量后，用 `-y N` 逐个 dump 列表，即可对比每个字体装了哪些表：

```
$ ttx -l -y 0 Songti.ttc
Listing table info for "Songti.ttc":
    tag     checksum    length    offset
    ----  ----------  --------  --------
    OS/2  0x652F6BDA        96      2628
    ...

$ ttx -l -y 4 Songti.ttc
Listing table info for "Songti.ttc":
    tag     checksum    length    offset
    ----  ----------  --------  --------
    OS/2  0xA1E90A73        96  45395268
    ...
```

注意 index 0 的 `OS/2` 偏移 2628、index 4 的偏移 45395268——**两者完全独立**，说明 Songti.ttc 各字体的表**没有共享**（这与 msyh/cambria 不同；是否共享是各厂商的打包策略）。

### 3.3.4 提取每个字体的具体信息：name 表 + OS/2

`-y` 只能列出表清单。要拿到「序号 / 字重 / 名称 / 版权」这类**字体元数据**，得读 **`name` 表**（命名表）和 **`OS/2` 表**。字段对应关系：

| 目标字段 | 来源 |
|----------|------|
| 序号 | TTC 内的 index（即 `-y` 值，从 0 起） |
| 字重（数值 300/400/700/900） | `OS/2.usWeightClass` |
| 英文名称 / 英文字重 | `name`：nameID=**1**（Family）/ nameID=**2**（Subfamily），语言=英文 |
| 简体名称 / 简体字重 | 同上，语言=简体中文 |
| 繁体名称 / 繁体字重 | 同上，语言=繁体中文 |
| 版权年份 | `name`：nameID=**0**（Copyright），用正则提取年份 |

`name` 表里每条记录由三元组 **(platformID, encodingID, langID)** 定位语言，常见值：

| 语言 | Windows 平台 (platformID=3) | Mac 平台 (platformID=1) |
|------|------------------------------|--------------------------|
| 英文 | (3, 1, 0x409) | (1, 0, 0) |
| 简体中文 | (3, 1, 0x804) | (1, 25, 33) |
| 繁体中文 | (3, 1, 0x404) | (1, 2, 19) |

> ⚠️ **真实字体的 name 记录并不统一**：新字体一般只写 Windows 平台，**老字体（如 STSong）常只有 Mac 平台**记录。因此查询时必须 **Windows 优先、Mac 兜底**，否则会取到空值（见下文 index 4）。

### 3.3.5 实战：Songti.ttc 完整枚举

下面的脚本对 `Songti.ttc` 的 8 个字体逐一提取上述字段（繁体缺失时回退到简体串，模拟系统字体选择器的行为）：

```python
# -*- coding: utf-8 -*-
from fontTools.ttLib import TTCollection
import re

c = TTCollection('Songti.ttc')
CAND = {                       # (platformID, encodingID, langID)，按优先级排列
    'en'  : [(3,1,0x409), (1,0,0)],
    'zhs' : [(3,1,0x804), (1,25,33)],
    'zht' : [(3,1,0x404), (1,2,19), (1,3,21)],
}
def rec(f, nid, loc):
    for pe in CAND[loc]:
        r = f['name'].getName(nid, *pe)
        if r:
            try: return r.toUnicode()
            except: continue
    return ''
def nm(f, nid, loc):
    v = rec(f, nid, loc)
    if not v and loc == 'zht':          # 繁体缺失 -> 回退简体
        v = rec(f, nid, 'zhs')
    return v
def years(s):
    return ','.join(sorted(set(re.findall(r'(?:19|20)\d{2}(?:-\d{2,4})?', s or '')), key=lambda x:x[:4]))

print('序号 字重 英文名称 英文字重 简体名称 简体字重 繁体名称 繁体字重 版权年份')
for i, f in enumerate(c.fonts):
    w = f['OS/2'].usWeightClass
    print(i, w, nm(f,1,'en'), nm(f,2,'en'),
          nm(f,1,'zhs'), nm(f,2,'zhs'), nm(f,1,'zht'), nm(f,2,'zht'),
          years(nm(f,0,'en')), sep=' | ')
```

运行结果（与参考表 [`YDX-Songti.pdf`](YDX-Songti.pdf) 完全吻合）：

```
序号 字重 英文名称 英文字重 简体名称 简体字重 繁体名称 繁体字重 版权年份
0 | 900 | Songti SC | Black | 宋体-简 | 黑体 | 宋體-簡 | 黑體 | 1991-1998,2012
1 | 700 | Songti SC | Bold | 宋体-简 | 粗体 | 宋體-簡 | 粗体 | 2000-2005,2012
2 | 700 | Songti TC | Bold | 宋体-繁 | 粗体 | 宋體-繁 | 粗体 | 2010-2012,2013
3 | 300 | Songti SC | Light | 宋体-简 | 细体 | 宋體-簡 | 細體 | 1991-2001
4 | 300 | STSong | Regular | 华文宋体 | 常规体 | 华文宋体 | 標準體 | 2002
5 | 300 | Songti TC | Light | 宋体-繁 | 细体 | 宋體-繁 | 細體 | 2010-2012,2013
6 | 400 | Songti SC | Regular | 宋体-简 | 常规体 | 宋體-簡 | 標準體 | 2010-2012
7 | 400 | Songti TC | Regular | 宋体-繁 | 常规体 | 宋體-繁 | 標準體 | 2010-2012,2013
```

**几点解读**：

- **SC / TC**：Songti **SC** = Simplified Chinese（简体），Songti **TC** = Traditional Chinese（繁体）；同一字重通常成对出现（如 index 1/2 都是 Bold 700）。
- **index 4 是异类**：`STSong`（华文宋体）是更老的字体，**版权年份只有 2002**（其余多为 2010 年后），且 name 记录**只存在于 Mac 平台**——这正是上文「Mac 兜底」的来源；它也没有专门的繁体记录，故繁体列回退显示简体串「华文宋体」。
- **测试文本列**：参考表里的「然而永远！分的喆冇鿬？」是**固定的样张字符串**（`喆`/`冇`/`鿬` 是生僻字，用来抽测字形覆盖），**并非字体元数据**，需自行定义与渲染；原参考图用思源黑体补齐缺字。
- **字重数值**：300=Light、400=Regular、700=Bold、900=Black，来自 `OS/2.usWeightClass`（OpenType 标准 weight 值）。

> 💡 **纯 ttx CLI 也能看 name 表**：`ttx -y 4 -t name -o - Songti.ttc` 会把 index 4 的 name 表反编译成 XML 打到标准输出，可直接看到每条 `<namerecord nameID="..." platformID="..." encodingID="..." langID="...">` 的对应关系，便于人工核对。

## 3.4 TrueType vs OpenType/CFF：两种轮廓格式

这是理解字体的**核心区分点**，也直接决定 ttx 输出的差异：

| 维度 | TrueType（`.ttf`） | OpenType/CFF（`.otf`） |
|------|--------------------|------------------------|
| 轮廓表 | `glyf` + `loca` | `CFF ` |
| 曲线类型 | 二次贝塞尔（控制点 `on`/`off`） | 三次贝塞尔 |
| hinting | TrueType 指令（`fpgm`/`prep`/`cvt`） | CFF 提示（Private dict 里的 `BlueValues` 等） |
| `maxp` 表 | 32 字节，含 15 个限制字段 | 仅 6 字节（版本 + numGlyphs） |
| 后缀 | `.ttf` | `.otf`（但 `.otf` 也能装 glyf） |

> 注意：**文件后缀不绝对等于轮廓类型**。`.otf` 既可以装 CFF 也可以装 glyf，判定依据是表里有没有 `CFF ` 表还是 `glyf` 表。

---

# 第四部分　Dump 操作实战

## 4.1 反编译：字体 → TTX

```bash
$ ttx -v -d ttx_out times.ttf
Dumping "times.ttf" to "ttx_out\times.ttx"...
Dumping 'GlyphOrder' table...
Reading 'post' table from disk
Decompiling 'post' table
Reading 'maxp' table from disk
Decompiling 'maxp' table
...（略，逐表 Reading / Decompiling / Dumping）...
Reading 'DSIG' table from disk
Decompiling 'DSIG' table
Dumping 'DSIG' table...
Done dumping TTX in 6.727 seconds
```

产物 `times.ttx` 约 **22.5 MB**（原始字体 1.2 MB，XML 化后膨胀约 18 倍）。`-v` 让每个表的"读盘→反编译→写出"步骤都可见；不带 `-v` 则只打印一行汇总。

## 4.2 编译：TTX → 字体（无损往返）

```bash
$ ttx -d ttx_out ttx_out/times.ttx
Compiling "ttx_out/times.ttx" to "ttx_out\times.ttf"...
Parsing 'GlyphOrder' table...
Parsing 'head' table...
...（逐表 Parsing）...
Parsing 'DSIG' table...
```

体积对比：

```
原 times.ttf        : 1201620 字节
重编译 times.ttf    : 1196136 字节
```

> **说明**：往返后体积略有差异（小约 0.5%）是正常的——ttx 在编译时会**重新计算**部分字段（如校验和、某些 bbox），二进制内部排列也可能与原始工具不同，但**字形数据与字体功能等价**。这就是"无损往返"的含义：视觉与功能一致，而非逐字节相同。

## 4.3 输出控制 `-d` / `-o` / `-f`

### `-d` 指定目录（**必须事先存在**）

```bash
$ ttx -s -q -d ttx_out/split_t times.ttf
ERROR: The -d option value must be an existing directory
```

→ 需先 `mkdir -p ttx_out/split_t` 再运行。

### `-o -` 输出到标准输出

```bash
$ ttx -t head -o - times.ttf
Dumping "times.ttf" to "<stdout>"...
<?xml version="1.0" encoding="UTF-8"?>
<ttFont sfntVersion="\x00\x01\x00\x00" ttLibVersion="4.61">
  <head>
    ...
  </head>
</ttFont>
```

适合管道处理（如 `ttx -t head -o - font.ttf | grep modified`）。

### `-f` 覆盖 vs 默认唯一命名

默认永不覆盖，自动追加 `#N`：

```
第1次 ttx times.ttf -> ['times.ttx']
第2次 ttx times.ttf (默认不覆盖) -> ['times#1.ttx', 'times.ttx']
第3次 ttx -f times.ttf (覆盖) -> ['times#1.ttx', 'times.ttx']   # 直接覆盖 times.ttx，不产生 #2
```

> ⚠️ **重要陷阱**：当 `-o` 和 `-d` 同时出现时，**`-o` 生效、`-d` 被忽略**，文件按 `-o` 的值（相对当前工作目录）写出，而非落到 `-d` 目录里。要写到指定目录，应写 `-o 目录/文件.ttx`。

## 4.4 详细与静默 `-v` / `-q`

同样只导出 `head` 表：

```
--- verbose (-v) ---
Dumping "times.ttf" to "<stdout>"...
Reading 'head' table from disk
Decompiling 'head' table
Dumping 'head' table...
Done dumping TTX in 0.006 seconds

--- quiet (-q) ---
（无任何进度输出，只有 XML 数据本身）
```

## 4.5 选择性导出 `-t` / `-x`

### `-t` 只导出指定表（可多个）

```bash
$ ttx -q -t head -t name -t maxp -d ttx_out times.ttf
```

| 导出方式 | 文件大小 |
|----------|----------|
| 完整 `times.ttx`（全部表） | **22 549 613 字节**（22.5 MB） |
| `-t head,name,maxp`（仅 3 表） | **14 515 字节**（14 KB） |

### `-x` 排除指定表（可多个）

```bash
$ ttx -q -x glyf -x hdmx -x GPOS -d ttx_out times.ttf
```

排除 3 个大表后 → **2 383 480 字节**（2.38 MB）。`-t` 与 `-x` **互斥**，不能同一条命令里混用。

> 实用技巧：只想看字体的元数据而不关心庞大轮廓时，`-x glyf` 能把 22 MB 砍到 2 MB，极大便于阅读和 diff。

## 4.6 拆分 `-s` / `-g`

### `-s` 按表拆分

```bash
$ ttx -q -s -d ttx_out/split_t times.ttf
```

生成 **27 个文件**（每个表一个 `.ttx`），另有一个主文件用 `src=` 属性引用它们：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ttFont sfntVersion="\x00\x01\x00\x00" ttLibVersion="4.61">
  <GlyphOrder src="times.GlyphOrder.ttx"/>
  <head src="times._h_e_a_d.ttx"/>
  <hhea src="times._h_h_e_a.ttx"/>
  <maxp src="times._m_a_x_p.ttx"/>
  <OS_2 src="times.O_S_2f_2.ttx"/>
  <hmtx src="times._h_m_t_x.ttx"/>
  ...
  <glyf src="times._g_l_y_f.ttx"/>
  ...
</ttFont>
```

> 注意表标签里的空格用下划线转义（如 `cvt ` → `_c_v_t`）。这个主文件可以直接喂回 ttx 编译，只要各分表文件在同目录。

### `-g` 按字形拆分 glyf

```bash
$ ttx -q -g -d ttx_out/split_g times.ttf
```

生成 **4750 个文件**，其中 **4724 个字形文件**，命名形如：

```
times._g_l_y_f..notdef.ttx
times._g_l_y_f.A_.ttx
times._g_l_y_f.A_E_.ttx
times._g_l_y_f.A_E_acute.ttx
...
```

`-g` 隐含 `-s`（无需再加 `-s`）。适合在 Git 里单独追踪每个字形的修改，或精确定位某字形问题。

## 4.7 TTC 字体集合 `-y`

`.ttc`/`.otc` 含多个字体，必须用 `-y` 选编号（从 0 起）：

```bash
$ ttx -l cambria.ttc
ERROR: specify a font number between 0 and 1 (inclusive)

$ ttx -y 1 -t MATH -o - cambria.ttc     # 导出编号1(数学字体)的 MATH 表
```

## 4.8 `-i` 不反汇编 TrueType 指令

TrueType 的 hinting 是一套栈式字节码指令。默认 ttx 把它**反汇编成可读汇编**；加 `-i` 则保留为**十六进制**，更快更小。

默认（反汇编）：

```xml
<fpgm>
  <assembly>
    NPUSHB[ ]	/* 87 values pushed */
    133 116 115 114 113 112 ...（略）
    FDEF[ ]	/* FunctionDefinition */
      RCVT[ ]	/* ReadCVT */
      SWAP[ ]	/* SwapTopStack */
      GC[0]	/* GetCoordOnPVector */
      ADD[ ]	/* Add */
      DUP[ ]	/* DuplicateTopStack */
      ...
```

加 `-i`（十六进制）：

```xml
<fpgm>
  <bytecode>
    40578574 73727170 6f6e6d6c 6b6a6968
    67666562 5d55544f 4e403f3e 3d3c3b3a
    39383736 35343332 31302f2e 2d2c2b2a
    ...
```

> 区别：默认用 `<assembly>` 元素（含助记符 + 注释，可读但占空间）；`-i` 用 `<bytecode>` 元素（紧凑 hex）。两者编译回字体后功能等价。

## 4.9 `-z` 位图导出格式

针对 `EBDT`/`CBDT`（嵌入式点阵位图）表，`-z` 控制位图数据的 XML 表达。以 Cambria 字形 `A`（8×7 点阵）为例：

**`-z raw`**（整块十六进制）：

```xml
<rawimagedata>
  10102828 287c44ee  
</rawimagedata>
```

**`-z row`**（逐行十六进制，每行一个字节）：

```xml
<rowimagedata bitDepth="1" height="8" width="7">
  <row value="10"/>
  <row value="10"/>
  <row value="28"/>
  <row value="28"/>
  <row value="28"/>
  <row value="7c"/>
  <row value="44"/>
  <row value="ee"/>
</rowimagedata>
```

**`-z bitwise`**（ASCII 点阵艺术，`@`=黑 `.`=白）：

```xml
<bitwiseimagedata bitDepth="1" height="8" width="7">
  <row value="...@..."/>
  <row value="...@..."/>
  <row value="..@.@.."/>
  <row value="..@.@.."/>
  <row value="..@.@.."/>
  <row value=".@@@@@."/>
  <row value=".@...@."/>
  <row value="@@@.@@@"/>
</bitwiseimagedata>
```

> `bitwise` 格式下能**肉眼直接看出字母 A 的形状**，是检视点阵字形最直观的方式。另有 `-z extfile` 把位图导出为外部文件、主 XML 里放引用。CBDT（彩色位图）只支持 `raw` 和 `extfile`。默认格式是 `raw`。

## 4.10 `--newline` 行尾控制

控制写出的 XML 文件用哪种换行符：

```bash
$ ttx -q --newline LF    -t head -o newline_lf.ttx   times.ttf
$ ttx -q --newline CRLF  -t head -o newline_crlf.ttx times.ttf
```

校验（统计字节数据）：

```
LF   模式文件: CRLF = 0 次, 纯 LF = 25 次
CRLF 模式文件: 纯 LF(无 CR) = 0 次
```

> 跨平台协作（尤其在 Windows 上生成、Linux 上用 Git 管理）时，统一用 `--newline LF` 可避免无意义的换行符 diff。

---

# 第五部分　Compile 操作实战

## 5.1 `--flavor woff2` 输出 WOFF2

把 TTX 编译成 WOFF2（需安装 brotli）：

```bash
$ ttx --flavor woff2 -o ttx_out/times.woff2 ttx_out/times.ttx
```

体积对比：

```
原 times.ttf   : 1201620 字节
WOFF2          :   506396 字节
压缩率         : 42.1%
```

WOFF2 用 Brotli 压缩，通常能把字体压到原大小的 30%–50%，是 Web 字体的主流格式。`--flavor woff` 则生成 WOFF（Zlib 压缩，可配 `--with-zopfli` 提高压缩率）。

## 5.2 `--recalc-timestamp` 时间戳控制

字体的 `modified` 时间戳默认用 TTX 文件的修改时间。两种显式控制：

```
原始 times.ttf head.modified:
    <modified value="Fri Jun 17 17:02:27 2022"/>

编译 --no-recalc-timestamp (保留原值):
    <modified value="Fri Jun 17 17:02:27 2022"/>

编译 --recalc-timestamp (设为当前时间):
    <modified value="Thu Jul 23 05:58:34 2026"/>
```

> 发布字体时通常希望时间戳反映本次构建时间 → 用 `--recalc-timestamp`。

## 5.3 `-m` 合并（只改几张表的工作流）

`-m` 把一个 TTX 文件**合并进**一个已有字体：以原字体为底，TTX 里出现的表覆盖原表，其余表保留。例：`times#1.ttx` 只含 `head`+`name`+`maxp`，把它合并进原字体：

```bash
$ ttx -m times.ttf -o ttx_out/merged.ttf ttx_out/times#1.ttx
```

```
合并产物 merged.ttf : 1201688 字节（基本=原字体，仅 3 张表被 TTX 覆盖）
原 times.ttf        : 1201620 字节
```

> 这是"**只改某张表再回编**"的标准流程：导出单表 → 改 XML → 用 `-m` 合并回原字体，无需处理全部表。限制：最多只能指定一个 TTX 文件。

## 5.4 其余编译选项

| 选项 | 用途 |
|------|------|
| `-b` | 不重算字形包围盒，用 TTX 里的原值（默认会重算） |
| `--with-zopfli` | 生成 WOFF 时用 Zopfli 压缩（更小更慢，需装 zopfli） |
| `--optimize-font-speed` | 编译时优化渲染速度优先于体积（产物更大，HarfBuzz 等渲染更快），主要影响 `glyf`/`gvar`/`VARC` |

---

# 第六部分　TTX (XML) 格式逐表解读

每个 TTX 文件的根元素是 `<ttFont>`，`sfntVersion` 标识类型：`\x00\x01\x00\x00` = TrueType，`OTTO` = OpenType/CFF。

## 6.1 `head` 字体头

```xml
<head>
  <!-- Most of this table will be recalculated by the compiler -->
  <tableVersion value="1.0"/>
  <fontRevision value="7.05"/>              <!-- 字体修订版本号 -->
  <checkSumAdjustment value="0x328bad8a"/>  <!-- 校验和调整（编译时重算）-->
  <magicNumber value="0x5f0f3cf5"/>         <!-- 魔数，恒为 0x5F0F3CF5 -->
  <flags value="00001000 00011001"/>        <!-- 全局标志位 -->
  <unitsPerEm value="2048"/>                <!-- em 单位（坐标系的分辨率）-->
  <created value="Mon Aug  6 13:14:42 1990"/>
  <modified value="Fri Jun 17 17:02:27 2022"/>
  <xMin value="-1164"/>  <yMin value="-628"/>
  <xMax value="4190"/>   <yMax value="2129"/>  <!-- 所有字形的包围盒 -->
  <macStyle value="00000000 00000000"/>     <!-- 粗体/斜体等风格位 -->
  <lowestRecPPEM value="9"/>                <!-- 最低推荐像素大小 -->
  <fontDirectionHint value="1"/>
  <indexToLocFormat value="1"/>             <!-- loca 表偏移格式(0=short,1=long) -->
  <glyphDataFormat value="0"/>
</head>
```

`unitsPerEm` 是关键：所有字形坐标都以它为分母，Times New Roman 用 2048，思源黑体用 1000。

## 6.2 `name` 命名表

每个 `namerecord` 用 `nameID` + `platformID`/`platEncID`/`langID` 三元组定位一条字符串：

```xml
<name>
  <namerecord nameID="0" platformID="0" platEncID="3" langID="0x0">
    © 2020 The Monotype Corporation. All Rights Reserved. ...
  </namerecord>
  <namerecord nameID="1" platformID="0" platEncID="3" langID="0x0">
    Times New Roman              <!-- nameID=1 字体家族名 -->
  </namerecord>
  <namerecord nameID="2" platformID="0" platEncID="3" langID="0x0">
    Regular                     <!-- nameID=2 子家族名(样式) -->
  </namerecord>
  <namerecord nameID="4" platformID="0" platEncID="3" langID="0x0">
    Times New Roman             <!-- nameID=4 完整字体名 -->
  </namerecord>
  <namerecord nameID="5" platformID="0" platEncID="3" langID="0x0">
    Version 7.05                <!-- nameID=5 版本字符串 -->
  </namerecord>
  <namerecord nameID="6" platformID="0" platEncID="3" langID="0x0">
    TimesNewRomanPSMT           <!-- nameID=6 PostScript 名 -->
  </namerecord>
  ...
```

常用 `nameID`：0=版权，1=家族名，2=子家族名，3=唯一ID，4=完整名，5=版本，6=PostScript名，7=商标，8=厂商，9=设计师，13=许可证。

## 6.3 `hhea` / `hmtx` 水平度量

`hhea` 给出整体水平排版参数：

```xml
<hhea>
  <tableVersion value="0x00010000"/>
  <ascent value="1825"/>       <descent value="-443"/>  <!-- 上升/下降 -->
  <lineGap value="87"/>
  <advanceWidthMax value="4096"/>   <!-- 最大前进宽度 -->
  <numberOfHMetrics value="4729"/>  <!-- 字形数 -->
  ...
</hhea>
```

`hmtx` 给每个字形的**前进宽度**(width)和**左侧间距**(lsb)：

```xml
<hmtx>
  <mtx name=".notdef" width="1593" lsb="284"/>
  <mtx name="A" width="1479" lsb="16"/>
  <mtx name="AE" width="1821" lsb="-24"/>
  ...
</hmtx>
```

## 6.4 `maxp`：TrueType vs CFF 对比

这是两种轮廓格式差异最直观的体现：

**TrueType（times.ttf，32 字节，15 个限制字段）**：

```xml
<maxp>
  <tableVersion value="0x10000"/>
  <numGlyphs value="4729"/>
  <maxPoints value="693"/>           <!-- 单字形最多点数 -->
  <maxContours value="60"/>          <!-- 最多轮廓数 -->
  <maxCompositePoints value="209"/>
  <maxCompositeContours value="7"/>
  <maxZones value="2"/>              <!-- hinting 区域数 -->
  <maxTwilightPoints value="16"/>
  <maxStorage value="64"/>
  <maxFunctionDefs value="134"/>     <!-- 最多函数定义(TrueType指令) -->
  <maxStackElements value="1513"/>
  <maxSizeOfInstructions value="3987"/>
  ...
</maxp>
```

**CFF（思源黑体，仅 6 字节）**：

```xml
<maxp>
  <tableVersion value="0x5000"/>
  <numGlyphs value="65535"/>
</maxp>
```

> CFF 字体不需要 TrueType 那套指令内存限制，所以 `maxp` 退化到只剩版本和字形数。`tableVersion` 也不同：`0x10000`=TrueType，`0x5000`=CFF。

## 6.5 `OS/2`

```xml
<OS_2>
  <version value="3"/>
  <usWeightClass value="400"/>      <!-- 字重: 100-900, 400=Regular -->
  <usWidthClass value="5"/>         <!-- 字宽: 1-9, 5=Medium(normal) -->
  <panose> ... </panose>            <!-- PANOSE 分类(10维视觉特征) -->
  <achVendID value="TMC "/>         <!-- 厂商标识 -->
  <sTypoAscender value="1420"/>
  <sTypoDescender value="-442"/>
  <sTypoLineGap value="307"/>
  <usWinAscent value="1825"/>
  <usWinDescent value="443"/>
  <sxHeight value="916"/>           <!-- x 字高 -->
  <sCapHeight value="1356"/>        <!-- 大写字高 -->
  ...
</OS_2>
```

`usWeightClass` 决定字重（100 细 → 900 黑），`PANOSE` 是 10 维字体视觉分类。

## 6.6 `post`

```xml
<post>
  <formatType value="3.0"/>         <!-- 版本3=不含字形名(省空间) -->
  <italicAngle value="0.0"/>
  <underlinePosition value="-223"/>
  <underlineThickness value="100"/>
  <isFixedPitch value="0"/>         <!-- 是否等宽 -->
  ...
</post>
```

`formatType 2.0` 含完整字形名表，`3.0` 省略（现代字体常用以减小体积）。

## 6.7 `cmap` 字符映射

把 Unicode 码点映射到字形名。CJK 字体的 cmap 极其丰富：

```xml
<cmap>
  <tableVersion version="0"/>
  <cmap_format_4 platformID="0" platEncID="3" language="0">
    ...
    <map code="0x20" name="space"/><!-- SPACE -->
    <map code="0x21" name="exclam"/><!-- EXCLAMATION MARK -->
    <map code="0x22" name="quotedbl"/><!-- QUOTATION MARK -->
    <map code="0x23" name="numbersign"/><!-- NUMBER SIGN -->
    <map code="0x24" name="dollar"/><!-- DOLLAR SIGN -->
    ...
```

注释里的字符名（`<!-- SPACE -->`）来自 Unicode 数据库，可用 `--unicodedata` 自定义。一个字体通常含多个子表（`cmap_format_4` UCS-2、`cmap_format_12` 全 Unicode、`cmap_format_14` 异体序列等），覆盖不同平台/编码。

## 6.8 `glyf` TrueType 轮廓

每个字形是一个 `<TTGlyph>`，含若干 `<contour>`（轮廓），轮廓由点组成，`on` 控制点是顶点、`off` 是二次贝塞尔控制点。字形 `I`：

```xml
<TTGlyph name="I" xMin="51" yMin="0" xMax="632" yMax="1356">
  <contour>
    <pt x="632" y="37" on="1"/>     <!-- on=轮廓顶点 -->
    <pt x="632" y="0" on="1"/>
    <pt x="51" y="0" on="1"/>
    <pt x="51" y="37" on="1"/>
    <pt x="99" y="37" on="1"/>
    <pt x="183" y="37" on="0"/>     <!-- off=二次贝塞尔控制点 -->
    <pt x="221" y="86" on="1"/>
    <pt x="245" y="118" on="0"/>
    <pt x="245" y="240" on="1"/>
    ...
```

复合字形（如带重音的字母）用 `<component>` 引用其他字形。

## 6.9 `CFF` PostScript 轮廓（CID-keyed）

思源黑体的 CFF 表展示了 **CID-keyed** 结构（CJK 字体典型）：

```xml
<CFF>
  <major value="1"/>  <minor value="0"/>
  <CFFFont name="SourceHanSansSC-Regular">
    <ROS Registry="Adobe" Order="Identity" Supplement="0"/>  <!-- CID Registry/Ordering -->
    <FullName value="Source Han Sans Simplified Chinese Regular"/>
    <FontMatrix value="0.001 0 0 0.001 0 0"/>   <!-- 1/1000 缩放 -->
    <FontBBox value="-1002 -1048 2928 1808"/>
    <CIDFontVersion value="2.00500011"/>
    <CIDCount value="65535"/>                    <!-- CID 总数 -->
    <FDSelect format="3"/>
    <FDArray>
      <FontDict index="0">
        <FontName value="SourceHanSansSC-Regular-Alphabetic"/>
        <Private>
          <BlueValues value="-13 0 544 557 735 747"/>  <!-- CFF hinting 对齐区 -->
          <OtherBlues value="-250 -229"/>
          <StdHW value="78"/>  <StdVW value="85"/>      <!-- 标准笔画宽 -->
          <StemSnapH value="78 111"/>  <StemSnapV value="85 95"/>
          ...
```

关键概念：
- **`ROS`**（Registry/Ordering/Supplement）声明这是 CID 字体，用 Adobe-Identity-0 排序。
- **`CIDCount`** 65535 = CID 编号空间上限。
- **`FDArray`/`FDSelect`**：把字形分组到多个 FontDict（不同字符集用不同 hinting 参数），是 CJK CID 字体管理海量字形的方式。
- CFF 的 hinting 在 `Private` dict 里（`BlueValues` 对齐区、`StdHW` 笔画宽），与 TrueType 的指令式 hinting 完全不同。

## 6.10 `MATH` 数学排版表

Cambria Math 独有，定义数学排版常量与字形变体：

```xml
<MATH>
  <Version value="0x00010000"/>
  <MathConstants>
    <ScriptPercentScaleDown value="73"/>          <!-- 下标缩放到 73% -->
    <ScriptScriptPercentScaleDown value="60"/>    <!-- 二级下标缩放到 60% -->
    <DelimitedSubFormulaMinHeight value="3000"/>
    <DisplayOperatorMinHeight value="2500"/>      <!-- 显示模式算符最小高度 -->
    <MathLeading><Value value="300"/></MathLeading>
    <AxisHeight>                                  <!-- 数学轴高度(分数线位置) -->
      <Value value="585"/>
      <DeviceTable> ... </DeviceTable>            <!-- 按字号的微调表 -->
    </AxisHeight>
    <AccentBaseHeight><Value value="976"/></AccentBaseHeight>
    <SubscriptShiftDown>                          <!-- 下标下移量 -->
      <Value value="418"/>
      <DeviceTable> ... </DeviceTable>
    </SubscriptShiftDown>
    ...
```

`MATH` 表还含 `MathGlyphInfo`（拉伸字形、斜体修正）和 `MathVariants`（巨型括号、根号等的变体组装），是 Word 公式编辑器、LaTeX 渲染器做高质量数学排版的基础。

## 6.11 `GSUB` / `GPOS` OpenType 布局

**GSUB（字形替换）** 定义 ScriptList/FeatureList/LookupList 三级结构：

```xml
<GSUB>
  <Version value="0x00010000"/>
  <ScriptList>
    <!-- ScriptCount=5 -->
    <ScriptRecord index="0">
      <ScriptTag value="arab"/>           <!-- 阿拉伯文脚本 -->
      <Script>
        <DefaultLangSys>
          <ReqFeatureIndex value="65535"/>
          <!-- FeatureCount=11 -->
          <FeatureIndex index="0" value="5"/>
          <FeatureIndex index="1" value="10"/>
          ...
        </DefaultLangSys>
        <LangSysRecord index="0">
          <LangSysTag value="FAR "/>      <!-- 波斯语 -->
          <LangSys> ... </LangSys>
```

结构：脚本(Script，如 `latn`/`arab`/`hani`) → 语言(LangSys，如 `ENG `) → 特性(Feature，如 `liga` 连字/`kern` 字距) → 查找(Lookup，具体替换规则)。

- **GSUB** 做替换：连字(`f`+`i`→`fi`)、风格替换、小型大写等。
- **GPOS** 做定位：字距调整(`kern`)、标注定位、连音符等。

## 6.12 `vhea` / `vmtx` 纵向排版（CJK）

CJK 字体需要纵向（竖排）度量：

```xml
<vhea>
  <tableVersion value="0x00010000"/>
  <ascent value="1740"/>  <descent value="-308"/>
  <advanceHeightMax value="2048"/>   <!-- 最大前进高度(竖排行进方向) -->
  <numberOfVMetrics value="1"/>
  ...
</vhea>
```

```xml
<vmtx>
  <mtx name=".notdef" height="2048" tsb="269"/>  <!-- height=前进高度, tsb=顶部间距 -->
  <mtx name="A" height="2048" tsb="249"/>
  ...
</vmtx>
```

## 6.13 `VORG` / `BASE`

**VORG**（CFF 字体纵向原点）记录字形在竖排时的垂直原点：

```xml
<VORG>
  <defaultVertOriginY value="880"/>      <!-- 默认纵向原点 Y -->
  <VOriginRecord>
    <glyphName value="cid00736"/>        <!-- 需要单独调整的字形(CID名) -->
    <vOrigin value="867"/>
  </VOriginRecord>
  ...
</VORG>
```

**BASE** 表定义不同脚本的基线：

```xml
<BASE>
  <HorizAxis>
    <BaseTagList>
      <BaselineTag index="0" value="icfb"/>   <!-- 表意字下基线 -->
      <BaselineTag index="1" value="icft"/>   <!-- 表意字上基线 -->
      <BaselineTag index="2" value="ideo"/>   <!-- 表意字基线 -->
      <BaselineTag index="3" value="romn"/>   <!-- 罗马字基线 -->
    </BaseTagList>
    ...
```

`icfb`/`icft`/`ideo` 是 CJK 表意字专用基线，让混合排版时不同脚本对齐正确。

## 6.14 hinting：`fpgm` / `prep` / `cvt`

TrueType 字体的 hinting 由三张表配合（见 [4.8 节](#48--i-不反汇编-truetype-指令) 看 `fpgm` 内容）：

- **`fpgm`**（font program）：字体级函数定义，全局只执行一次，定义供字形调用的函数。
- **`prep`**（pre-program）：每次字号/分辨率变化时执行，设置控制值和状态。
- **`cvt`**（control value table）：一组预定义数值，hinting 程序用它对齐笔画到像素网格：

```xml
<cvt>
  <cv index="0" value="1422"/>   <!-- 常与大写高度相关 -->
  <cv index="2" value="1356"/>   <!-- cap height -->
  <cv index="6" value="916"/>    <!-- x-height -->
  <cv index="14" value="-438"/>  <!-- descender 相关 -->
  ...
</cvt>
```

`cvt` 里的值（如 1356=cap height、916=x-height）与 OS/2 表里的 `sCapHeight`/`sxHeight` 呼应，是 hinting 把字形关键高度"吸"到整数像素的依据。

---

# 第七部分　字体特性与功能

## 7.1 两种轮廓格式

| | TrueType `glyf` | PostScript `CFF ` |
|---|---|---|
| 曲线 | 二次贝塞尔（`on`/`off` 点） | 三次贝塞尔 |
| 数据形态 | 显式点坐标 + 指令 | 紧凑的 charstring 操作码 |
| hinting | 指令式（强大、可编程） | 声明式（BlueValues 等） |
| 典型 | `.ttf`（如 Times、雅黑） | `.otf`（如思源、Adobe 字体） |

判定方法：`ttx -l` 看有 `glyf` 还是 `CFF ` 表。

## 7.2 TTC 字体集合

`.ttc`（TrueType Collection）/`.otc` 把多个**共享字形数据**的字体打包在一个文件里（如一套字体的多个字重），省空间。用 `ttx -y N` 选择第 N 个（从 0 起）。本文中 `msyh.ttc`、`cambria.ttc` 各含 2 个字体。

## 7.3 OpenType 布局特性（GSUB/GPOS）

让字体支持复杂排版：连字(`liga`)、小型大写(`smcp`)、花体替换(`swsh`)、字距(`kern`)、阿拉伯文/天城文等复杂脚本的形变。通过 GSUB（替换）+ GPOS（定位）实现，按 脚本→语言→特性→查找 四级组织。

## 7.4 数学排版（MATH 表）

仅数学字体（如 Cambria Math）拥有。提供数学常量（上下标缩放、轴高、极限堆叠间距）、拉伸字形（巨型括号/根号）、斜体修正等，是 Word 公式、LaTeX 引擎高质量渲染数学公式的基础。

## 7.5 CJK 纵向排版

中文/日文/韩文需要竖排，靠 `vhea`/`vmtx`（纵向度量）+ `VORG`（CFF 纵向原点）+ `BASE`（表意字基线）+ GSUB 里的竖排异体替换（如标点旋转）实现。雅黑、思源都有完整的纵向表。

## 7.6 TrueType hinting

TrueType 字体可在小字号下用字节码指令把字形对齐到像素网格，保证清晰锐利。三张表配合：`fpgm`（函数）、`prep`（预程序）、`cvt`（控制值）。ttx 默认反汇编成可读指令，`-i` 可保留为 hex。

## 7.7 嵌入式点阵（EBDT/EBLC、CBDT/CBLC）

部分字体在小字号下用预渲染的位图替代矢量轮廓以保证清晰（如 Cambria 的 `EBDT`/`EBLC`）。ttx 用 `-z` 控制位图导出格式（`raw`/`row`/`bitwise`/`extfile`），其中 `bitwise` 能肉眼看到点阵形状。

## 7.8 可变字体（fvar/gvar/HVAR，简介）

可变字体（Variable Fonts）在一份文件里用 `fvar`（轴定义，如字重/字宽）、`gvar`（字形变体数据）、`HVAR`/`VVAR`（度量变体）表达一个设计空间，浏览器/应用通过插值得到任意中间样式。ttx 能 dump/compile 这些表（需带 `--optimize-font-speed` 影响其编译），本文演示字体中暂无可变字体实例。

---

# 第八部分　常见工作流

## 8.1 修改字体名称

```bash
ttx -t name font.ttf                 # 只导出 name 表
# 编辑 font.ttx 里的 <namerecord>
ttx -m font.ttf -o newfont.ttf font.ttx   # 合并回原字体
```

## 8.2 提取 / 替换单张表

```bash
ttx -t cmap font.ttf                 # 提取 cmap
# 修改后
ttx -m font.ttf -o fixed.ttf font.ttx
```

## 8.3 字体子集化

ttx 本身不做子集化，但同属 fontTools 的 `pyftsubset` 是配套工具，可按字符集裁剪字体（Web 嵌入常用）：

```bash
pyftsubset font.ttf --text="Hello世界" --output-file=font.subset.ttf
```

## 8.4 对比两个字体

```bash
ttx -x glyf -o a.ttx fontA.ttf
ttx -x glyf -o b.ttx fontB.ttf
diff a.ttx b.ttx                     # 排除轮廓后比对元数据差异
```

## 8.5 转换为 WOFF2

```bash
ttx font.ttf                                  # 字体 → ttx
ttx --flavor woff2 -o font.woff2 font.ttx     # ttx → woff2
```

---

# 附录 A　命令行选项速查表

| 选项 | 类别 | 作用 |
|------|------|------|
| `-h` | 通用 | 打印帮助 |
| `--version` | 通用 | 显示版本 |
| `-d <目录>` | 通用 | 输出目录（须先存在） |
| `-o <文件>` | 通用 | 输出文件名；`-o -` 为 stdout（与 `-d` 同用时优先） |
| `-f` | 通用 | 覆盖而非追加序号 |
| `-v` | 通用 | 详细输出 |
| `-q` | 通用 | 静默 |
| `-a` | 通用 | 允许虚拟字形 ID |
| `-l` | Dump | 列表模式（不导出） |
| `-t <表>` | Dump | 只导出指定表（可多个） |
| `-x <表>` | Dump | 排除指定表（可多个，与 `-t` 互斥） |
| `-s` | Dump | 按表拆分 |
| `-g` | Dump | 按字形拆分 glyf（隐含 `-s`） |
| `-i` | Dump | 指令不反汇编（输出 hex） |
| `-z <格式>` | Dump | 位图格式 raw/row/bitwise/extfile |
| `-e` | Dump | 不忽略错误（完整 traceback） |
| `-y <编号>` | Dump | 选 TTC 字体编号 |
| `--unicodedata` | Dump | 自定义字符名数据库 |
| `--newline` | Dump | 行尾 LF/CR/CRLF |
| `-m <字体>` | Compile | 合并 TTX 进已有字体 |
| `-b` | Compile | 不重算字形 bbox |
| `--recalc-timestamp` | Compile | 时间戳设为当前 |
| `--no-recalc-timestamp` | Compile | 保留原时间戳 |
| `--flavor <类型>` | Compile | woff / woff2 |
| `--with-zopfli` | Compile | WOFF 用 Zopfli 压缩 |
| `--optimize-font-speed` | Compile | 渲染速度优先 |

---

# 附录 B　演示字体清单

| 文件 | 类型 | 来源 | 演示重点 |
|------|------|------|----------|
| `times.ttf` | 西文 TrueType | Windows 系统字体 | glyf 轮廓、TrueType hinting(fpgm/prep/cvt)、GSUB/GPOS/JSTF、hdmx/VDMX |
| `msyh.ttc` | 中文 TTC(2字体) | Windows 微软雅黑 | TTC 集合(`-y`)、超大 cmap、CJK 纵向排版(vhea/vmtx)、MERG |
| `cambria.ttc` | 数学 TTC(2字体) | Windows Office | **MATH 表**、嵌入式点阵(EBDT/EBLC)、`-z` 位图格式 |
| `SourceHanSansSC-Regular.otf` | 中文 CFF | 思源黑体(开源) | **CID-keyed CFF**、VORG、BASE、6 字节 maxp、超大 GSUB |

> 以上均为真实字体（系统自带或开源），非合成。

---

# 附录 C　常见表标签速查

| 标签 | 全称 | 作用 |
|------|------|------|
| `head` | Font Header | 字体头（版本、unitsPerEm、bbox、时间戳） |
| `name` | Naming | 字体名称、版权、版本字符串 |
| `hhea` | Horizontal Header | 水平排版头 |
| `hmtx` | Horizontal Metrics | 每字形水平度量（width/lsb） |
| `vhea` | Vertical Header | 纵向排版头（CJK 竖排） |
| `vmtx` | Vertical Metrics | 每字形纵向度量 |
| `maxp` | Maximum Profile | 字形数 + 限制（TTF 详细 / CFF 极简） |
| `OS/2` | OS/2 and Windows Metrics | 字重、字宽、PANOSE、Win 度量 |
| `post` | PostScript Name | 字形名、斜体角、下划线 |
| `cmap` | Character to Glyph Mapping | Unicode → 字形 |
| `loca` | Index to Location | glyf 的字形偏移索引 |
| `glyf` | Glyph Data | TrueType 轮廓 |
| `CFF ` | Compact Font Format | PostScript/CFF 轮廓（含 CID） |
| `VORG` | Vertical Origin | CFF 纵向原点 |
| `BASE` | Baseline Data | 多脚本基线 |
| `GSUB` | Glyph Substitution | 替换（连字/小型大写等） |
| `GPOS` | Glyph Positioning | 定位（字距/标注） |
| `GDEF` | Glyph Definition | 字形类别、附着点 |
| `JSTF` | Justification | 两端对齐 |
| `fpgm` | Font Program | TrueType 全局函数 |
| `prep` | Control Value Program | TrueType 预程序 |
| `cvt ` | Control Value Table | TrueType hinting 控制值 |
| `gasp` | Grid-fitting/Scan-conversion | 按字号的 hinting 策略 |
| `MATH` | Math Layout | 数学排版常量与变体 |
| `EBDT`/`EBLC` | Embedded Bitmap Data/Location | 灰度点阵 |
| `CBDT`/`CBLC` | Color Bitmap Data/Location | 彩色点阵 |
| `hdmx` | Horizontal Device Metrics | 按字号的水平度量 |
| `VDMX` | Vertical Device Metrics | 按字号的垂直度量 |
| `LTSH` | Linear Threshold | 线性缩放阈值 |
| `kern` | Kerning | 旧式字距（已被 GPOS 取代） |
| `DSIG` | Digital Signature | 数字签名 |
| `meta` | Metadata | 元数据 |
| `fvar` | Font Variations | 可变字体轴定义 |
| `gvar` | Glyph Variations | 可变字体字形变体 |
| `HVAR`/`VVAR` | Metrics Variations | 可变字体度量变体 |

---

*文档生成环境：fontTools 4.61.0 · Windows 11 · 演示命令均实际运行验证。*
