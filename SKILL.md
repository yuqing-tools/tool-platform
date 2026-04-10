# 网页工具转换标准规范

## 概述

将 exe 工具转换为网页版工具的标准流程，确保每个功能都被正确保留。

---

## 转换检查清单

### 一、输入输出

| 检查项 | 原工具实现 | 必须保留 |
|--------|-----------|---------|
| 输入文件数量 | 2个（今日数据.xlsx、待填表.xlsx） | ✅ |
| 输出文件 | 直接修改待填表，保存为新文件 | ✅ |
| 输出文件名格式 | `成品：蒙牛乳业舆情汇总{MMDD}.xlsx` | ✅ |

### 二、数据处理

| 检查项 | 原工具实现 | 必须保留 |
|--------|-----------|---------|
| 列A | 序号（最后重新编号） | ✅ |
| 列B | 标题 | ✅ |
| 列C | 摘要 | ✅ |
| 列D | 发布时间（yyyy/mm/dd hh:mm:ss） | ✅ |
| 列E | 发布人 | ✅ |
| 列F | 发布平台 | ✅ |
| 列G | 原文链接（https→http） | ✅ |
| 列H | 报送时间（今天日期，yyyy/mm/dd） | ✅ |
| 列I | 情感属性 | ✅ |
| 列J | 不填 | ✅ |
| 列K | 竞品品类（仅竞品分类时填写） | ✅ |

### 三、格式保留

| 检查项 | 原工具实现 | 必须保留 |
|--------|-----------|---------|
| 原表格格式 | openpyxl 直接操作单元格 | ⚠️ SheetJS 必须用 cell API |
| 样式复制 | 每行复制最后一行的样式 | ✅ |
| 公式保留 | 不修改原公式 | ✅ |
| 其他sheet | 保留所有原sheet | ✅ |

### 四、分类逻辑

| 检查项 | 原工具实现 | 必须保留 |
|--------|-----------|---------|
| 标题优先 | 先匹配标题 | ✅ |
| 摘要补充 | 标题无匹配时用摘要 | ✅ |
| 优先级 | 蒙牛 > 伊利 > 竞品 > 行业 | ✅ |
| 普通模式 | 直接使用已有分类（row[8]） | ✅ |
| 智能分类模式 | 关键词匹配 | ✅ |

### 五、去重与转换

| 检查项 | 原工具实现 | 必须保留 |
|--------|-----------|---------|
| URL去重 | 对比目标sheet中的URL | ✅ |
| https转http | 今日数据+待填表都转换 | ✅ |
| 日期格式化 | 发布时间 D列，yyyy/mm/dd hh:mm:ss | ✅ |
| 报送时间 | H列，今天日期，yyyy/mm/dd | ✅ |

---

## SheetJS 正确用法

### ❌ 错误做法（丢失格式）
```javascript
// 会丢失所有格式、公式、样式
const data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
// ... 处理数据 ...
const newSheet = XLSX.utils.aoa_to_sheet(data);
```

### ✅ 正确做法（保留格式）
```javascript
// 读取时保留样式
const wb = XLSX.read(data, { type: 'array', cellDates: true, cellStyles: true });

// 写入时用 sheet_add_json 追加到指定行
XLSX.utils.sheet_add_json(sheet, [newRowData], { skipHeader: true, origin: newRowIndex });

// 手动复制样式
if (sheet[refCellRef] && sheet[refCellRef].s) {
    if (!sheet[newCellRef]) sheet[newCellRef] = {};
    sheet[newCellRef].s = sheet[refCellRef].s;
}

// 设置日期格式
sheet[`D${newRowIndex}`].z = 'yyyy/mm/dd hh:mm:ss';
sheet[`H${newRowIndex}`].z = 'yyyy/mm/dd';
```

---

## 代码模板

```javascript
// ============ 核心处理模板 ============
function processTool(todayWorkbook, templateWorkbook, options) {
    // 1. 读取今日数据
    const todaySheet = todayWorkbook.Sheets[todayWorkbook.SheetNames[0]];
    const todayData = XLSX.utils.sheet_to_json(todaySheet, { header: 1 });

    // 2. 收集待填表URL（去重用）
    const existingUrls = {};
    for (const sheetName of templateWorkbook.SheetNames) {
        if (isExcludedSheet(sheetName)) continue;
        existingUrls[sheetName] = collectUrls(templateWorkbook.Sheets[sheetName]);
    }

    // 3. 转换待填表URL
    for (const sheetName of templateWorkbook.SheetNames) {
        if (isExcludedSheet(sheetName)) continue;
        convertHttpsToHttp(templateWorkbook.Sheets[sheetName]);
    }

    // 4. 记录处理前行数
    const rowCountsBefore = {};
    for (const sheetName of targetSheets) {
        rowCountsBefore[sheetName] = countDataRows(templateWorkbook.Sheets[sheetName]);
    }

    // 5. 处理每行数据
    for (let i = 1; i < todayData.length; i++) {
        const row = todayData[i];
        if (isEmptyRow(row)) continue;

        // 5.1 分类（根据options选择模式）
        const { category, brand } = classify(row, options);

        // 5.2 确定目标sheet
        const targetSheet = getTargetSheet(category);

        // 5.3 URL去重
        if (isDuplicate(row[6], existingUrls[targetSheet])) {
            skipped++;
            continue;
        }

        // 5.4 找到新行位置
        const newRowIndex = findLastDataRow(templateWorkbook.Sheets[targetSheet]) + 1;
        const refRow = newRowIndex > 2 ? newRowIndex - 1 : 2;

        // 5.5 写入数据
        const newRowData = buildRowData(row, category, brand, todayDate);
        XLSX.utils.sheet_add_json(templateWorkbook.Sheets[targetSheet], [newRowData], {
            skipHeader: true,
            origin: newRowIndex
        });

        // 5.6 设置日期格式
        setDateFormats(templateWorkbook.Sheets[targetSheet], newRowIndex);

        // 5.7 复制样式
        copyRowStyle(templateWorkbook.Sheets[targetSheet], refRow, newRowIndex);
    }

    // 6. 重新编号
    renumberAllSheets(templateWorkbook);

    // 7. 返回结果
    return {
        workbook: templateWorkbook,
        stats: calculateStats(rowCountsBefore, templateWorkbook),
        skipped: skipped
    };
}

// ============ 辅助函数 ============
function isExcludedSheet(name) {
    return name === '数据统计（公式勿动）';
}

function collectUrls(sheet) {
    const urls = new Set();
    const data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
    for (let i = 1; i < data.length; i++) {
        if (data[i][6]) urls.add(normalizeUrl(String(data[i][6])));
    }
    return urls;
}

function convertHttpsToHttp(sheet) {
    const data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
    let changed = false;
    for (let i = 1; i < data.length; i++) {
        if (data[i][6] && data[i][6].toString().startsWith('https://')) {
            data[i][6] = data[i][6].replace('https://', 'http://');
            changed = true;
        }
    }
    if (changed) {
        const newSheet = XLSX.utils.aoa_to_sheet(data);
        if (sheet['!cols']) newSheet['!cols'] = sheet['!cols'];
        Object.assign(sheet, newSheet);
    }
}

function normalizeUrl(url) {
    if (!url) return '';
    url = url.toLowerCase().trim()
        .replace(/^https?:\/\//, '')
        .replace(/^www\./, '')
        .replace(/\/+$/, '')
        .split('#')[0];
    return url;
}

function isEmptyRow(row) {
    return !row || (!row[1] && !row[2]);
}

function isDuplicate(url, existingUrls) {
    if (!url) return false;
    const normalized = normalizeUrl(String(url));
    return normalized && existingUrls.has(normalized);
}

function findLastDataRow(sheet) {
    const data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
    let last = 1;
    for (let i = 1; i < data.length; i++) {
        if (data[i] && data[i][1]) last = i + 1;
    }
    return last;
}

function buildRowData(row, category, brand, todayDate) {
    return [
        null,                      // A
        row[1] || null,           // B
        row[2] || null,           // C
        row[3] || null,           // D
        row[4] || null,           // E
        row[5] || null,           // F
        convertHttpsToHttp(row[6]) || null,  // G
        todayDate,                 // H
        row[7] || null,           // I
        null,                      // J
        category === '竞品' ? brand : null  // K
    ];
}

function setDateFormats(sheet, rowIndex) {
    // D列：发布时间
    const dCell = sheet[`D${rowIndex}`];
    if (dCell) {
        dCell.t = 'd';
        dCell.z = 'yyyy/mm/dd hh:mm:ss';
    }
    // H列：报送时间
    const hCell = sheet[`H${rowIndex}`];
    if (hCell) {
        hCell.t = 'd';
        hCell.z = 'yyyy/mm/dd';
    }
}

function copyRowStyle(sheet, refRow, newRow) {
    for (let col = 0; col < 11; col++) {
        const colLetter = String.fromCharCode(65 + col);
        const refRef = `${colLetter}${refRow}`;
        const newRef = `${colLetter}${newRow}`;
        if (sheet[refRef] && sheet[refRef].s) {
            if (!sheet[newRef]) sheet[newRef] = {};
            sheet[newRef].s = sheet[refRef].s;
        }
    }
}

function renumberAllSheets(templateWorkbook) {
    for (const sheetName of targetSheets) {
        if (!templateWorkbook.Sheets[sheetName]) continue;
        const sheet = templateWorkbook.Sheets[sheetName];
        const data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
        let seq = 1;
        for (let i = 1; i < data.length; i++) {
            const cellRef = `A${i + 1}`;
            if (data[i] && data[i][1]) {
                sheet[cellRef] = { t: 'n', v: seq++ };
            } else {
                sheet[cellRef] = { t: 's', v: '' };
            }
        }
    }
}
```

---

## 上传新工具步骤

### 1. 创建工具文件夹
```
tool-platform/
├── tools/
│   └── [tool-id]/
│       └── config.json
└── index.html
```

### 2. 编写 config.json
```json
{
    "id": "tool-id",
    "name": "工具名称",
    "icon": "🔧",
    "description": "工具描述",
    "version": "v1.0",
    "author": "作者",
    "inputFiles": ["文件1.xlsx", "文件2.xlsx"],
    "outputFile": "结果.xlsx"
}
```

### 3. 在 index.html 添加工具配置
```javascript
const TOOLS_CONFIG = [
    // 现有工具...
    {
        id: 'new-tool',
        name: '新工具',
        icon: '🔧',
        description: '...',
        version: 'v1.0',
        author: '作者'
    }
];
```

### 4. 添加处理函数
按照上方模板实现 `processNewTool` 函数，并在 `runTool()` 中调用。

---

## 常见错误

| 错误 | 原因 | 解决 |
|------|------|------|
| 格式丢失 | 用了 sheet_to_json + aoa_to_sheet | 改用 sheet_add_json + cell API |
| 样式丢失 | 没复制样式 | 必须复制 refRow 的样式到新行 |
| 日期格式错 | 没设置 number_format | 必须设置 `cell.z = 'yyyy/mm/dd hh:mm:ss'` |
| 文件名错 | 格式不对 | 按原工具格式：`成品：xxx{MMDD}.xlsx` |
| 报送时间错 | 用了 datetime 而非 date | 应该是 `yyyy/mm/dd`，不是 `yyyy/mm/dd hh:mm:ss` |
