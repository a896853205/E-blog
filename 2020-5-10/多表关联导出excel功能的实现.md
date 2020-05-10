## 多表关联导出excel功能的实现

需求:多表关联导出个人填写信息

如下图,A为主表,链接B,C,D,E,F,G表,与B表为一对一关系,与C,D,E,F,G表为一对多的关系,表中A表示A表中某用户的所有信息,并非为一个字段(列)

| A    | B    | C1   | D1   | E1   | F1   | G1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      | C2   |      | E2   |      | G2   |
|      |      | C3   |      |      |      | G3   |
|      |      | C4   |      |      |      |      |

 

#### 方向1:导出word

找到了两个常用的插件 [Docxtemplater](https://www.npmjs.com/package/docxtemplater)和[officegen](https://www.npmjs.com/package/officegen)

插件1:

使用word模板插件Docxtemplater,先在项目文件夹里新建一个模板文件,将需要填充的地方用{name}这样的占位符代替.

优点:这个api可以生成二进制数据buffer数组,便于导出文件url

缺点:格式固定,无法灵活生成文档

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/C5I6tuQmWty2LHkfPkGvPsfcJ2OGbTkkm0mXUpc8Y3yZp*n3En1Vn1H8ItO*JJ59PZ3KbgFid9B9ObLuLD22rA!!/b&bo=tAIrAQAAAAARB6w!&rf=viewer_4) 

```javascript
   // Load the docx file as a binary

   let content = fs.readFileSync(

     path.resolve(__dirname, '../../../template/record.docx'),

     'binary'

   );

 

   let zip = new PizZip(content);

   let doc = new Docxtemplater();

   doc.loadZip(zip);

   let data = {};

 

   Object.assign(

     data,

     contract,

     basic,

     registration,

     contractManager,

     specimen

   );

 

   moment.locale('zh-cn');

   // 设置模板数据

   doc.setData(data);

   // render the document (replace all occurences of {first_name} by John, {last_name} by Doe, ...)

   doc.render();

 

   let buf = doc.getZip().generate({ type: 'nodebuffer' });
```


插件2:

使用[officegen](https://www.npmjs.com/package/officegen)模块,其中有doc文件的接口,代码如下:

优点:可以自定义生成文本,插入表格、图片等等,可以自定义文本格式，并且可以导出ppt等文件

缺点:没有导出流文件，导出buffer的方法，只能在本地生成word


```javascript
  let docx = officegen('docx');
 

  let pObj = docx.createP({ align: 'center' }); // 创建行 设置居中 大标题

 

  pObj.addText('全国所有城市', {

  bold: true,

  font_face: 'Arial',

  font_size: 18,

  }); // 添加文字 设置字体样式 加粗 大小

 

  let dataLen = data.length; // data为json格式

  for ( let i = 0; i < dataLen; i++ ) {

  /************************* 文本 *******************************/

   pObj = docx.createP(); //创建一行

   pObj.addText(`(${i+1}), `, { bold: true, font_face: 'Arial' });

   pObj.addText(`省级:`, { bold: true, font_face: 'Arial' });

   pObj.addText(`${data[i]['provinceZh']} `);

   pObj.addText(`市级：`, { bold: true, font_face: 'Arial' });

   pObj.addText(`${data[i]['leaderZh']} `);

   pObj.addText(`县区：`, { bold: true, font_face: 'Arial' });

   pObj.addText(`${data[i]['cityZh']}`);

  /************************* 表格 *******************************/

   let SingleRow = [

   data[i]['id'],

   data[i]['provinceZh'],

   data[i]['leaderZh'],

   data[i]['cityZh'],

   ];

   table.push(SingleRow);

 }

 docx.createTable(table, tableStyle);

 let out = fs.createWriteStream('out.docx'); // 文件写入

 docx.generate(out); // 服务端生成word
```


结论:第一种方法不灵活,导出的word样式固定,当导出内容长度不固定的时候就不适合使用此方法,第二种方法灵活性很好,但是输出的文件只能保存在后端,上传到前端给用户查看比较费事,且无法实现预览功能,而且在导出多人的信息的时候word很不直观.因此没有考虑使用导出word这一思路.

 

#### 方向2:导出excel

首先想到使用sequelize查询A表通过6次左连接查询,输出结果如下表:

| A    | B    | C1   | D1   | E1   | F1   | G1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| A    | B    | C2   | D1   | E2   | F1   | G2   |
| A    | B    | C3   | D1   | E1   | F1   | G3   |
| A    | B    | C4   | D1   | E2   | F1   | G1   |
| A    | B    | C1   | D1   | E1   | F1   | G2   |
| A    | B    | C2   | D1   | E2   | F1   | G3   |
| A    | B    | C3   | D1   | E1   | F1   | G1   |
| A    | B    | C4   | D1   | E2   | F1   | G2   |
| A    | B    | C1   | D1   | E1   | F1   | G3   |
| A    | B    | C2   | D1   | E2   | F1   | G1   |
| A    | B    | C3   | D1   | E1   | F1   | G2   |
| A    | B    | C4   | D1   | E2   | F1   | G3   |
| A    | B    | C1   | D1   | E1   | F1   | G1   |
| A    | B    | C2   | D1   | E2   | F1   | G2   |
| A    | B    | C3   | D1   | E1   | F1   | G3   |
| A    | B    | C4   | D1   | E2   | F1   | G1   |
| A    | B    | C1   | D1   | E1   | F1   | G2   |
| A    | B    | C2   | D1   | E2   | F1   | G3   |
| A    | B    | C3   | D1   | E1   | F1   | G1   |
| A    | B    | C4   | D1   | E2   | F1   | G2   |
| A    | B    | C1   | D1   | E1   | F1   | G3   |
| A    | B    | C2   | D1   | E2   | F1   | G1   |
| A    | B    | C3   | D1   | E1   | F1   | G2   |
| A    | B    | C4   | D1   | E2   | F1   | G3   |

 

显然,导出结果条数为1×4×1×2×1×3 = 24条,也就是所谓的笛卡尔乘积,无法达到预期结果

尝试使用distinct和group语句进行结果去重,但是这两个语句仅仅可以在同一张表内去重,比如假设C1和C5数据是一样的,那么返回结果中只是不会出现C5的数据,并不能改变输出条数为笛卡尔乘积的事实.

 

此时我开始了漫长的搜索之旅,几番周折竟然真的找到了一模一样的需求,真的是一模一样的!

[点此查看](https://bbs.csdn.net/topics/390559635?utm_medium=distribute.pc_relevant.none-task-discussion_topic-BlogCommendFromBaidu-3&depth_1-utm_source=distribute.pc_relevant.none-task-discussion_topic-BlogCommendFromBaidu-3)

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4Ne0a9npnalVnaziy1zlLFhUkCFnnQwvU4VkoP1WYLPul5YuNKwxATo765QzQwbK.z8eXWHgItdYSb.tkEmrF7N0!/b&bo=iwHLAQAAAAARF2A!&rf=viewer_4) 

如上图,回复中有人说用id连接b,c,d表,考虑到每张表里面会出现不同的keyjobno值,这个方案显然行不通,又有人说使用设置行号功能,比如row_number()和rank()函数,但是我在navicat上试了半天都说语法错误,百度了一下竟然是说mysql没有生成行号的功能,此时我还没死心,想试试找一找有没有方法自己生成行号的,或者使用函数等等,结果还真的有,先set一些初值来保存行号,然后再每次连接查询的时候+1就可以了,代码如下:


```sql
set @row_number1 = 0,@row_number2 = 0,@row_number3 = 0,@row_number4 = 0;

 

SELECT  a.id ,

        a.name,

		b.age, 

		b.rank2,

		c.college,

		c.rank3,

		d.address

FROM  (SELECT * from t_name) a

  RIGHT JOIN ( SELECT *,@row_number2:=@row_number2+1 AS rank2 FROM t_age) b  ON a.name = b.name 

  LEFT JOIN (SELECT *,@row_number3:=@row_number3+1 AS rank3 FROM t_college) c  ON a.name = c.name AND b.rank2=c.rank3 

LEFT JOIN (SELECT *,@row_number4:=@row_number4+1 AS rank4 FROM t_address) d  ON a.name = d.name	AND b.rank2=d.rank4		

WHERE a.name = ”xiaoming”
```


 D表t_college                                  B表t_age                         C表t_address                             A表t_name

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NTsYb6talHSBfdziYuCy8JL3mIg.z56CFmSaIF7aNf3vFBSwylgg96MzkUda3LFgxMzIiK*XXCTC68jGBa4.KDU!/b&bo=0ACAAAAAAAARF3A!&rf=viewer_4)![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NedVKUvJ.qJy1DPC4tERqc7BLwMMQghvJxp1pmh4HnNCckbC5*CtVEt3*pD1y.QBRJDViDS0R.m5EqUIxqjonSg!/b&bo=ugBhAAAAAAARF*s!&rf=viewer_4)![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NWBpBTbyo90YlC61Tp4E.nTV80.xRNx1bZLZ1sIQVGccW5bfkhM6VLJxmEPDWQk1M*q16YYIOqTqKv2hFHaZDgo!/b&bo=0QBiAAAAAAARF5M!&rf=viewer_4)![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NWQukx8H32MXpLZhOAl0QJbslZq6MZogE2FJJplRPU5vCxE9ugMNilLxk1ozHaLwxPFd.xrvPDbHwbRd75Zqs3E!/b&bo=fgBHAAAAAAARFxk!&rf=viewer_4)



 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NTLAs6PyWCY*PJ8vUBRggkluT3jFhFX3hnvC6Uoo4noHBJqvxRSHHkjsUL6VAlwbudYuICqbxvixYeHc60uSrwg!/b&bo=tAJdAAAAAAARF8s!&rf=viewer_4) 

 

可以看到主表A先用了一次右连接,连接了行数最多的B表,再使用B的行号依次左连接匹配C,D的行号,输出的结果就是上图左边的这样,可以达到预期的目标.

如果不先右连接B表而是连接其他表,假设连接C表,因为C表的行数小于3,匹配行号的时候匹配不到行号 = 3这一条数据,所以输出是右边的情况,少输出一个B3的数据,输出为上图右边的这样.

此时我已经打算使用这种看似不错但写起来并不优雅的方案了,难点也来了,查询语句非常复杂,又有左连接又有右连接用sequelize写语句非常困难,直接引用mysql语句会出现需要大量判断的情况(先要查询出行数最大的那一条,然后把它放在第一个右连接的位置,然后查询剩下的,光引入的行号@row_number2还是3456就需要好几次判断,而且表中数组有重名的还需要使用select as语句)

 

几番思想斗争之后还是放弃了,突然想到之前有一次写findAll的时候没有设置raw:true,导致返回了一个大的resultEntity对象,没想到就是这玩意儿居然是可以用的(原理到现在我还是不太明白),格式大概是这样的:{A:{...},[B],[{C1},{C2},{C3}],[D1 D2 D3],...},当然实际上要比这复杂一些,写了一些函数从这个大的对象中把这些数据全部掏出来,放到二维数组中去,然后用xlsx或者node-xlsx插件生成buffer数据流,通过服务器生成url导出到前端就ok了.

代码如下,数据处理那一块太长就省略了,导出excel文件的url步骤如下:


``` JavaScript
 let data = [];

  let title = ['账号', '姓名', '电话', '权限', '状态', '部门'];

  data.push(title);

  _data.forEach((element) => {

   let arrInner = [];

   arrInner.push(element.userName);

   arrInner.push(element.name);

   arrInner.push(element.phone);

   arrInner.push(roleToText(element.role));

   arrInner.push(element.isCancel);

   arrInner.push(element.department);

   data.push(arrInner); //data中添加的要是数组，可以将对象的值分解添加进数组，例如：['1','name','上海']

  });

 

  let buffer = xlsx.build([

   {

     name: 'sheet1',

     data: data,

   },

  ]);

  // 上传到oss

  const fileUuid = uuid.v1(),

   fileUrl = `temp/adminExportAll/${fileUuid}.xlsx`;

  // 上传文件

  await client.put(fileUrl, buffer);

  return await client.signatureUrl(fileUrl);
```


导出的效果类似下表这样:

| A    | B    | C1   | D1   | E1   | F1   | G1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      | C2   |      | E2   |      | G2   |
|      |      | C3   |      |      |      | G3   |
|      |      | C4   |      |      |      |      |

 

耗时整整三天的导出功能终于实现了,当然如果按照函数生成二维数组的思路还可以使用多次查询,使用排序代替行号,然后进行数据拼接的方法等等,当然这样的方法可能更不优雅,查询的次数也要多得多,总之还有很多要学习的地方,如果有更好的方法也可以一起交流学习!

 

其他:

1、解决浮点类型计算不准确问题

问题描述：float、double浮点类型数值计算时会产生偏差，这是由于小数在存储二进制数存在误差导致的，例如0.1+0.2计算出来的数值不等于0.3，而是0.3000000000001,这也就导致在后端判断两个带小数的浮点类型数值时产生偏差

 

解决方法:

(1)最初考虑使用.tofixed()函数,只保留小数点后两位,但仍然出现偏差.

(2)误差控制法:令A=0.1,B=0.2,C=A+B-0.3,判断C的绝对值是否小于2^-64,若小于,则可以认为A+B-C===0,即A+B=C

(3)还可以使用更加精确的数值类型:decimal类型

对于精度比较高的东西，比如money，我会用decimal类型，不会考虑float,double,因为他们容易产生误差，numeric和decimal同义，numeric将自动转成decimal。

DECIMAL从MySQL 5.1引入，列的声明语法是DECIMAL(M,D)。在MySQL 5.1中，参量的取值范围如下：

 

·M是数字的最大数（精度）。其范围为1～65（在较旧的MySQL版本中，允许的范围是1～254），M 的默认值是10。

·D是小数点右侧数字的数目（标度）。其范围是0～30，但不得超过M。

 

说明：float占4个字节，double占8个字节，decimail(M,D)占M+2个字节。

 

2、前端预览app的方法

office文档在线预览功能，在微软给出的url后面拼接上想要预览的office文件下载链接，就可以让任何用户预览该office文件。功能类似于Office Web View，但Office Web View对word和ppt大小限制为10M，excel为5M，不符合产品需求。

代码如下:
``` JavaScript
const url = 

`http://view.officeapps.live.com/op/view.aspx?src=${encodeURIComponent(

   tempUrl

  )}`;

window.open(url);
```


其中,encodeURIComponent()函数可把字符串作为 URI 组件进行编码.

 