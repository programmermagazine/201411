## d3.js -- 互動式繪圖框架

d3.js 是一個在瀏覽器裏使用的互動式繪圖框架，使用 HTML、CSS、JavaScript 與 SVG (Scalable Vector Graphics) 等技術。

d3.js 專案起始於 2011 年，是從 [Protovis](http://en.wikipedia.org/wiki/Protovis) 專案修改過來的，通常我們會用 D3 來產生 SVG 或 CSS 的繪圖結果，在瀏覽器上檢視時還可以利用 SVG 或 CSS 與使用者進行互動。

d3.js 的用法有點像 jQuery，都是透過選擇器來進行選取後操作的，舉例而言、下列指令可以選出所有 p 標記的節點並將顏色修改為 lavender (淡紫色、熏衣草)。

```javascript
 d3.selectAll("p")
   .style("color", "lavender");
```

我們可以透過「標記 tag、類別 class、代號 identifier、屬性 attribute、或位置 place 來選取節點，然後進行新增、刪除、修改等動作，然後透過設定 CSS 的轉移 (transition) 屬性，讓繪圖的結果可以和使用者進行互動，舉例而言、以下程式就會讓網頁裡的 p 標記節點逐漸地改變為紫色。 (d3.js 預設的改變速度為 250ms 完成轉換)

```javascript
 d3.selectAll("p")
   .transition()
   .style("color", "pink");
```

由於 SVG 裏的標記也是 HTML 的一部分， d3 的指令也可以選取 SVG 裏的內容，以下來自 [Mike Bostock] 網站的 [範例](http://bost.ocks.org/mike/circles/) 顯示了這個狀況：

```html
<svg width="720" height="120">
  <circle cx="40" cy="60" r="10"></circle>
  <circle cx="80" cy="60" r="10"></circle>
  <circle cx="120" cy="60" r="10"></circle>
</svg>
...
<script>
var circle = d3.selectAll("circle");
circle.style("fill", "steelblue");
circle.attr("r", 30);
</script>
```

d3.js 的主要 API 包含下列幾類：

* Selections
* Transitions
* Arrays
* Math
* Color
* Scales
* SVG
* Time
* Layouts
* Geography
* Geometry
* Behaviors

而以下的專案則是延伸自 d3.js 的套件，

* [freeDataMap](http://freedatamap.com/) - Company data visualisation tool
* [dimple.js](http://dimplejs.org/) - Flexible axis-based charting API
* [Cubism](http://square.github.io/cubism/) - Time series visualisation
* [Rickshaw](http://code.shutterstock.com/rickshaw/) - Toolkit for creating interactive time series graphs
* [NVD3](http://nvd3.org/) - Re-usable charts for d3
* [Crossfilter](http://square.github.io/crossfilter/) - Fast Multidimensional Filtering for Coordinated Views
* [dc.js](http://nickqizhu.github.io/dc.js/) - Dimensional Charting Javascript Library
* [c3.js](http://c3js.org/) - D3-based reusable chart library

甚至還有人專門為 d3.js 寫了一本書，而且這本書還有中文版。

* [網頁互動式資料視覺化：使用D3](http://www.books.com.tw/products/0010621239), Scott Murray.

### 參考文獻
* Wikipedia: D3.js -- <http://en.wikipedia.org/wiki/D3.js>
* Mike Bostock -- <http://bost.ocks.org/mike/>
* <http://d3js.org/>
* [D3 Gallery](https://github.com/mbostock/d3/wiki/Gallery)

[Mike Bostock]:http://bost.ocks.org/mike/

