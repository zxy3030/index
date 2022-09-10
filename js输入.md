#### Input

```javascript
//javascript v8
var line = readline()
while (line = readline()) {
    let inputs = line.split(' ');
    let a = parseInt(inputs[0]);
    let b = parseInt(inputs[1]);
    function add(a,b) {
        return a+b
    }
     console.log(add(a,b));
}
```

#### Array

from()

参数：①一个类数组对象（任何可迭代结构），②映射函数（数组中每个元素要调用的函数,可选），③指定映射函数中this的值（此值在箭头函数中不适用）
返回：一个新的数组实例
