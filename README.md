# A Metro Messy Transfer System
一个非常复杂繁琐的地铁换乘系统(*非人类编写*)，于临时个人需求使用，但不建议尝试。该系统为纯js代码实现且可单网页运行。

#### 该项目引用了 `panzoom.min` js ，请确保您的 html 文件目录下含有`panzoom.min.js`文件，或于 html 文件中更改引用 `panzoom.min` js 的路径。

## Step 1
### 首先你需要自己绘制一张svg格式的地铁图。
站点应用 **圆圈(`circle`)** 或 **矩形(`rect`)** 进行绘制

线路应用 **路径(`path`)** 或 **`polyline`** 进行绘制

虚拟换乘标识(图中用虚线来表示)可等价视做一种特殊的地铁线路，用 **路径(`path`)** 或 **`polyline`** 进行绘制之后，将线段颜色调成灰色虚线即可（或者随你来）。

我是使用[AARC](https://github.com/Aurouscia/aarc)进行绘制，再使用 **AI(Illustrator)** 绘制软件（不是人工智能那个ai）进行描绘。

应注意整条地铁线之间不应为连续的一条线，应每个站点与站点之间都分隔一条线。

> [!TIP]
> 你可以先绘制一条完整的地铁线路，再使用橡皮擦工具进行分隔。(详细操作可参考 [b站视频BV1ED4y1t7tu](https://www.bilibili.com/video/BV1ED4y1t7tu/) 1分30秒处)
> 
![readme1.png](<readme-image/readme1.png>)
> [!NOTE]
> 如图，在`map1.svg`中，有四个站点，也被分割成了四条线段路径，图中展示的为峡谷湾和松山站的线段路径。

若使用 **Illustrator** 绘制好地图，在文件-导出-导出为svg格式即可。

## Step 2
### 此步骤最为麻烦！！！！！这也是我不推荐使用的原因，或者说这是此系统不适用于大型地铁线路图的原因。

### 你需要对每条线段，每个地铁站都进行标识。

1. **地铁站**
你需要对每一个地铁站图标后面添加标注代码 **地铁站名称`data-station-name`** 、**地铁站id`data-station-id`** 、**地铁站所属线路`data-station-line`**(地铁站所属线路可选略不标注)。

   例如：

   如下是远航城际线松山站(非换乘站)

    ```HTML
    <circle class="cls-22" cx="963.51" cy="1116.57" r="10.6" data-station-name="松山站" data-station-id="S002" data-station-line="远航城际" style="cursor: pointer;"></circle>
    ```
    如下是主城城际和远航城际的峡谷湾站(站内换乘站)
    ```HTML
    <rect class="cls-18" x="740.87" y="840.98" width="20.43" height="46.47" rx="10.21" ry="10.21" transform="translate(1615.3 113.13) rotate(90)" data-station-name="峡谷湾" data-station-id="S028" data-station-line=""主城城际","远航城际"" style="cursor: pointer;"></rect>
    ```

> [!WARNING]
> 每一个非可站内换乘的独立站点都应该被分配一个独立的id。即便是可通过虚拟换乘的同名站点。
> 
> 如黄线与灰线的 **遗忘花园站** ，即便两个站点同名且可虚拟换乘，但还是被分配了两个独立的id。
> 
> 而海底观光列车与滨海环线观光列车的 **观景平台车站** 可站内换乘，所以可以被分配同一个id。

2. **地铁线路**
你需要对每一个地铁线路后面添加标注代码 **线段所属地铁线路`data-line`** 、 **该线段连接的两地铁站点`data-connet`** 、 **线段是否为单向线段(可略)`data-direction`**。

    线段是否为单向线段`data-direction`有三个有效字段: **`forward`** 、 **`backward`** 、 **`both`** 。

   若为 **`forward`** 则表示该线路只能正向通行(方向与`data-connet`有关)、 **`backward`** 则表示该线路只能反向通行、 **`both`** 则为双向通行。若地铁线段不标注 `data-direction` ，则默认为 **`both`** 。

    例如：
    地铁线段跨洋城际 **峡谷湾(id=S012)** 至 **末地传送大厅站(id=S016)** 。此段线段未标注 `data-direction` ，故为双向通行。
    ```HTML
    <path class="cls-1 route-edge" d="M813.23,839.69c195.29-.75,389.29,2.25,584.29-.75,12-6,23-15,25-28,2-16,1-32,2-48,1-24,0-47,0-71,1-73,0-147-.4-219.58" data-line="跨洋城际" data-connet=""S012","S016""></path>
    ```

    地铁线段拉特兰专线 **终末地(id=S035)** 至 **伊始站(id=S033)** 。此段线段标注 `data-direction` 为 `forward`，故为单向通行，仅可从 **终末地** 到 **伊始站** 。若需从 **伊始站** 到 **终末地** ，需绕行乘坐。
   ```HTML
   <polyline class="latelan_line" points="1530.79 510 1530.79 800" data-line="拉特兰专线" data-connet=""S035","S033"" data-direction="forward"></polyline>
   ```
   
> [!IMPORTANT]
> 你需要特别注意`data-connet`的格式，其格式为`data-connet=""站点A_id","站点B_id""`。请注意引号(")的数量，可能会在你标注时，ide会自动闭合你的引号导致无法正常连接两个地铁站。
> 
> 如果对此格式不理解，可以尝试修改其原js代码。（求求了TAT对不起！）

3. **虚拟换乘线路**
你需要对每一个虚拟换乘线路后面添加标注代码 **虚拟换乘线路标识符`data-line-type`** 、**该线路连接的两地铁站点`data-line-change`**。

    例如：
    如下是主城城郊线 **熔炉奇观站(id=S005)** 与远航城际 **主城北站(id=S019)** 的虚拟换乘线
    ```HTML
    <path class="cls-9" d="M913.98,263.81l-5.55,3.85c-15.35,10.65-15.35,33.35,0,44l5.55,3.85" data-line-type="transfer" data-line-change=""S005","S019""></path>
    ```

> [!IMPORTANT]
> 你需要特别注意`data-line-change`的格式，其格式为`data-line-change=""站点A_id","站点B_id""`。请注意引号(")的数量，可能会在你标注时，ide会自动闭合你的引号导致无法正常连接两个地铁站。
> 
> 如果对此格式不理解，可以尝试修改其原js代码。（求求了TAT对不起！）

4. **其他注释及图例**

不需要进行额外的标注。

## Step 3
### 将svg地图导入到html中。
在html中找到这一行:
```HTML
<div id="map" style="overflow: hidden; user-select: none; touch-action: none;">
  <!--?xml version="1.0" encoding="UTF-8"?-->
  <svg ……以下略>
    以下略
  </svg>
</div>
```
将中间的svg替换为你的svg地图。

此时有概率你会看到你的text标注会乱飞。请您打开浏览器的调试控制台，一个个选择text标注调试他的`transform="translate(x y)"`。

(兄弟，都弄到这了，没办法了，弄吧=.=)

浏览器的更改不会同步到源代码，还请您一个个调试好之后自己同步更改到源代码啦。

## Step 4

大功告成！可以开始用啦！
