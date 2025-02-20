---
create_date: 2022-11-21T22:24:08 (UTC +08:00)
tags: wx/pbi/PQ数据处理 
aliases: null
pagetitle: PowerQuery | 利用List.Zip进行特殊格式的一维表转换
source: https://mp.weixin.qq.com/s/rD40NouVCR9YVtSWcpXdnw
author: 采悟
status: 已完成
category: 精读文章
notes: False
ZK: Origin
uid: 
---

最近有星友问到这样结构的表格如何转换为一维表，简化后的结构如下：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRy4sxwtxkyfekxjiaWOyesZAEcXlJmoCtc1FodkVPiaZsCojw1D9I1e4KQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

整体上看各个字段，好像已经是一维表了，每一列都是独立的属性，但是型号和数量是多个数据挤在一个单元格里，需要将他们按顺序拆分成行，变成下面这样，才算是标准的一维表：

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyeKtMYhcw46lXOCZMwPuficBqyLaTfbdAibGbxrjteyfchYFsWUqicNEbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如何完成这种转换呢？其实利用PowerQuery很简单，只需要下面几个步骤。  

**1\. 添加自定义列，将挤在一起的列拆分为列表**

利用==Text.Split函数分别将这两列拆分为列表==，先拆分\[型号\]列：

> Text.Split(\[型号\],",")

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyTiaQmU4L0jj4XPWWiadQHvFHVhvYSAcj739EPKdzBYic5z1OtLeEic5Ejw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

点击第一行的List可以在左下角看到拆分后的效果。

同样的方式，再添加个自定义列，拆分\[数量\]列：

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRy5wnqQ0TmfiaYBl6VTFbTF65iaEFnLyFaPmcHfJfrVicmm2gibuZsc5xichg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

通过这个步骤，就将挤在一个单元格内的数据拆分开了。

**2\. 利用List.Zip函数，将两个List合并到一起**

将==两个列表合并到一起，有个M函数是List.Zip专门做这种处理==，它可以将多个列表相同位置的数据合并到一起。

再添加个自定义列：  

> List.Zip( { \[型号列表\] , \[数量列表\] } )

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyezRBunicZhgSwypJgvS3540oJgHxla9FNLl3CrHhWbjEb6wmCzz9DmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上面两个步骤，其实也可以合并为一个步骤，==直接添加一个这样的自定义列==就可以了：  

> List.Zip(
> 
>     {
> 
>         Text.Split(\[型号\],","),
> 
>         Text.Split(\[数量\],",")
> 
>     }
> 
> )

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyeYfcRiapdvYibEeeQDGxl07Piagiced3vg9mZYw4FRANazkpdjov0WvPdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里处理以后，==List.Zip就可以将{A1,A2,A3}和{1,2,3}这样的列表转换成{ {A1,1},{A2,2},{A3,3} }的形式==，然后下一步将转后的列表展开就可以了。

**3\. 展开列表**

对于经过List.Zip处理后的自定义列，点击展开按钮，==选择“扩展到新行”==，  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyrNd5pM6vGwQX6M88m4JHGFAgC4CKxa8Ngpo8DibMEVv1qhib4R04wibcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

扩展以后这一列还是List，==再次展开，选择“提取值”：  ==

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRyibgoCYkicxjZddOTprf7p6hFne4leAibexdqd40B30K5HVFJpFQm32mbg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后自定义列就能看到想要的效果了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/aHEbZtANQJNcvQHNop91TA2ZqeWW6icRySh9orW8pOq8h23hAnY4cKTyBiaoCXtYBdAicyaH9OY5a35ek7zodLvpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**4\. 拆分自定义列，完成转换**

对上面的自定义列按分隔符拆分，并删除原有的\[型号\]和\[数量\]列，就得到了最终的效果。

上面这种数据转换的关键是M函数List.Zip函数的运用，利用它可以轻松将多个列表按顺序合并在一起，本文也是帮你理解该函数的一个很经典的示例。
