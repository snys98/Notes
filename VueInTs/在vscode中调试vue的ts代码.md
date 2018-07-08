# 启动vue的ts调试 #

默认的vue.config.js配置如下:

``` js
{
  mode: 'development',
  context: 'D:\\workspace\\_极客格子\\Geexbox\\Geexbox.YourPen\\Geexbox.YourPen\\clientapp',
  // 看这里!!!!!!!!!!!!!!!!!!!!!!
  devtool: 'cheap-module-eval-source-map',
  node: {
    setImmediate: false,
    process: 'mock',
    dgram: 'empty',
    fs: 'empty',
    net: 'empty',
    tls: 'empty',
    child_process: 'empty'
  },
  output: {
    path: 'D:\\workspace\\_极客格子\\Geexbox\\Geexbox.YourPen\\Geexbox.YourPen\\clientapp\\dist',
    filename: '[name].js',
    publicPath: '/'
  }
  // 以下省略很多行
}
```

其中devtool的值为`cheap-module-eval-source-map`,我们需要将这个值覆写为`source-map`,才能在vs中正常进行ts的调试

如何值的修改,参见[覆写vue.config.js的全局配置.md]