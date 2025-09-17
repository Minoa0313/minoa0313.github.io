---
title: tree-sitter 升级到0.25之后so文件的编译方法
date: 2025-09-17
draft: false
tags: 
  - python
categories:
---

# 旧版本写法
之前用的是0.24版本的
```python
GRAMMAR_DIR = 'tree-sitter-c'
LIB_PATH = 'build/my-languages.so'

# 言語ライブラリのビルド（一回のみ実施）
if not os.path.exists(LIB_PATH):
    print("Building...")
    os.makedirs('build', exist_ok=True)
    Language.build_library(
        LIB_PATH,
        [os.path.abspath(GRAMMAR_DIR)]
    )

C_LANGUAGE = Language(LIB_PATH, 'c')
parser = Parser()
parser.set_language(C_LANGUAGE)
```

# 新版本写法
升级到0.25之后这样的写法就会出错，更新了写法，好像也应该更新.so文件  
```python
LIB_PATH = 'build/tree-sitter-c.so'
lib = ctypes.cdll.LoadLibrary(LIB_PATH)
lib.tree_sitter_c.restype = ctypes.c_void_p
c_ptr = lib.tree_sitter_c()
ptr_value = ctypes.cast(c_ptr, ctypes.c_void_p).value

C_LANGUAGE = Language(ptr_value)
parser = Parser(C_LANGUAGE)
```

# 新版本so文件的编译方法
so文件的编译方法如下，需要下载[MSYS](https://www.msys2.org/#installation)  
打开`MSYS2 MINGW64` 的软件  
需要确认已安装 Node.js, npm, gcc  
```bash
node -v
npm -v
gcc -v
```

进入到tree-sitter-c文件夹目录  
依次执行以下命令  
1. 生成解析器源文件，这一步会在 src/ 目录下生成或更新 parser.c（以及可能存在的 scanner.c）  
   `$ tree-sitter generate`
2. 编译生成新的 .so 文件  
   `$ gcc -shared -o ../build/tree-sitter-c.so -fPIC src/parser.c`

执行完成后会在 build 文件夹下面生成一个名为 `tree-sitter-c.so` 的文件。    
这样就可以在代码中使用这个.so 文件了。
