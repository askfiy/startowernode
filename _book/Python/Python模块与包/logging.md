# logging简介

Python内置模块logging提供了日志记录相关功能，是一款非常强大且常用的模块。

[官方文档](https://docs.python.org/zh-cn/3.6/library/logging.html)

它的使用如果刨根问底可能比较复杂，所以里仅介绍1种最方便的使用方式，其他的使用方式日常开发中基本不会用到，故不进行介绍。

# 简单了解

logging模块中规定日志拥有6个级别，每个级别都有单词、数字2种表现形式，如下表所示：

| level            | number | 描述     |
| ---------------- | ------ | -------- |
| logging.critical | 50     | 致命错误 |
| logging.error    | 40     | 常规错误 |
| logging.warning  | 30     | 警告信息 |
| logging.info     | 20     | 普通信息 |
| logging.debug    | 10     | 调试信息 |
| logging.NOSET    | 0      | ...      |

logging.NOSET一般不会进行使用，所以你也可以认为logging的日志级别只有5个。

等级越低，越能看到更多的日志信息，它会根据等级依次向上推进，如下图所示：

![image-20210528094406877](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210528094406877.png)

默认的等级是30，也就是warning级别，仅能看到critical、error、warning这个级别的日志，而info和debug则看不见。

如下所示：

```
import logging

logging.debug("debug")
logging.info("info..")
logging.warning("warning..")
logging.error("error..")
logging.critical("critical..")

# ❶ ❷
# WARNING:root:warning..
# ERROR:root:error..
# CRITICAL:root:critical..
```

❶：root指的是记录日志的用户，默认就是root

❷：默认的日志输出位置是向屏幕，也可以将日志输出至文件



# 配置文件

如果想快速的使用logging模块进行日志记录。

可以使用配置文件的形式，一般来说该配置文件会存放至settings.py中：

```
import os

# 定义2种日志记录格式
# 标准的：standard_format
# 简单的：simple_format
standard_format = "[%(asctime)s][%(threadName)s:%(thread)d][username:%(name)s][%(filename)s:%(lineno)d]" \
                  "[%(levelname)s][%(message)s]"

simple_format = "[%(levelname)s][%(asctime)s][%(filename)s:%(lineno)d]%(message)s"

# 定义日志存放路径
LOG_DIR = os.path.dirname(os.path.abspath(__file__))
if not os.path.isdir(os.path.join(LOG_DIR, "log")):
    os.mkdir(os.path.join(LOG_DIR, "log"))
LOG_PATH = os.path.join(LOG_DIR, "log", "run.log")

# LOGGING的应用配置字典，无需太大改动，开箱即用
LOGGING_SETTING = {
    "version": 1,
    "disable_existing_loggers": False,
    # 添加2种日志记录格式
    "formatters": {
        "standard": {
            "format": standard_format
        },
        "simple": {
            "format": simple_format
        },
    },
    # 控制流相关配置
    "handlers": {
        # 输出到终端(logging.StreamHandler)，采用简单的日志记录格式
        "screen": {
            "level": "DEBUG",
            "class": "logging.StreamHandler",
            "formatter": "simple"
        },
        # 输出到文件(logging.handlers.RotatingFileHandler)，采用标准的日志记录格式
        "file": {
            "level": "DEBUG",
            "class": "logging.handlers.RotatingFileHandler",
            "formatter": "standard",
            # 日志文件位置及名称，若不指定则默认在当前目录下
            "filename": LOG_PATH,
            # 每个日志文件最大5M，当存在5个日志文件后开启日志轮转
            "maxBytes": 1024*1024*5,
            "backupCount": 5,
            "encoding": "utf-8",
        },
    },
    # 定义不同用户采用的控制流
    "loggers": {
        # 如果不指定用户，或指定用户未在loggers字典中，则采用该配置
        "": {
            "handlers": ["screen", "file"],
            # 以控制流相关配置中过滤级别为准，这里是1次过滤，控制流中是2次过滤
            "level": "DEBUG",
            # 关闭日志冒泡，切勿手动更改
            "propagate": False,
        },
        # 若指定用户为testUser，则采用该配置
        "testUser": {
            "handlers": ["screen"],
            "level": "DEBUG",
            "propagate": False,
        }
    }
}
```



# 项目应用

如何使用该配置文件？我们假设项目目录如下：

```
Project/
|-- bin/
|   |-- run.py    # 启动脚本
|
|-- view/
|   |-- main.py   # 主程序
|
|-- common.py     # 公用模块
|-- settings.py   # logging配置文件
```

首先是启动脚本：

```
# bin/run.py

import os
import sys

sys.path.append(
    os.path.dirname(os.path.dirname(__file__))
)

from view.main import main

if __name__ == "__main__":
    main()
```

其次是主程序：

```
# view/main.py

from common import logger

def main():
    logger.debug("debug")
    logger.info("info..")
    logger.warning("warning..")
    logger.error("error..")
    logger.critical("critical..")
```

最后是公用模块：

```
# common.py

from logging import config
from logging import getLogger
from settings import LOGGING_SETTING

config.dictConfig(LOGGING_SETTING)
logger = getLogger("adminstartion")
```

可以看到在公用模块中导入了配置字典，并且将它应用进了logging模块。

然后获取了一个日志对象logger，在以后使用时都使用这个logger进行日志记录即可，这里getLogger()的用户名是adminstartion，未定义在LOGGIN_SETTING的loggers中，故会采用第一个配置，也就是下面这个：

```
"loggers": {
        # 如果不指定用户，或指定用户未在loggers字典中，则采用该配置
        "": {
            "handlers": ["screen", "file"],
            "level": "DEBUG",
            "propagate": False,
        },
        ...
        }
```

