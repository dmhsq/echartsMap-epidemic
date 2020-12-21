# 疫情地图

在线地址<www.zczyp.top>

## Project setup安装依赖包
```
npm install
```

### Compiles and hot-reloads for development运行
```
npm run serve
```

### Compiles and minifies for production打包
```
npm run build
```

打包后的在dist中

需要配置nginx代理转发


```html
"/api": {
        target: "https://www.ncovchina.com/data",
   
 "/aki": {
        target: "https://api.inews.qq.com/newsqa/v1/automation/foreign/country",

```
