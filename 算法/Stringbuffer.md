StringBuilder
===

如果需要用许多小段的字符串构建一个字符串，则需要用到StringBuilder类
StringBuilder builder = new StringBuilder();
builder.append(c);
builder.append(str);
String completedString = builder.toString

StringBuilder append(String str)
追加一个字符串并返回this

StringBuilder append(char c)
追加一个代码单元并返回this

StringBuilder appendCodePoint(int cp)
追加一个代码点，并将其转换为一个或两个代码待援并返回this

void setCharAt(int i, char c)
将第i个代码单元设置为c

StringBuilder insert(int offset, String str)
在offset位置插入一个字符串并返回this

StringBuilder delete(int startIndex, int endIndex)
删除偏移量从startIndex到endIndex-1的代码单元并返回this

    StringBuilder().deleteCharAt()

String toString()
返回一个与构建器或缓冲器内容相同的字符串
