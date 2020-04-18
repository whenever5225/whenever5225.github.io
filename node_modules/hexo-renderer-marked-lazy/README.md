# hexo-renderer-marked-lazy

This is  a hexo renderer based on [hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked).

The only difference is that `marked-lazy` supports simple lazy-loading images. The renderer will transform `![some desc](http://img.com/jpg)` into something like:

```html
<img src="placeholder.gif" data-echo="http://img.com/jpg" alt="some desc">
```

As above implies, `data-echo` is the default "lazy src" attribute name, and its value is the real image source. The name `echo` come from [echo](https://github.com/toddmotto/echo/).

In your `_config.yml`, add your custom attribute as following:

```yaml
marked:
    lazyAttr: data-src
    blankSrc: /img/placeholder.png
    # for more options
    # please refer to https://github.com/hexojs/hexo-renderer-marked
```


Then you can add some lazy-loading plugins to your pages.

Enjoy it!
