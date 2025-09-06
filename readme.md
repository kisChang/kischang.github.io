1. 安装
   npm install -g hexo-cli
2. 运行
   hexo s
3. 发布
   hexo clean   #清除缓存文件 db.json 和已生成的静态文件 public
   hexo generate   #生成网站静态文件到默认设置的 public 文件夹(hexo generate 的缩写)
   hexo deploy   #自动生成网站静态文件，并部署到设定的仓库(hexo deploy 的缩写)

创建文章
hexo new “我的第一篇文章”
