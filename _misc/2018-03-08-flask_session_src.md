---
layout: cnblog_post
title:  "flask_session_src"
permalink: '/misc/flask_session_src'
date:   2018-03-08 07:34:39
categories: misc
---

session处理流程是：
任何一个新的请求(flask.app.Flask.open_session() 调用self.session_interface.open_session) ---> 
open_session创建新的session --->
ctx push session --->
views处理request ---->
ctx pop session --->
save_session(flask.app.Flask.open_session() 调用self.session_interface.save_session)

flask session 使用redis backend cache 之后，
flask_session.sessions.RedisSessionInterface 的save_session方法默认实现是这样的

```
def save_session(self, app, session, response):
    domain = self.get_cookie_domain(app)
    path = self.get_cookie_path(app)
    if not session:
        if session.modified:
            self.redis.delete(self.key_prefix + session.sid)
            response.delete_cookie(app.session_cookie_name,
                                   domain=domain, path=path)
        return

    # if not self.should_set_cookie(app, session):
    #    return

    httponly = self.get_cookie_httponly(app)
    secure = self.get_cookie_secure(app)
    expires = self.get_expiration_time(app, session)
    val = self.serializer.dumps(dict(session))
    self.redis.setex(name=self.key_prefix + session.sid, value=val,
                     time=total_seconds(app.permanent_session_lifetime))
    if self.use_signer:
        session_id = self._get_signer(app).sign(want_bytes(session.sid))
    else:
        session_id = session.sid
    response.set_cookie(app.session_cookie_name, session_id,
                        expires=expires, httponly=httponly,
                        domain=domain, path=path, secure=secure)
```

在save_session 中只要session是存在的都会：
①存到redis中
self.redis.setex(name=self.key_prefix + session.sid, value=val,
                         time=total_seconds(app.permanent_session_lifetime))
②响应设置cookie
response.set_cookie(app.session_cookie_name, session_id,
                            expires=expires, httponly=httponly,
                            domain=domain, path=path, secure=secure)

将注释打开之后，多了两层判断， should_set_cookie()实现，在flask.sessions.SessionInterface中

```
def should_set_cookie(self, app, session):
    """Indicates whether a cookie should be set now or not.  This is
    used by session backends to figure out if they should emit a
    set-cookie header or not.  The default behavior is controlled by
    the ``SESSION_REFRESH_EACH_REQUEST`` config variable.  If
    it's set to ``False`` then a cookie is only set if the session is
    modified, if set to ``True`` it's always set if the session is
    permanent.

    This check is usually skipped if sessions get deleted.

    .. versionadded:: 0.11
    """
    if session.modified:
        return True
    save_each = app.config['SESSION_REFRESH_EACH_REQUEST']
    return save_each and session.permanent
```

①如果session被修改了，直接返回True，也就是说return执行不到，这样代码继续下去就会将session存到redis并对response设置Set-Cookie
②在没有修改的情况下，如果app.config设置了SESSION_REFRESH_EACH_REQUEST=True，并且 session.permanent=True 即可持久化，这样程序才会继续：在redis存入和将request
这两个条件同时满足的时候才会存入redis和 设置Set-Cookie

另外，如果request处理逻辑中，将session clear了，那么就会走save_session()上半部分的分支

```
if not session:
    if session.modified:
        self.redis.delete(self.key_prefix + session.sid)
        response.delete_cookie(app.session_cookie_name,
                               domain=domain, path=path)
    return
```

直接在redis中将session删除了，并且响应中删除session_id 对应的cookie