---
title: 前端导出数据到excel文件
date: 2018-12-18 19:01:12
tags: 
- 学习笔记
categories: 
- 前端
---

## 背景：
![undefined](https://cdn.nlark.com/lark/0/2018/png/146541/1545117260764-7cad7831-0a93-4338-b76b-3a25c68be1b7.png) 
这是需求。。
请教静姐，找到一个库 -- xlsx.full.min.js，支持导出导入数据到csv或者excel文件。
alicdn：
```html
<script src="http://g.alicdn.com/cs70/luban-component/0.2.3/xlsx.full.min.js"></script>
```
## 使用方法：

### 数据格式：
![undefined](https://cdn.nlark.com/lark/0/2018/png/146541/1545117761507-630eb32a-37dc-46aa-bdf7-9baf1e2beefb.png) 

### 页面上添加a标签：
```html
<a href="" download="machine_info.xlsx" id="hf"></a>
```

### 导出过程：
```javascript
				function exportToExcel (dataIndex) {
                    
                    let jsono = processData(dataIndex); //这里的jsono就是处理好的数据
                    
                    var tmpDown; //导出的二进制对象
                    function downloadExl(json, type) {
                        var tmpdata = json[0];
                        json.unshift({});
                        var keyMap = []; //获取keys
                        //keyMap =Object.keys(json[0]);
                        for (var k in tmpdata) {
                            keyMap.push(k);
                            json[0][k] = k;
                        }
                      var tmpdata = [];//用来保存转换好的json 
                            json.map((v, i) => keyMap.map((k, j) => Object.assign({}, {
                                v: v[k],
                                position: (j > 25 ? getCharCol(j) : String.fromCharCode(65 + j)) + (i + 1)
                            }))).reduce((prev, next) => prev.concat(next)).forEach((v, i) => tmpdata[v.position] = {
                                v: v.v
                            });
                            var outputPos = Object.keys(tmpdata); //设置区域,比如表格从A1到D10
                            var tmpWB = {
                                SheetNames: ['Sheet1'], //保存的表标题
                                Sheets: {
                                    'Sheet1': Object.assign({},
                                        tmpdata, //内容
                                        {
                                            '!ref': outputPos[0] + ':' + outputPos[outputPos.length - 1] //设置填充区域
                                        })
                                }
                            };
                            tmpDown = new Blob([s2ab(XLSX.write(tmpWB, 
                                {bookType: (type == undefined ? 'xlsx':type),bookSST: false, type: 'binary'}//这里的数据是用来定义导出的格式类型
                                ))], {
                                type: ""
                            }); //创建二进制对象写入转换好的字节流
                        var href = URL.createObjectURL(tmpDown); //创建对象超链接
                        document.getElementById("hf").href = href; //绑定a标签
                        document.getElementById("hf").click(); //模拟点击实现下载
                        setTimeout(function() { //延时释放
                            URL.revokeObjectURL(tmpDown); //用URL.revokeObjectURL()来释放这个object URL
                        }, 100);
                    }

                    function s2ab(s) { //字符串转字符流
                        var buf = new ArrayBuffer(s.length);
                        var view = new Uint8Array(buf);
                        for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
                        return buf;
                    }
                     // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
                    function getCharCol(n) {
                        let temCol = '',
                        s = '',
                        m = 0
                        while (n > 0) {
                            m = n % 26 + 1
                            s = String.fromCharCode(m + 64) + s
                            n = (n - m) / 26
                        }
                        return s
                    }
                    downloadExl(jsono)

                }
```
### 为button绑定事件：
![undefined](https://cdn.nlark.com/lark/0/2018/png/146541/1545118006261-6c25e7b6-290c-4b74-a1f0-17f7077e8a8d.png) 

### 效果：
![undefined](https://cdn.nlark.com/lark/0/2018/png/146541/1545118052145-25c4fbcf-8a21-41af-a29a-6f4697c0b968.png) 

## 简单封装成鲁班组件：
app-export.js:
```javascript
Vue.component('app-export', Vue.extend({
    template: join([
        '<button is="app-button" id="{{id}}" bs-style="primary" @click="exportToExcel()">导出</button>',
        '<a href="" download="{{filename}}.xlsx" id="hf"></a>',
    ]),
    props: ['id', 'data','filename'],
    created: function () {
        this.id = (!this.id ? ('app-export' + RandomUtil.randomInt(10)) : this.id);
        VueUtil.parseProps(this);
        this.$root[this.id] = this;
        this.filename = (!this.filename ? ('export' + RandomUtil.randomInt(10)) : this.filename);
    },
    methods: {
        exportToExcel: function (jsono) {
            jsono = this.data
            if(jsono == undefined || jsono == ''||jsono.length == 0){
                return
            }
            var tmpDown; //导出的二进制对象
            function downloadExl(json, type) {
                
                var tmpdata = json[0];
                json.unshift({});
                var keyMap = []; //获取keys
                //keyMap =Object.keys(json[0]);
                for (var k in tmpdata) {
                    keyMap.push(k);
                    json[0][k] = k;
                }
                var tmpdata = [];//用来保存转换好的json 
                json.map((v, i) => keyMap.map((k, j) => Object.assign({}, {
                    v: v[k],
                    position: (j > 25 ? getCharCol(j) : String.fromCharCode(65 + j)) + (i + 1)
                }))).reduce((prev, next) => prev.concat(next)).forEach((v, i) => tmpdata[v.position] = {
                    v: v.v
                });
                var outputPos = Object.keys(tmpdata); //设置区域,比如表格从A1到D10
                var tmpWB = {
                    SheetNames: ['Sheet1'], //保存的表标题
                    Sheets: {
                        'Sheet1': Object.assign({},
                            tmpdata, //内容
                            {
                                '!ref': outputPos[0] + ':' + outputPos[outputPos.length - 1] //设置填充区域
                            })
                    }
                };
                tmpDown = new Blob([s2ab(XLSX.write(tmpWB, 
                    {bookType: (type == undefined ? 'xlsx':type),bookSST: false, type: 'binary'}//这里的数据是用来定义导出的格式类型
                    ))], {
                    type: ""
                }); //创建二进制对象写入转换好的字节流
                var href = URL.createObjectURL(tmpDown); //创建对象超链接
                    document.getElementById("hf").href = href; //绑定a标签
                    document.getElementById("hf").click(); //模拟点击实现下载
                    setTimeout(function() { //延时释放
                        URL.revokeObjectURL(tmpDown); //用URL.revokeObjectURL()来释放这个object URL
                    }, 100);
                }

                function s2ab(s) { //字符串转字符流
                    var buf = new ArrayBuffer(s.length);
                    var view = new Uint8Array(buf);
                    for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
                    return buf;
                }
                 // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
                function getCharCol(n) {
                    let temCol = '',
                    s = '',
                    m = 0
                    while (n > 0) {
                        m = n % 26 + 1
                        s = String.fromCharCode(m + 64) + s
                        n = (n - m) / 26
                    }
                    return s
                }
                downloadExl(jsono)

            }
        }
    }))

```
用法：
```html
<app-export filename="machine-info" :data="context.machineData"></app-export>
```