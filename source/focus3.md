## 「JsLab 科學計算平台」背後的開放原始碼結構

為了要用 JavaScript 建構出科學計算平台，我們創建了以下的 JsLab 開放原始碼專案。

* <https://github.com/ccckmit/jslab/tree/gh-pages>

我們大量的採用了 JavaScript 的開放原始碼函式庫，像是在「機率統計」領域採用了 jStat 這個專案，在繪圖領域採用了 d3.js、c3.js、vis.js 等等，這些專案的網址如下。

* <http://d3js.org/>
* <http://c3js.org/>
* <http://visjs.org/>
* <https://github.com/jstat/jstat>

JsLab 科學計算平台的核心，是一組基於 jStat 專案的重新封裝，我們將 jStat 重新封裝成類似 R 函式庫的介面，並且加上了一些 jStat 當中所沒有的功能，特別是統計檢定的部份，形成了 [R.js] 這個 JavaScript 程式，讓 JavaScript 也能擁有類似 R 軟體的機率統計函式庫。

我們雖然使用了 d3.js 這個繪圖函式庫進行 2D 繪圖，但由於 d3.js 並不容易使用，所以我們採用了 c3.js 這個基於 d3 的延伸套件，簡化 d3 繪圖函式庫的使用，讓我們不需瞭解太多繪圖的細節就能寫出 JsLab 的繪圖函式庫。

舉例而言、下列的圖形就是依靠 c3.js 所繪製的，只不過我們將 c3.js 進一步封裝到 JsLab 的 [G.js] 這個的繪圖函式庫當中而已。

![](../img/jsLabC3Graph.jpg)

以下是我們將 c3.js 重新封裝成 [G.js] 的程式碼片段。

檔案： [G.js]

```javascript
var C3G=function() {
  this.g = {
        data: {
          xs: {},
          columns: [ /*["x", 1, 2, 3, 4 ]*/ ],
		  type: "line", 
		  types : {}
        },
        axis: {
          x: {
            label: 'X',
            tick: { fit: false, format:d3.format(".2f") }
          },
          y: { label: 'Y', 
		    tick : { format: d3.format(".2f") }
    	  }
        }, 
		bar: { width: { ratio: 0.9 } }, 
      };
  this.varcount = 0;
  this.xrange(-10, 10);
  this.step = 1;
  this.setrange = false;
}
....
C3G.prototype.hist = function(x, options) {
  var name = R.opt(options, "name", this.tempvar()); 
  var mode = R.opt(options, "mode", ""); 
  var step = R.opt(options, "step", this.step); 
  var from = R.opt(options, "from", this.xmin); 
  var to   = R.opt(options, "to", this.xmax);
  this.g.data.types[name] = "bar";
  this.g.data.xs[name] = name+"x";
  var xc = R.steps(from+step/2.0, to, step);
  var n = (to-from)/step + 1;
  var count = R.repeats(0, n);
  for (var i in x) {
    var slot=Math.floor((x[i]-from)/step);
	if (slot>=0 && slot < n)
	  count[slot]++;
  }
  this.g.data.columns.push([name+"x"].concat(xc));
  var total = R.sum(count);
  if (mode === "normalized")
    count = R.apply(count, function(c) { return (1.0/step)*(c/total); });
  this.g.data.columns.push([name].concat(count));
}
...
C3G.prototype.show = function() {
  if (typeof(module)==="undefined")
    return c3.generate(this.g);
}
```

另外、由於 d3.js 並沒有支援 3D 曲面圖形的繪製，所以我們又引入了 vis.js 這個繪圖函式庫來完成 3D 圖形的繪製工作。

以下是我們將 vis.js 封裝成 G.js 的程式碼片段

檔案： [G.js]

```javascript
...
VISG.prototype.curve3d = function(f, box) {
  // Create and populate a data table.
  var data = new vis.DataSet();
  // create some nice looking data with sin/cos
  var counter = 0;
  var steps = 50;  // number of datapoints will be steps*steps
  var axisMax = 314;
  var axisStep = axisMax / steps;
  for (var x = 0; x < axisMax; x+=axisStep) {
    for (var y = 0; y < axisMax; y+=axisStep) {
      var value = f(x,y);
      data.add({id:counter++,x:x,y:y,z:value,style:value});
    }
  }
  this.graph = new vis.Graph3d(box, data, this.options);
}
...
G.curve3d = function(f) {
  G.visg.curve3d(f, G.box);
}
```

接著、為了讓使用者編輯程式可以更容易，我們採用了這個 JavaScript 開源線上編輯器專案，該專案之援類似微軟 Visual Studio 的 IntelliSense 這樣的函數提示功能，

* <http://codemirror.net/>

![圖、JsLab 當中的程式碼上色與 IntelliSense 功能](../img/JsLabIntelliSense.jpg)

以下是我們將 codemirror 封裝成 [E.js] (Editor) 的完整程式碼。

檔案： [E.js] 

```javascript
"use strict";

U.use("../js/codemirror/lib/codemirror.js", "CodeMirror");
U.use("../js/codemirror/addon/hint/show-hint.js");
U.use("../js/codemirror/addon/hint/javascript-hint.js");
U.use("../js/codemirror/mode/javascript/javascript.js");

E = {};

if (typeof module!=="undefined") module.exports = E;

E.editor = null;

E.loadEditor = function(codebox) {
  E.codebox = codebox;
  E.editor = CodeMirror.fromTextArea(codebox, {
    lineNumbers: true,
    extraKeys: {"Ctrl-.": "autocomplete"},
    lineWrapping: true, 
    styleActiveLine: true,
    mode: {name: "javascript", globalVars: true}
  });
  
  E.editor.on('update', function(instance){
    E.codebox.value = instance.getValue();
  });  
}
```

未來、我們可能會進一步整合更多的函式庫，目前預計在「矩陣運算領域」會採用了 jStat 與 numeric.js 等函式庫，而在數位訊號語音處理領域可能會採用 DSP.js，影像處理領域可能採用  CamanJS
等等，這些函式庫的網址如下：

* <http://numericjs.com/>
* <https://github.com/corbanbrook/dsp.js/>
* <http://camanjs.com/>

目前、網路上的 JavaScript 的函式庫還持續的急速增長中，我們相信之後會有更多更好用的 JavaScript 科學計算函式庫出現，我們只要能將這些好用的函式庫整合進來，就能創建一個完整的科學計算平台了，而這也正是 JsLab 計劃所想要達成的目標。

由於 JavaScript 是瀏覽器當中唯一能使用的官方語言，因此我們認為未來 JavaScript 的發揮空間將會越來越大，就像 Jeff Atwood 在 2007 年所提出的 Atwood's Law 所說的：

> Any application that can be written in JavaScript, will eventually be written in JavaScript.

換句話說 -- 「任何可以寫成 JavaScript 的應用程式，最後都會被寫成 JavaScript」。

如果 Atwood's Law 成立的話，那基於 JavaScript 的科學計算平台就應該要出現啊！

我們正在以行動來實現 Atwood 的話，這個行動的代號就是 JsLab 。

### 參考文獻

* [Atwood定律：“任何可以使用JavaScript来编写的应用，最终会由JavaScript编写。”](http://www.iterduo.com/0401-atwood.html)

[G.js]:https://github.com/ccckmit/jslab/blob/gh-pages/source/G.js
[E.js]:https://github.com/ccckmit/jslab/blob/gh-pages/source/E.js
[R.js]:https://github.com/ccckmit/jslab/blob/gh-pages/source/R.js

