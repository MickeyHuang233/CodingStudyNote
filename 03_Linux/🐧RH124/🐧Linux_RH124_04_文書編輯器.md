# vi和vim
參考：
- [vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php)
- [vi/vim快捷鍵一覽圖](https://kknews.cc/code/m22jz42.html)

## 🐧常見模式
![vi三種模式的相互關係](http://linux.vbird.org/linux_basic/0310vi//centos7_vi-mode.gif)
- 正常模式
- 編輯模式
- 命令行模式

## 🐧常用指令
| command            | description             |
| ------------------ | ----------------------- |
| i                  | 遊標==前==方插入字元    |
| a                  | 遊標==後==方插入字元    |
| o                  | 遊標==上==方插入空白行  |
| Shift O            | 遊標==下==方插入空白行  |
| del鍵              | 刪除一個字元            |
| dd<br/>3dd         | 刪除一行<br/>刪除三行   |
| yy<br/>3yy         | 複製<br/>複製三行       |
| p                  | 遊標==下==方貼上        |
| Shift P            | 遊標==上==方貼上        |
| `:1`<br/>gg        | 遊標定位至第一行        |
| 10 按Shift + g     | 移到文檔第10行          |
| `:$`<br/>Shift + g | 遊標定位至最後一行      |
| Ctrl + g           | 顯示檔案 行 和列 的位置 |
| u                  | 上一步                  |
| ctrl + r           | 下一步                  |
| /string            | 搜索字串                |
| n                  | 向下找                  |
| Shift + n          | 向上找                  |
| `:set number`      | 顯示行數                |
| `:set nonu`        | 取消顯示行數            |
| `:w`               | 存檔                    |
| `:wq!`             | 存檔後離開              |
| `:q!`              | 不存檔離開              |

## 🐧分屏指令
| command | description  |
| ------- | ------------ |
| `:sp`   | 橫向增加分屏 |
| `:vsp`  | 縱向增加分屏 |

- `ctrl + w`，切換分屏窗口，再來使用以下指令做出相應的操作

| command | English | description                                |
| ------- | ------- | ------------------------------------------ |
| w       | window  | 切換到下一個窗口                           |
| r       | reverse | 互換窗口                                   |
| c       | close   | 關閉當前窗口，但不能關閉最後一個窗口       |
| q       | quit    | 退出當前窗口，退出最後一個窗口，相當退出vi |
| o       | other   | 關閉其他窗口                               |
