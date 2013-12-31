---
layout: post
title: "fabric执行流程"
date: 2013-12-31 14:26
comments: true
categories: 
---

{% codeblock lang:py fabric main %}
parser, options, arguments = parse_options()

# Handle regular args vs -- args
arguments = parser.largs
remainder_arguments = parser.rargs
{% endcodeblock %}