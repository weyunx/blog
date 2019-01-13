[![Build Status](https://travis-ci.org/qiubaiying/qiubaiying.github.io.svg?branch=master)](https://travis-ci.org/qiubaiying/qiubaiying.github.io)[![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/home-assistant/home-assistant-iOS/blob/master/LICENSE)

# 微云的技术博客
博客的搭建修改自 [Hux](https://github.com/Huxpro/huxpro.github.io) 。

### [查看博客戳这里 👆](http://weyunx.com)



## Tips

 [Hux](https://github.com/Huxpro/huxpro.github.io) 已经写了详细的部署说明，我就不再赘述。本人在搭建的过程中碰到了不少问题，原文中没有提到，我在这里说明一下。

1. 如何去掉标题前面的 # 。

查看 _layouts/post.html

````js
<script>
    async("//cdnjs.cloudflare.com/ajax/libs/anchor-js/1.1.1/anchor.min.js",function(){
        anchors.options = {
          visible: 'always',
          placement: 'right',
          icon: '#'
         };
        anchors.add().remove('.intro-header h1').remove('.subheading').remove('.sidebar-container h5');
    })
</script>
````

修改其中的anchors.options即可，具体的配置参见[官网说明](https://www.bryanbraun.com/anchorjs/)。

2. 如何关闭评论插件。

查看 _config.yml

````yaml
# Disqus settings
disqus_username:
````

将插件配置的用户名删掉即可。评论插件也可以替换为Gittalk，可以参考 [BY](http://qiubaiying.github.io) 。

3. 替换了favicon.ico 浏览器不生效。

查看 _includes/head.html

````html
<!-- Favicon -->
<link rel="shortcut icon" href="{{ site.baseurl }}/img/favicon.ico?">
````

在 favicon.ico 后加上问号。

