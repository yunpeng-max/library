# library# 图书借阅管理系统 — 流程图

> 使用 Mermaid 语法绘制，在支持 Mermaid 的 Markdown 渲染器中可直接查看（GitHub、VS Code + 插件等）。

---

## 一、系统主流程图

```mermaid
flowchart TD
    A["程序启动 main()"] --> B["显示欢迎信息"]
    B --> C["load_books() 加载 CSV 数据"]
    C --> D{"CSV 文件是否存在？"}
    D -->|"不存在"| E["创建带表头的空 CSV 文件"]
    E --> F["books = 空列表"]
    D -->|"存在"| G["解析 CSV 行到 books 列表"]
    G --> H["数据清洗: 标准化状态/借阅次数/借阅人字段"]
    H --> F
    F --> I["输出: 已加载 N 本图书"]
    I --> J["检查 matplotlib 可用性并提示"]

    J --> K["进入主循环 while True"]
    K --> L["display_menu() 显示主菜单"]
    L --> M["用户输入选项 choice (0-7)"]

    M --> N{"handle_choice() 分发"}
    N -->|"1"| O["add_book() 添加图书"]
    N -->|"2"| P["query_book() 查询图书"]
    N -->|"3"| Q["borrow_book() 借阅图书"]
    N -->|"4"| R["return_book() 归还图书"]
    N -->|"5"| S["delete_book() 删除图书"]
    N -->|"6"| T["show_all() 显示全部图书"]
    N -->|"7"| U["get_statistics() 统计与可视化"]
    N -->|"0"| V["输出再见信息 → 退出循环"]
    N -->|"无效输入"| W["提示错误 → 按回车继续"]

    O --> X["press_enter_to_continue()"]
    P --> X
    Q --> X
    R --> X
    S --> X
    T --> X
    U --> X
    W --> X

    X -->|"捕获 CancelOperation?"| Y{"是否捕获到取消异常？"}
    Y -->|"是"| Z["提示已取消 → 按回车继续"]
    Z --> K
    Y -->|"否"| K

    V --> END["程序结束"]

    style A fill:#4CAF50,color:#fff
    style END fill:#f44336,color:#fff
    style K fill:#2196F3,color:#fff
    style N fill:#FF9800,color:#fff
```

---

## 二、数据加载模块 — load_books()

```mermaid
flowchart TD
    A["load_books(filepath)"] --> B{"文件是否存在？"}
    B -->|"不存在"| C["创建新 CSV 文件 + 写入表头"]
    C --> D["返回空列表 []"]
    B -->|"存在"| E["以 utf-8-sig 编码打开文件"]
    E --> F["csv.DictReader 逐行读取"]
    F --> G{"逐行解析"}

    G --> H{"行是否有 '编号' 和 '书名'？"}
    H -->|"缺少"| I["输出警告，跳过该行"]
    H -->|"有"| J["去除所有字段首尾空白"]
    J --> K["转换 '借阅次数' 为 int（失败则置 0）"]
    K --> L["标准化 '状态' 字段"]

    L --> M{"状态值是什么？"}
    M -->|"'0' / '未借出' / '在库'"| N["设为 '在库'"]
    M -->|"'1' / '已借出'"| O["设为 '已借出'"]
    M -->|"其他"| P["默认设为 '在库'"]

    N --> Q["确保 '借阅人' 字段存在"]
    O --> Q
    P --> Q
    Q --> R["将该行字典加入 books 列表"]
    R --> G

    I --> G

    G -->|"全部行处理完毕"| S["返回 books 列表"]

    style A fill:#4CAF50,color:#fff
    style D fill:#2196F3,color:#fff
    style S fill:#2196F3,color:#fff
```

---

## 三、数据保存模块 — save_books()

```mermaid
flowchart TD
    A["save_books(books, filepath)"] --> B["生成临时路径: filepath + '.tmp'"]
    B --> C["以 utf-8-sig 编码写入临时文件"]
    C --> D{"写入是否成功？"}
    D -->|"成功"| E["os.replace() 原子替换原文件"]
    E --> F["返回 True"]
    D -->|"失败 (IOError)"| G["输出错误信息"]
    G --> H{"临时文件是否存在？"}
    H -->|"是"| I["删除临时文件"]
    I --> J["返回 False"]
    H -->|"否"| J

    style A fill:#4CAF50,color:#fff
    style F fill:#4CAF50,color:#fff
    style J fill:#f44336,color:#fff
```

---

## 四、操作日志模块 — log_history()

```mermaid
flowchart TD
    A["log_history(action, book_id, book_title, borrower, detail)"] --> B["生成时间戳 timestamp"]
    B --> C["格式化日志行: timestamp | action | book_id | book_title | borrower | detail"]
    C --> D["以追加模式打开 history.log"]
    D --> E{"写入是否成功？"}
    E -->|"成功"| F["日志写入完成（无返回值）"]
    E -->|"失败 (IOError)"| G["输出: [警告] 写入操作日志失败"]

    style A fill:#4CAF50,color:#fff
    style F fill:#4CAF50,color:#fff
    style G fill:#FF9800,color:#fff
```

---

## 五、添加图书模块 — add_book()

```mermaid
flowchart TD
    A["add_book(books)"] --> B["输出: --- 添加图书 ---"]
    B --> C["get_input: 输入图书编号"]
    C --> D["find_book() 检查编号是否已存在"]
    D --> E{"编号是否重复？"}
    E -->|"重复"| F["输出错误，按回车返回"]
    E -->|"不重复"| G["get_input: 输入书名"]
    G --> H["get_input: 输入作者"]
    H --> I["get_input: 输入出版社"]
    I --> J["构建 book 字典: 状态='在库', 借阅人='', 借阅次数=0"]
    J --> K["books.append(book)"]
    K --> L["save_books() 保存到 CSV"]
    L --> M{"保存是否成功？"}
    M -->|"成功"| N["输出: [成功] 图书添加成功"]
    M -->|"失败"| O["输出: [警告] 已加入内存但保存失败"]
    N --> P["press_enter_to_continue()"]
    O --> P

    style A fill:#4CAF50,color:#fff
    style F fill:#FF9800,color:#fff
    style N fill:#4CAF50,color:#fff
```

---

## 六、查询图书模块 — query_book()

```mermaid
flowchart TD
    A["query_book(books)"] --> B{"books 是否为空？"}
    B -->|"空"| C["输出: 书库为空 → 按回车返回"]
    B -->|"非空"| D["get_input: 输入搜索关键词（允许为空）"]
    D --> E{"关键词是否为空？"}
    E -->|"空"| F["输出: 未输入关键词 → 按回车返回"]
    E -->|"非空"| G["列表推导式: fuzzy_match(书名, keyword) OR fuzzy_match(作者, keyword)"]
    G --> H{"匹配结果是否为空？"}
    H -->|"空"| I["输出: 未找到匹配 → 按回车返回"]
    H -->|"非空"| J["输出: 找到 N 本匹配图书"]
    J --> K["_print_book_table() 打印格式化表格"]
    K --> L["press_enter_to_continue()"]

    style A fill:#4CAF50,color:#fff
    style C fill:#9E9E9E,color:#fff
    style F fill:#9E9E9E,color:#fff
    style I fill:#9E9E9E,color:#fff
```

---

## 七、借阅图书模块 — borrow_book()

```mermaid
flowchart TD
    A["borrow_book(books)"] --> B{"books 是否为空？"}
    B -->|"空"| C["输出: 书库为空 → 按回车返回"]
    B -->|"非空"| D["get_input: 输入要借阅的图书编号"]
    D --> E["find_book() 查找图书"]
    E --> F{"编号是否存在？"}
    F -->|"不存在"| G["输出: [错误] 图书不存在 → 按回车返回"]
    F -->|"存在"| H{"图书状态是否为 '已借出'？"}
    H -->|"已借出"| I["输出: [错误] 已被 xx 借出 → 按回车返回"]
    H -->|"'在库'"| J["get_input: 输入借阅人姓名"]
    J --> K["修改图书: 状态='已借出', 借阅人=姓名, 借阅次数+1"]
    K --> L["save_books() 保存"]
    L --> M{"保存是否成功？"}
    M -->|"成功"| N["log_history('借阅', ...)"]
    N --> O["输出: [成功] 借阅成功"]
    M -->|"失败"| P["输出: [警告] 已借阅但保存失败"]
    O --> Q["press_enter_to_continue()"]
    P --> Q

    style A fill:#4CAF50,color:#fff
    style O fill:#4CAF50,color:#fff
    style G fill:#f44336,color:#fff
    style I fill:#f44336,color:#fff
    style P fill:#FF9800,color:#fff
```

---

## 八、归还图书模块 — return_book()

```mermaid
flowchart TD
    A["return_book(books)"] --> B{"books 是否为空？"}
    B -->|"空"| C["输出: 书库为空 → 按回车返回"]
    B -->|"非空"| D["get_input: 输入要归还的图书编号"]
    D --> E["find_book() 查找图书"]
    E --> F{"编号是否存在？"}
    F -->|"不存在"| G["输出: [错误] 图书不存在 → 按回车返回"]
    F -->|"存在"| H{"图书状态是否为 '在库'？"}
    H -->|"'在库'"| I["输出: [提示] 未被借出无需归还 → 按回车返回"]
    H -->|"'已借出'"| J["记录当前借阅人 borrower"]
    J --> K["修改图书: 状态='在库', 借阅人=''"]
    K --> L["save_books() 保存"]
    L --> M{"保存是否成功？"}
    M -->|"成功"| N["log_history('归还', ...)"]
    N --> O["输出: [成功] 归还成功"]
    M -->|"失败"| P["输出: [警告] 已归还但保存失败"]
    O --> Q["press_enter_to_continue()"]
    P --> Q

    style A fill:#4CAF50,color:#fff
    style O fill:#4CAF50,color:#fff
    style G fill:#f44336,color:#fff
    style I fill:#2196F3,color:#fff
```

---

## 九、删除图书模块 — delete_book()

```mermaid
flowchart TD
    A["delete_book(books)"] --> B{"books 是否为空？"}
    B -->|"空"| C["输出: 书库为空 → 按回车返回"]
    B -->|"非空"| D["get_input: 输入要删除的图书编号"]
    D --> E["find_book() 查找图书"]
    E --> F{"编号是否存在？"}
    F -->|"不存在"| G["输出: [错误] 图书不存在 → 按回车返回"]
    F -->|"存在"| H{"图书状态是否为 '已借出'？"}
    H -->|"'已借出'"| I["输出: [错误] 已被借出请先归还 → 按回车返回"]
    H -->|"'在库'"| J["get_input: 确认删除? (y/n)"]
    J --> K{"用户是否输入 'y'？"}
    K -->|"否"| L["输出: 已取消删除 → 按回车返回"]
    K -->|"是"| M["books.pop(index) 从列表移除"]
    M --> N["save_books() 保存"]
    N --> O{"保存是否成功？"}
    O -->|"成功"| P["log_history('删除', ...)"]
    P --> Q["输出: [成功] 图书已删除"]
    O -->|"失败"| R["输出: [警告] 已从内存删除但保存失败"]
    Q --> S["press_enter_to_continue()"]
    R --> S

    style A fill:#4CAF50,color:#fff
    style Q fill:#4CAF50,color:#fff
    style G fill:#f44336,color:#fff
    style I fill:#f44336,color:#fff
    style L fill:#9E9E9E,color:#fff
```

---

## 十、显示全部图书模块 — show_all()

```mermaid
flowchart TD
    A["show_all(books)"] --> B{"books 是否为空？"}
    B -->|"空"| C["输出: 书库为空 → 提示使用添加功能 → 按回车返回"]
    B -->|"非空"| D["sorted(books, key=编号) 按编号排序"]
    D --> E["_print_book_table() 打印格式化表格"]
    E --> F["计算各列显示宽度 (CJK字符=2, ASCII=1)"]
    F --> G["打印表头分隔线"]
    G --> H["打印表头: 编号 书名 作者 出版社 状态 借阅人 借阅次数"]
    H --> I["循环打印每一行数据 (对齐后)"]
    I --> J["打印底部分隔线"]
    J --> K["输出: 共 N 本图书"]
    K --> L["press_enter_to_continue()"]

    style A fill:#4CAF50,color:#fff
```

---

## 十一、统计与可视化模块 — get_statistics()

```mermaid
flowchart TD
    A["get_statistics(books)"] --> B["计算: total / available / borrowed"]
    B --> C["输出文字统计信息"]
    C --> D["输出: 图书总数、在库数量、已借出数量"]
    D --> E{"total > 0?"}
    E -->|"是"| F["计算平均借阅次数"]
    F --> G["sorted 排序: 按借阅次数降序取 Top 5"]
    G --> H["输出 Top 5 借阅排行表格"]

    H --> I{"matplotlib 是否可用？"}
    I -->|"可用"| J["询问用户是否显示图表 (默认 y)"]
    J --> K{"用户选择？"}
    K -->|"y / 回车"| L["_draw_charts() 绘制图表"]
    K -->|"n"| M["跳过图表"]
    I -->|"不可用"| N["输出提示: matplotlib 未安装"]

    E -->|"否 (书库为空)"| O["输出提示: 书库为空无法生成图表"]

    L --> P["press_enter_to_continue()"]
    M --> P
    N --> P
    O --> P

    L --> L1["图1: 饼图 — 借阅状态分布"]
    L1 --> L2["图2: 横向条形图 — Top 5 借阅次数 (过滤0次)"]
    L2 --> L3["plt.show() 显示图表"]

    style A fill:#4CAF50,color:#fff
    style L1 fill:#9C27B0,color:#fff
    style L2 fill:#9C27B0,color:#fff
```

---

## 十二、菜单分发模块 — handle_choice()

```mermaid
flowchart TD
    A["handle_choice(choice, books)"] --> B{"choice 的值？"}

    B -->|"'1'"| C["add_book(books)"]
    B -->|"'2'"| D["query_book(books)"]
    B -->|"'3'"| E["borrow_book(books)"]
    B -->|"'4'"| F["return_book(books)"]
    B -->|"'5'"| G["delete_book(books)"]
    B -->|"'6'"| H["show_all(books)"]
    B -->|"'7'"| I["get_statistics(books)"]
    B -->|"'0'"| J["输出: 感谢使用，再见！"]
    J --> K["返回 False (退出主循环)"]
    B -->|"其他"| L["输出: 无效选择，请输入 0-7"]

    C --> M{"是否抛出 CancelOperation？"}
    D --> M
    E --> M
    F --> M
    G --> M
    H --> M
    I --> M
    L --> N["press_enter_to_continue()"]
    N --> M

    M -->|"是"| O["捕获异常 → 输出: 已取消 → 按回车继续"]
    M -->|"否"| P["按回车继续"]
    O --> Q["返回 True (继续主循环)"]
    P --> Q

    style A fill:#FF9800,color:#fff
    style K fill:#f44336,color:#fff
    style Q fill:#4CAF50,color:#fff
```

---

## 十三、各模块关系总览

```mermaid
flowchart LR
    subgraph 入口层
        MAIN["main() 主入口"]
        MENU["display_menu() 菜单"]
        DISPATCH["handle_choice() 分发"]
    end

    subgraph 业务层
        ADD["add_book() 添加"]
        QUERY["query_book() 查询"]
        BORROW["borrow_book() 借阅"]
        RETURN["return_book() 归还"]
        DELETE["delete_book() 删除"]
        SHOW["show_all() 显示全部"]
        STATS["get_statistics() 统计"]
    end

    subgraph 数据层
        LOAD["load_books() 加载"]
        SAVE["save_books() 保存"]
        LOG["log_history() 日志"]
    end

    subgraph 工具层
        INPUT["get_input() 输入"]
        FIND["find_book() 查找"]
        FUZZY["fuzzy_match() 模糊匹配"]
        TABLE["_print_book_table() 表格打印"]
        WIDTH["display_width() / pad_str() / truncate_str()"]
    end

    subgraph 可视化层
        CHARTS["_draw_charts() 图表"]
    end

    subgraph 存储
        CSV["books.csv"]
        LOGFILE["history.log"]
    end

    MAIN --> MENU
    MENU --> DISPATCH
    DISPATCH --> ADD & QUERY & BORROW & RETURN & DELETE & SHOW & STATS

    ADD --> INPUT & FIND & SAVE
    QUERY --> INPUT & FUZZY & TABLE
    BORROW --> INPUT & FIND & SAVE & LOG
    RETURN --> INPUT & FIND & SAVE & LOG
    DELETE --> INPUT & FIND & SAVE & LOG
    SHOW --> TABLE
    STATS --> CHARTS

    SAVE --> CSV
    LOAD --> CSV
    LOG --> LOGFILE
    CHARTS --> MATPLOTLIB["matplotlib.pyplot"]

    style MAIN fill:#4CAF50,color:#fff
    style DISPATCH fill:#FF9800,color:#fff
    style CSV fill:#607D8B,color:#fff
    style LOGFILE fill:#607D8B,color:#fff
```
