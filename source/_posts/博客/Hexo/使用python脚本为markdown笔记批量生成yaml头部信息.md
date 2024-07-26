---
title: 使用python脚本为markdown笔记批量生成yaml头部信息
date: 2024-7-25 23:11 
tags: 
  - Hexo
  - Python
categories:
  - [博客, Hexo]
---

为了快速的将其他地方Markdown笔记导入到Hexo中，使用Python脚本为Markdown笔记批量生成YAML头部信息


### 整理笔记目录

比如已经规划好了目录，如：
```

├─开发
│  ├─java
│  │      Java入门.md
│  │      Java数据结构.md 
│  │      xxxx.md
│  │
│  ├─Spring
│  │      Spring MVC.md
│  │      Spring AOP.md
│  │      Spring Data.md
│  │
│  └─网络
│          协议.md
│          网络通信.md
```

### 生成YAML头部信息要求

目标是为将目录和下级目录中全部的Markdown文件自动生成YAML头部信息，如在文件：  
**.\博客\Hexo\Hexo安装使用.md**  
的头部添加：
```
---
title: Hexo安装使用
date: 2024-07-24 23:31
tags: 
  - Hexo
categories:
  - [博客, Hexo]
---
```

要求：
- title：为文件名称，不要后缀
- date：随机时间（小于当前时间）
- tags: 为最近一级的目录名称
- categories：为多级目录名称

----
### Python脚本

创建Python脚本文件：**add_front_matter.py**   
内容如下：
```
import os
import random
import datetime

def get_random_date():
    now = datetime.datetime.now()
    random_days = random.randint(1, 365 * 5)  # 过去五年中的随机天数
    random_date = now - datetime.timedelta(days=random_days)
    return random_date.strftime('%Y-%m-%d %H:%M')

def add_front_matter(directory):
    for root, _, files in os.walk(directory):
        print(f"Scanning directory: {root}")
        for file in files:
            if file.endswith('.md'):
                file_path = os.path.join(root, file)
                print(f"Found markdown file: {file_path}")

                try:
                    with open(file_path, 'r', encoding='utf-8') as f:
                        content = f.read()
                    
                    title = os.path.splitext(file)[0]
                    date = get_random_date()
                    tags = os.path.basename(root)
                    categories = os.path.relpath(root, directory).split(os.sep)

                    front_matter = f'''---
title: {title}
date: {date}
tags: 
  - {tags}
categories:
  - [{', '.join(categories)}]
---

'''

                    new_content = front_matter + content
                    with open(file_path, 'w', encoding='utf-8') as f:
                        f.write(new_content)

                    print(f"Updated: {file_path}")
                except Exception as e:
                    print(f"Error updating {file_path}: {e}")

if __name__ == "__main__":
    directory = r"D:\develop\blog\hexo-blog\source\_posts"  # 替换为你的Markdown文件所在目录
    print(f"Starting to update markdown files in: {directory}")
    add_front_matter(directory)
    print("Finished updating markdown files.")

```

### 执行脚本更新文件

**注意：将上面脚本中的Markdown文件所在目录替换为你的整理后的笔记目录**


打开终端或命令提示符，导航到脚本所在目录，运行脚本：
```
python add_front_matter.py

```

> 问题： 时间不好处理，需要自己修改