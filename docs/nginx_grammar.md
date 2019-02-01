# nginx配置文件的通用语法

- 配置文件由指令与指令块构成
- 每条指令以`;`分号结尾，指令与参数间以空格符号分隔
- 指令块以`{}`大括号将多余的指令组织在一起
- `include`语句允许组合多个配置文件以提高可维护性
- 使用`#`符号添加注释，提高可读性
- 使用`$`符号使用变量
- 部分指令的参数支持正则表达式
