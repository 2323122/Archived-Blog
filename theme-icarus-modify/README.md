

> 以下是邮件提到内容的解释, 需要改动的部分都附上了需要改动的源代码路径, 你可以按需修改
>
> 文件中修改源码的部分我应该附上了注释, 方便对比, 不过我忘啦注释全不全

---

### 三栏变两栏布局

原版的主页和文章页都使用三栏布局, 在文章页阅读会显得内容很窄, 尤其是代码部分, 需要左右滚动, 故修改文章页为两栏布局

改动文件如下: 

- `themes/hexo-theme-icarus/includes/helpers/layout.js`
- `themes/hexo-theme-icarus/layout/common/widget.ejs`
- `themes/hexo-theme-icarus/layout/layout.ejs`
- `themes/hexo-theme-icarus/source/css/style.styl`




### 文章目录

#### 开启文章目录

icarus主题本身支持文章目录显示, 只需在文章的md文件中加入`toc: true`即可

这里我为了避免每个md文件都去写`toc: true`, 做到每篇文章默认就开启目录

改动文件如下:

- `themes/hexo-theme-icarus/includes/helpers/config.js`

#### 目录黏性布局

默认目录在滚动文章时如果太长会显示不全, 所以增加目录粘性布局

改动文件如下: 

- `themes/hexo-theme-icarus/layout/widget/toc.ejs`



