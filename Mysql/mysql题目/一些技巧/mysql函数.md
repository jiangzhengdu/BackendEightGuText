函数
===

MySQL聚合函数
---

    AVG - 计算一组值或表达式的平均值。
    COUNT - 计算表中的行数。
    INSTR - 返回字符串中第一次出现的子字符串的位置。
    SUM  - 计算一组值或表达式的总和。
    MIN - 在一组值中找到最小值
    MAX - 在一组值中找到最大值

MySQL字符串函数
---

    CONCAT - 将两个或多个字符串组合成一个字符串。
    LENGTH＆CHAR_LENGTH  - 获取字符串的长度，以字节和字符为单位。
    LEFT - 获取具有指定长度的字符串的左侧部分。
    REPLACE - 搜索并替换字符串中的子字符串。
    SUBSTRING - 从具有特定长度的位置开始提取子字符串。
    TRIM - 从字符串中删除不需要的字符。
    FIND_IN_SET - 在以逗号分隔的字符串列表中查找字符串。
    FORMAT - 格式化具有特定区域设置的数字，四舍五入到小数位数

MySQL控制流功能
---

    CASE - THEN如果WHEN满足分支中的条件，则返回分支中的相应结果，否则返回ELSE分支中的结果。
    IF - 根据给定条件返回值。
    IFNULL - 如果它不是NULL则返回第一个参数，否则返回第二个参数。
    NULLIF - 如果第一个参数等于第二个参数，则返回NULL，否则返回第一个参数。

MySQL日期和时间函数
---

    CURDATE - 返回当前日期。
    DATEDIFF  - 计算两个DATE值之间的天数   。
    DAY  - 获取指定日期的月份日期。
    DATE_ADD  - 将日期值添加到日期值。
    DATE_SUB - 从日期值中减去时间值。
    DATE_FORMAT - 根据指定的日期格式格式化日期值。
    DAYNAME - 获取指定日期的工作日名称。
    DAYOFWEEK - 返回日期的工作日索引。
    EXTRACT - 提取日期的一部分。
    NOW - 返回执行语句的当前日期和时间。
    MONTH - 返回表示指定日期月份的整数。
    STR_TO_DATE - 根据指定的格式将字符串转换为日期和时间值。
    SYSDATE - 返回当前日期。
    TIMEDIFF - 计算两个TIME或DATETIME值之间的差异。
    TIMESTAMPDIFF - 计算两个DATE或DATETIME值之间的差异。
    WEEK - 返回一个星期的日期。
    WEEKDAY  - 返回日期的工作日索引。
    YEAR -返回日期值的年份部分。

MySQL比较功能
---

    COALESCE - 返回第一个非null参数，这对于替换null非常方便。
    GREATEST＆LEAST - 取n个参数并分别返回n个参数的最大值和最小值。
    ISNULL - 如果参数为null，则返回1，否则返回零。

MySQL数学函数

    ABS - 返回数字的绝对值。
    CEIL - 返回大于或等于输入数字的最小整数值。
    FLOOR - 返回不大于参数的最大整数值。
    MOD - 返回数字的余数除以另一个。
    ROUND  - 将数字四舍五入到指定的小数位数。
    TRUNCATE - 将数字截断为指定的小数位数。
