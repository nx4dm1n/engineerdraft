##删除操作

典型场景: 

	假设你有一个上万行的文件,需要你删除一段很长区域的文字, 场景的方式是利用 shift+v 来选中对应的行,然后删除, 这样做对于较短文件还行, 如果文件较长很容易误删除. 因此比较好的方式是找到要删除的开始行和结尾行的行号. 问题就降解为删除特定行之间的内容. 
    
vim 删除特定行之间的内容. 假设需要删除第2行和第10行之间的内容.

`2,10:d`

## 快速跳转到上次修改的地方
### 跳转上次修改的行
    
    '.
### 快速跳转上次修改的字符
    `.
    
#### 快速跳转上次修改的字符, 同时进入插入状态  

    gi