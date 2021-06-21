# tech-notes
快速技术笔记，短小精悍

## 为什么 UTF-8 不存在字节序问题？
UTF-8 是可变长度的编码格式，采用 1~4 字节表示一个 Unicode 字符。理论上在大于 1 个字节的表示时应该需要确定字节序，但是由于 UTF-8 有其固定的编码规则，这个规则恰好确定了多字节表示时的字节顺序，也就没有字节序的问题了。UTF-8 的编码规则如下：

- 1 字节表示时，格式为 `0xxxxxxx`
- 2 字节表示时，格式为 `110xxxxx 10xxxxxx`
- 3 字节表示时，格式为 `1110xxxx 10xxxxxx 10xxxxxx`
- 4 字节表示时，格式为 `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx`

可以看到，UTF-8 编码在不同字节数表示下，其中都有独特确定的比特位，如 `110`、`1110`、`11110`，而这刚好确定了多字节下的字节顺序。比如当遇到 `11100100 10111010 10111010` 这么一个 3 字节的 UTF-8 编码数据，那么根据规则，`1110` 开头的就是须先读的字节头，于是就确定了顺序得是从高读到低。或者也可以理解成 UTF-8 在多字节时其实固定了是 BE（Big-Endian）表示。

所以如果 `11100100 10111010 10111010` 先读低位的 `10111010` 那么它就不是一个有效的 UTF-8 数据了，因为不符合其编码规则。

- `11100100 10111010 10111010` 转换成十六进制即 `E4BABA`，对应 UTF-8 中表示汉字「人」。
- `10111010 10111010 11100100` 转换成十六进制即 `BABAE4`，不是有效的 UTF-8 编码。

## 控制字符集（Control Character）中的 Delete 符
[控制字符](https://en.wikipedia.org/wiki/Control_character)是一系列的非打印字符，主要用来作控制功能的用途。广泛使用但极其有限的 [ASCII 编码中共定义了 33 个字符](https://zh.wikipedia.org/wiki/ASCII#Control_characters)作为控制字符，这 33 个字符在 [ISO/IEC 2022](https://zh.wikipedia.org/wiki/ISO/IEC_2022) 标准中被重新划分，ASCII 中的 0x00-0x1F（共 30 个）被划入 [C0 控制字符集](https://en.wikipedia.org/wiki/C0_and_C1_control_codes#C0_controls)，剩下的 2 个分别是 0x20 即空格符` `和 0x7F 即 Del 符 `(delete)`，0x20 空格符其实是可打印字符（只是单独打印出来不能明显辨别）所以更新的一些标准没有把其作为控制字符。0x7F 即 Del 符是不可打印字符，在更新的一些标准中它仍是一个控制字符，只是在 ISO/IEC 2022 标准中没有划分其为 C0/C1，C0/C1 只是整个控制字符集中的两个子集。

Del 符在 Unicode 万国码中的表示是 U+007F。[Del 符](https://zh.wikipedia.org/wiki/Delete%E5%AD%97%E7%AC%A6)在古早的打孔卡打字时期是用来表示一个字符作废用的（语义同 HTML 的[`<del></del>`](https://webapps.stackexchange.com/a/15030)），因为其 ASCII 编码 0x7F 即二进制 `1111111` 刚好表示 7 个孔，于是打孔卡打错字想删除，将其改为 7 个孔就表示作废了。在现代计算机的打字环境下，跟 Del 符相等语义的使用已经不存在了，因为现代用户打错作废时都是直接使用退格键将错字直接删除不保留，计算机不像打孔卡那样不可“撤销”。所以现今计算机中 Del 符要么是无法打出来，要么是不使用（即打出来没效果），又或者给赋予了新的语义。前两者无实际意义不谈，赋予新的语义可以谈谈。

Del 符在 Windows 10 系统下被赋予了一个新的语义：[**删除一个词**](https://en.wikipedia.org/wiki/Delete_character#Current_use)。通常情况下打错字词时我们按退格键 <kbd>backspace</kbd> 都是一个一个字符的删除的。也就是说对于 `I makr a typo (光标)` 例子而言，目前光标落在 `typo` 后面一空格后的位置，此时我们按下一次 <kbd>backspace</kbd>，只会退一格删除 `typo` 后面的空格符` `。再按下一次 <kbd>backspace</kbd>，只会再退一格删除 `typo` 的 `o` 字符。而如果我们按下的是 <kbd>ctrl</kbd> + <kbd>backspace</kbd> 组合，那么系统会发出一个 Del 符，此时会直接退格删除整个 `typo ` 单词和后面的一个空格，剩下 `I makr a (光标)`，也就是 Del 符的新语义*删除一个词*。这在想快速删除整个词的时候比按多次 <kbd>backspace</kbd> 效率更高。不过对于不分词的中文来说，其语义就有点不一致，变成了删除以标点符号分隔的整个分句。Windows 10 更新迭代后的 Notepad 也是支持 <kbd>ctrl</kbd> + <kbd>backspace</kbd> 组合的。


