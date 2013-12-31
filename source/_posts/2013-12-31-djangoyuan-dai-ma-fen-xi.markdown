---
layout: post
title: "django源代码分析"
date: 2013-12-31 14:44
comments: true
categories:
---

内建的local http server实现
-------------------------

执行入口即`manage.py runserver --settings=local`
通过`execute_from_command_line(sys.argv)`来调用对应的command

{% codeblock lang:py manage main mark:51%}
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "settings.production")

from django.core.management import execute_from_command_line
sys.argv.append("--traceback")

execute_from_command_line(sys.argv)
{% endcodeblock %}

`execute_from_command_line`会创建一个`ManagementUtility`对象，并调用它的
`fetch_command(subcommand).run_from_argv(self.argv)`

之后会进行各种参数判断和环境检查，例如是否*autoreload*等，最终进入真正的`runserver`的handler，即`django.core.management.commands.runserver.py`
http server启动流程
1. `get_handler`->`get_internal_wsgi_application`获取内建的wsgi application，通过`import_module`加载`settings.wsgi`模块，得到定义在`django.core.handlers.wsgi`中的`WSGIHandler`
2. 执行`django.core.server.basehttp.run`，创建一个`WSGIServer`(继承自python标准库的`HTTPServer`)，并最终启动。
{% codeblock %}
server_address = (addr, port)
if threading:
    httpd_cls = type(str('WSGIServer'), (socketserver.ThreadingMixIn, WSGIServer), {})
else:
    httpd_cls = WSGIServer
httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
httpd.set_app(wsgi_handler)
httpd.serve_forever()
{% endcodeblock %}

下面是wsgi handler的实现分析
------------------------------
`django.core.handlers.base.BaseHandler`
核心函数为`get_response`，根据request返回一个response
`get_response`的基本执行流程如下:

* 创建url解析器`urlresolvers`，django本身的url.conf中配置好了每个url对应的处理函数，例如
{% codeblock lang:py %}
urlpatterns = patterns('',
    url(r'^articles/2003/$', 'news.views.special_case_2003'),
    url(r'^articles/(\d{4})/$', 'news.views.year_archive'),
    url(r'^articles/(\d{4})/(\d{2})/$', 'news.views.month_archive'),
    url(r'^articles/(\d{4})/(\d{2})/(\d+)/$', 'news.views.article_detail'),
)
{% endcodeblock %}

* 首先尝试使用已经注册过的request middleware method来处理request，看是否能得到有效的response，若成功则直接进行后续处理
{% codeblock lang:py%}
response = None
# Apply request middleware
for middleware_method in self._request_middleware:
    response = middleware_method(request)
    if response:
        break
{% endcodeblock %}

* 若2没有得到有效response，则通过urlresolver获取callback, callbackargs，然后使用view middleware method来处理request，看是否能得到有效response；若仍失败则直接通过callback来处理request

* 在前面的过程中若出错则抛出异常或者退出

* 在得到有效response之后，通过response middleware来对response进行处理
{% codeblock lang:py%}
for middleware_method in self._response_middleware:
    response = middleware_method(request, response)
response = self.apply_response_fixes(request, response)
{% endcodeblock %}


**TO BE CONTINUE...**