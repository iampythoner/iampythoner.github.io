---
layout: cnblog_post
title:  "pyenv_forIDE"
permalink: '/misc/pyenv_forIDE'
date:   2018-01-21 08:34:39
categories: misc
---


```
# Python IDE
https://github.com/jorgenschaefer/elpy

# jedi json-rpc service_factory flake8 pytest autoflake hy

pyenv deactivate
brew install pyenv
brew install pyenv-virtualenv
vim ~/.bash_profile
	eval "$(pyenv init -)"
	eval "$(pyenv virtualenv-init -)"
v=3.6.0|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
v=2.7.10|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
pyenv virtualenv 2.7.10 pyenv2_flask
pyenv activate pyenv2_flask
pip install flask
pip install -i https://pypi.doubanio.com/simple jedi flake8


$ pip install jedi
Collecting jedi
  Using cached jedi-0.11.1-py2.py3-none-any.whl
Collecting parso==0.1.1 (from jedi)
  Using cached parso-0.1.1-py2.py3-none-any.whl
Installing collected packages: parso, jedi
Successfully installed jedi-0.11.1 parso-0.1.1
(pyenv2_flask)



有PEP警告
interview/just.py|11 col 1 warning| E305 expected 2 blank lines after class or function definition, found 1

$ pip install flake8
Collecting flake8
  Using cached flake8-3.5.0-py2.py3-none-any.whl
Collecting pycodestyle<2.4.0,>=2.0.0 (from flake8)
  Using cached pycodestyle-2.3.1-py2.py3-none-any.whl
Collecting enum34; python_version < "3.4" (from flake8)
  Using cached enum34-1.1.6-py2-none-any.whl
Collecting configparser; python_version < "3.2" (from flake8)
  Using cached configparser-3.5.0.tar.gz
Collecting mccabe<0.7.0,>=0.6.0 (from flake8)
  Using cached mccabe-0.6.1-py2.py3-none-any.whl
Collecting pyflakes<1.7.0,>=1.5.0 (from flake8)
  Using cached pyflakes-1.6.0-py2.py3-none-any.whl
Building wheels for collected packages: configparser
  Running setup.py bdist_wheel for configparser ... done
  Stored in directory: /Users/mike/Library/Caches/pip/wheels/1c/bd/b4/277af3f6c40645661b4cd1c21df26aca0f2e1e9714a1d4cda8
Successfully built configparser
Installing collected packages: pycodestyle, enum34, configparser, mccabe, pyflakes, flake8
Successfully installed configparser-3.5.0 enum34-1.1.6 flake8-3.5.0 mccabe-0.6.1 pycodestyle-2.3.1 pyflakes-1.6.0
(pyenv2_flask)


$ pip install flask
Collecting flask
  Using cached Flask-0.12.2-py2.py3-none-any.whl
Collecting itsdangerous>=0.21 (from flask)
Collecting Werkzeug>=0.7 (from flask)
  Using cached Werkzeug-0.14.1-py2.py3-none-any.whl
Collecting Jinja2>=2.4 (from flask)
  Using cached Jinja2-2.10-py2.py3-none-any.whl
Collecting click>=2.0 (from flask)
  Using cached click-6.7-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask)
Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, Jinja2, click, flask
Successfully installed Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 flask-0.12.2 itsdangerous-0.24
(pyenv2_flask)
```

