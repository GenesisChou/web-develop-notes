```javascript
var express=require('express');
var app=express();
var bodyParser=require('body-parser'); //解析post数据
app.use(bodyParser.json());
app.use(express.static(path.resolve(__dirname,'public')))
app.set('views',path.resolve(__dirname,'views'));
app.set('view engine','ejs');//设置模版引擎 
app.use('/',(req,res)=>{
    res.render('index',{name1:value1,name2:value2});
});
app.listen(3000,()=>{
    console.log('server is listening http://localhost:3000')
});
```