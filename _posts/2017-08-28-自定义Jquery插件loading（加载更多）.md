---
layout: post
title: [自定义Jquery插件loading（加载更多）]
categories: [Javascript]
fullview: false
---
因项目需要自定义一个jquery插件，现将自定义jquery插件心得记录下来，以便加深印象。如下是自定义jquery插件的几个简单步骤。

1.编写自运行函数，结构如下
(function($,undefined){ //code; })(jQuery);

这里将jQuery函数体当作参数传入自运行函数中，并通过$变量接收，之后可以在自运行函数中直接使用$变量操作，达到与外界Js隔离，防止与其他js插件的$变量冲突。

undefined参数，不会接收实际入参，值为window.undefined。这样做可以防止window.undefined变量被外界重写，导致在jquery插件内错误使用。

2.在自运行函数中通过jQuery的extend方法定义插件，如下

[](https://code.jquery.com/jquery-1.0.js)
(function($,undefined){ $.fn.extend(loading:function(){ //code; }); })(jQuery);

$.fn.extend是扩展jQuery对象方法，$.extend是扩展jQuery类方法。对象方法访问方式为$("/#id").loading();类方法访问方式为$.loading()。如果学过java相关编程应该比较好理解。。。

3.之后就是编写自己插件的对应功能代码

编写jquery插件时，尽量返回this，或者this.each，以便继承jquery的链式操作。如下是我的loading插件，它实现了<加载更多>的功能。代码地址：[https://github.com/Alan3058/jquery-plugin/tree/master/loading](https://github.com/Alan3058/jquery-plugin/tree/master/loading)

(function($,undefined){ var Loading = function(element,options){ this.$element = element; var defaultPageSettings={pageNum:0,pageSize:5,pageName:"",pageNumName:"pageNum",pageSizeName:"pageSize"}; options.page = $.extend({},defaultPageSettings,options.page); this.settings = $.extend({},{registryEvent:"click",container:"",bootLoad:true},options); }; Loading.prototype = { loadMore:function(){ this.settings.page.pageNum += 1; var page = this.getPage(); var ajaxSettings = this.settings.ajax; ajaxSettings = $.extend({},ajaxSettings,{"data":page}); var container=this.getContainer(); if(container!=""){ ajaxSettings = $.extend({},ajaxSettings,{"success":function(data){ $(container).append(data); }}); } $.ajax(ajaxSettings); }, getContainer:function(){ return this.settings.container; }, getPage:function(){ var $page = this.settings.page; var page={}; page[$page.pageSizeName]=$page.pageSize; page[$page.pageNumName]=$page.pageNum; if(!$page.pageName || $page.pageName == ""){ return page; } return ({}[$page.pageName]=page); }, getRegistryEvent:function(){ return this.settings.registryEvent; } }; $.fn.extend({ loading : function(options){ var load = new Loading(this,options); this.bind(load.getRegistryEvent(),function(){ load.loadMore(); }); if(load.settings.bootLoad){ load.loadMore(); } return this; } }); })(jQuery);

参考jquery源码[https://code.jquery.com/jquery-1.0.js](https://code.jquery.com/jquery-1.0.js)
