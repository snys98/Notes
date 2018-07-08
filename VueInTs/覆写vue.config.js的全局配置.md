# 覆写vue.config.js的全局配置 #

## 获取vue.config.js(vue-cli的全局配置) ##

通过命令行`vue inspect > output.js`,可以导出当前的默认配置

``` js
{
  mode: 'development',
  context: 'D:\\workspace\\_极客格子\\Geexbox\\Geexbox.YourPen\\Geexbox.YourPen\\clientapp',
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

vue.config.js的全局配置在npm的vue-cli的安装目录,通过在项目根目录下面新建vue.config.js,可以覆写全局配置
*这点非常不直观,建议默认项目自动新建vue.config.js*

``` js
// vue.config.js
module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
    } else {
      // 这里修改生产环境的的配置修改,config对象和之前导出的js文件结构对应
      config.devtool = 'source-map';
    }
  }
}
```