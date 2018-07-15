---
layout: cnblog_post
title:  "code block"
permalink: '/misc/code_block'
date:   2018-07-07 07:34:39
categories: misc
---

```py
import logging
################################# log tools ####################################
BARE_LOG_FORMAT = '%(message)s'
TIME_LOG_FORMAT = '[%(asctime)s]: %(message)s'
TIME_LINENO_LOG_FORMAT = '%(asctime)s-[line:%(lineno)d]: %(message)s'
TIME_LINENO_LEVEL_LOG_FORMAT = '%(asctime)s-[line:%(lineno)d]-[%(levelname)s] : %(message)s'

def config_root_logger(level=logging.INFO, filename=None, filemode=None, format=None, datefmt=None, stream=None):
    dict_ = {}
    for arg_name, arg_val in locals().iteritems():
        if arg_val:
            dict_[arg_name] = arg_val
    logging.basicConfig(**dict_)

def logger_generator(level, name, handle_type, format_str=None):
    from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler
    import os
    FILE_NAME = os.path.join("./logs/", name)

    if not handle_type:
        handle_type = 'timed_rotating'
    file_handler = None
    if handle_type == 'rotating':
        file_handler = RotatingFileHandler(FILE_NAME, maxBytes=1024 * 1024 * 100, backupCount=10)
    elif handle_type == 'timed_rotating':
        file_handler = TimedRotatingFileHandler(FILE_NAME, 'midnight', 1)

    if not format_str:
        format_str = '%(asctime)s-%(filename)s-[line:%(lineno)d]-[%(levelname)s] : %(message)s'
    formatter = logging.Formatter(format_str)

    file_handler.setFormatter(formatter)
    file_handler.setLevel(level)
    logging.getLogger(name).addHandler(file_handler)
    return logging.getLogger(name)

config_root_logger()
logger = logger_generator(logging.INFO, os.path.splitext(str(__file__))[0], 'timed_rotating', TIME_LINENO_LEVEL_LOG_FORMAT)
############################### end log tools ##################################
```