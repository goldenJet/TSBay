## 基本操作

#### 运行 web 程序：

2.7 版本：`python -m SimpleHTTPServer 3000`

3.x 版本：`python -m http.server 3000`



## 安装相关

#### pip源设置：

中国科技大学：https://pypi.mirrors.ustc.edu.cn/simple/   
清华园：https://pypi.tuna.tsinghua.edu.cn/simple/


**临时使用**：  

`pip install pandas -i https://pypi.tuna.tsinghua.edu.cn/simple/`

**永久修改**：  

1. linux: 修改或新建文件 ~/.pip/pip.conf  
2. windows： 在user\pip目录下创建或新建文件，如：C:\Users\xx\pip，新建文件pip.ini

**文件内键入**：

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```


#### 解决 Windows Python 3.5 “from Crypto.Cipher import AES” 安装失败问题

**报错信息**：

```
from Crypto.Cipher import AES
ModuleNotFoundError: No module named 'Crypto'
```

**解决方法**:

- step1: `pip install pycryptodome`

- step2: 然后将 D:\develop\Python\Python37-64\Lib\site-packages 路径下的 crypto 目录的 c 改为 C (大写) 即可。