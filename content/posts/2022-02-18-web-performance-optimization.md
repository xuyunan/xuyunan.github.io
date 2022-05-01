---
title: Web优化加载速度，优化打包大小
tags: [web]
date: 2021-02-18T13:00:24+08:00
---

## 按需引入组件库, BizCharts

直接引入方式 

```javascript
import { Chart, Interval, Tooltip } from "bizcharts";  
```

按需引入方式 

```javascript
import { TooltipItem } from "bizcharts/lib/interface";
import Chart from 'bizcharts/lib/components/Chart';
import Tooltip from 'bizcharts/lib/components/Tooltip';
import Axis from 'bizcharts/lib/components/Axis';
import Interval from 'bizcharts/lib/geometry/Interval';
```
由于项目中我只使用了柱状图和饼图两个组件, 所以大小由bizcharts 3.72M缩减至643KB. (其中没有计算data-set.js文件，data-set占619KB)

如果你也想分析下项目文件大小，也可以试试 [Source map explorer](https://create-react-app.dev/docs/analyzing-the-bundle-size/) 

## 关闭sourcemap功能

将package.json中的 "build": "react-scripts build",
替换成 "build": "GENERATE_SOURCEMAP=false react-scripts build",

这样就避免生成类似`2.feb6b4a0.chunk.js.map`的map文件，2.9M的代码，map文件近10.9M, 关闭sourcemap功能，将大大缩减页面加载时间.

## Nginx开启gzip压缩

开启之后，2.9M的代码，可压缩至796KB，压缩了近77%的大小

gzip配置示例如下

```nginx
location / {
    try_files $uri $uri/ /index.html;
    # ...
}
gzip on; 
gzip_vary on; 
gzip_min_length 1024; 
gzip_proxied expired no-cache no-store private auth; 
gzip_types text/plain text/css text/xml text/javascript application/javascript application/x-javascript application/xml; 
gzip_disable "MSIE [1-6]\.";
```

注意不要忘记在gzip_types添加application/javascript

如需更详细的配置，可至官网查看 [Module ngx_http_gzip_module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)

这里也有一篇比较详细的文章，可以参考 [如何在 Nginx 服务器中配置 GZip 压缩？
](http://www.yaohaixiao.com/blog/how-to-configure-gzip-compression-with-nginx/)


