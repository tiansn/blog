这个是千里码里的题目。通过这个题感受到了10W级的数据。

下面是题目描述：
商品数量-1
当在天猫购物，使用关键词搜索商品时，可以选择很多附选条件，比如品牌，价格等，之后会出现符合当前条件的商品及数量。

如图，搜索词为：羽绒服，附选条件为：价格¥500-¥1000。

天猫搜索

主页陈列出符合条件的商品，以及一个商品数量（见右上角），但商品数量会因为商家上架、下架商品而改变。因此买家在不同时间段搜索商品时，显示的数量会不一样。

接下来包含一个含有多行的[文本文件](https://github.com/tiansn/blog/contents/goods1.txt)，文件的每行代表上架，下架，查询三种行为的一种，本题默认查询对象是衣服。

所有买家查询结果的总和就是本题的答案。

举例: 一个拥有6行的文件。

```
up 3 11 （有一个商家上架了3件11rmb的衣服。）
query 11 25 (有一个买家查询11rmb-25rmb的衣服的数量。这里的查询结果是:3件)
up 5 25 （有一个商家上架了5件25rmb的衣服。）
query 11 25 (有一个买家查询11rmb-25rmb的衣服的数量。这里的查询结果是:8件)
down 3 25 （有一个商家下架了3件25rmb的衣服。）
query 20 25 （有一个买家查询20rmb-25rmb的衣服的数量。这里的查询结果是:2件）
所以在这个例子里所有买家查询结果的总和为13（3+8+2=13）。

```

下面是我用node.js实现的方法，先记录在这里，现在跑完10W条数据要的时间是16s左右，算法待优化。

```
var lineReader = require('line-reader');
var startTime = Date.now();
lineReader.eachLine('./goods1.txt', (line, last) => {
    if (last) {
        countData(line, 'last');
    } else {
        countData(line);
    }
});

var queryCount = 0;
var goodsObj = {};

function countData(data, last) {
    var arrays = data.split(/\s+/);
    if (arrays[0] == 'up') {
        var num = parseInt(arrays[1]);
        var price = parseInt(arrays[2]);

        if (price in goodsObj) {
            goodsObj[price] += num;
        } else {
            goodsObj[price] = num;
        }
    }
    if (arrays[0] == 'down') {
        var num = parseInt(arrays[1]);
        var price = parseInt(arrays[2]);
        if (price in goodsObj) {
            goodsObj[price] -= num;
            if (goodsObj[price] <= 0) {
                delete goodsObj[price];
            }
        } else {
            console.info('没有可以下架的商品');
        }
    }
    if (arrays[0] == 'query') {
        var minPrice = parseInt(arrays[1]),
            maxPrice = parseInt(arrays[2]),
            tmpCount = 0;
        for (var i = minPrice; i <= maxPrice; i++) {
            if (i in goodsObj) {
                tmpCount += goodsObj[i];
            }
        }
        queryCount += tmpCount;
    }
    if (last) {
        var totalTime = Date.now() - startTime;
        console.info(totalTime / 1000);
        console.info(queryCount);
    }
}

```
