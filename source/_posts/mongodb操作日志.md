---
title: mongodb操作日志
date: 2017-03-27 17:31:47
tags: [mongodb,nosql]
categories: 工作
---
```javascript
// 根据查询结果,替换某个字段的字符串

// 模糊查询vcPicUrl中有'http:'字段的结果
var list = db.getCollection('tbArticle').find({"vcPicUrl":/http:/});
// var index = 0;
// list.toArray();
// while(list.hasNext()){
//         var url = list[index].vcPicUrl;
//         print(url);
//         var rurl = url.replace('http:','https:');
//         print(rurl);
//         db.getCollection('tbArticle').update({"_id":list[index]._id}, {$set:{'vcPicUrl':rurl}});
//         index=index+1;
//     }
print(list.length());
// 将vcPicUrl中的http: 替换成 https:
 for (var i = 0; i < list.length(); i++) {
        var article = list[i];
        var url = article.vcPicUrl;
        var rurl = url.replace('http:','https:');
        db.getCollection('tbArticle').update({"_id":article._id}, {$set:{'vcPicUrl':rurl}});
     }
     print("success")
```