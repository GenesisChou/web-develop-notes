
```javascript
const fs=require('fs');
//异步状态
fs.stat(dir,(err,stats)=>{
    stats.isDirectory();
    stats.isFile();
})
fs.readdir(dir,(err,files)=>{
    console.log(files);
})
//同步
let stats=fs.statSync(dir);
stats.isDirectory();
stats.isFile();
let files=fs.readdirSync(dir);
```