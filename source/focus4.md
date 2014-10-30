## R.js -- 從 jStat 延伸出的開源 JavaScript 機率統計框架

我自從開始用 R 學習機率統計之後，就覺得這樣的科學計算平台真的很棒。

但可惜的是、我現在所使用的主力語言是 JavaScript，因為 JavaScript 是瀏覽器的唯一語言，而且有了 node.js 這樣的平台之後，可以通吃客戶端與伺服端兩方的應用。

於是我想要用 JavaScript 創造出一個類似 R 的環境，並且可以在 Web 上執行，所以就創造了 jsLab 專案。

但是、為了讓 jsLab 能支援那些機率統計功能，我必須尋找用 JavaScript 語言撰寫的機率統計函式庫。

在 2012 年我就注意到了 jStat 這個支援機率模型的函式庫，當時這個函式庫還有專屬的網站，但是現在這個函式庫連網站都不見了，還好在 github 裏還有一份原始碼，網址如下：

* <https://github.com/jstat/jstat>

jStat 在機率模型的部分支援還算完整，但是在統計檢定的部分就相當薄弱，雖然也支援矩陣運算等功能，但是在 JavaScript 語言上， jStat 的矩陣運算支援並不算特別好的。

因此、我們決定將 jStat 重新包裝，成為一個新的 javascript 檔案，稱為 R.js ，這是 jsLab 專案的主要程式碼之一， R.js 的網址如下。

* <https://github.com/ccckmit/jslab/blob/gh-pages/source/R.js>

您可以看到在 R.js 檔案裏，能夠呼叫 jStat 完成的部分，我們都盡可能的交給 jStat 來做，而 jStat 在這部份也確實做得很不錯。

檔案： R.js

```javascript
...
// 均等分布
R.runif=function(n, a, b) { return R.calls(n, jStat.uniform.sample, a, b); }
R.dunif=function(x, a, b) { return jStat.uniform.pdf(x, a, b); }
R.punif=function(q, a, b) { return jStat.uniform.cdf(q, a, b); }
R.qunif=function(p, a, b) { return jStat.uniform.inv(p, a, b); }
// 常態分布
R.rnorm=function(n, mean, sd) { return R.calls(n, jStat.normal.sample, mean, sd); }
R.dnorm=function(x, mean, sd) { return jStat.normal.pdf(x, mean, sd); }
R.pnorm=function(q, mean, sd) { return jStat.normal.cdf(q, mean, sd); }
// R.qnorm=function(p, mean, sd) { return R.q2x(p, function (q) { return R.pnorm(q, mean, sd);});}
R.qnorm=function(p, mean, sd) { return jStat.normal.inv(p, mean, sd); }
// 布瓦松分布
R.rpois=function(n, l) { return R.calls(n, jStat.poisson.sample, l); }
R.dpois=function(x, l) { return jStat.poisson.pdf(x, l); }
R.ppois=function(q, l) { return jStat.poisson.cdf(q, l); }
R.qpois=function(p, l) { return jStat.poisson.inv(p, l); }

// F 分布
R.rf=function(n, df1, df2) { return R.calls(n, jStat.centralF.sample, df1, df2); }
R.df=function(x, df1, df2) { return jStat.centralF.pdf(x, df1, df2); }
R.pf=function(q, df1, df2) { return jStat.centralF.cdf(q, df1, df2); }
R.qf=function(p, df1, df2) { return jStat.centralF.inv(p, df1, df2); }
// T 分布
R.rt=function(n, dof) { return R.calls(n, jStat.studentt.sample, dof); }
R.dt=function(x, dof) { return jStat.studentt.pdf(x, dof); }
R.pt=function(q, dof) { return jStat.studentt.cdf(q, dof); }
R.qt=function(p, dof) { return jStat.studentt.inv(p, dof); }
...
```

不過、在離散的機率分布上面，jStat 就支援的比較不好，而且沒有支援像 inv 這類的函數，於是我們就得自己來補足，以下是我們用 jStat 與自己寫的程式所合成的一些離散分布原始碼。

```javascript
...
R.sample1=function(a, p) { 
  var r = Math.random();
  var u = R.repeats(1.0, a.length);
  p = R.def(p, R.normalize(u));
  var psum = 0;
  for (var i in p) {
    psum += p[i];
	if (psum > r)
	  return a[i];
  }
  return null;
}

R.sample=function(x, n, p) { return R.calls(n, R.sample1, x, p); }

// 二項分布
R.rbinom=function(n, N, p) { return R.calls(n, jStat.binomial.sample, N, p); }
R.dbinom=function(x, N, p) { return jStat.binomial.pdf(x, N, p); }
R.pbinom=function(k, N, p) { return jStat.binomial.cdf(k, N, p); }
R.qbinom=function(q, N, p) { 
  for (var i=0; i<=N; i++) {
    if (R.pbinom(i, N, p) > q) return i;
  }
  return N;
}

// 負二項分布
R.rnbinom=function(n, N, p) { return R.calls(n, jStat.negbin.sample, N, p); }
R.dnbinom=function(x, N, p) { return jStat.negbin.pdf(x, N, p); }
R.pnbinom=function(k, N, p) { return jStat.negbin.cdf(k, N, p); }
// R.qnbinom=function(p, N, q) { return jStat.negbin.inv(p, N, q); }
R.qnbinom=function(q, N, p) { 
  for (var i=0; i<N; i++) {
    if (R.pnbinom(i, N, p) > q) return i;
  }
  return N;
}
...
```

另外、由於 jStat 在統計檢定方面的函數也很薄弱，所以我們撰寫了以下這個檢定的抽象函數，實作時只要補足「 q2p, o2q, h, df」等函數，就可以做出一個檢定功能了。

```javascript
...
R.test = function(o) { // name, D, x, mu, sd, y, alpha, op
  var D      = o.D;
  var x      = o.x;
  var alpha  = R.opt(o, "alpha", 0.05);
  o.op       = R.opt(o, "op", "=");
  
  var pvalue, interval;
  var q1     = D.o2q(o); // 單尾檢定的 pvalue
  
  if (o.op === "=") {
    if (q1>0.5) q1 = 1-q1; // (q1>0.5) 取右尾，否則取左尾。
    pvalue= 2*q1; // 對稱情況：雙尾檢定的 p 值是單尾的兩倍。
    interval = [D.q2p(alpha/2, o, "L"), D.q2p(1-alpha/2, o, "R")];
  } else {
	if (o.op === "<") { // 右尾檢定 H0: q < 1-alpha, 
	  interval = [ D.q2p(alpha, o, "L"), Infinity ]; 
	  pvalue = 1-q1;
	}
	if (o.op === ">") { // 左尾檢定 H0: q > alpha
	  interval=[-Infinity, D.q2p(1-alpha, o, "R")];
	  pvalue = q1;
	}
  }

  return { name: o.name,
		   h: D.h(o),
		   alpha: alpha,
		   op: o.op, 
		   pvalue: pvalue, 
           ci : interval, 
		   df : D.df(o),
		   report: function() { R.report(this) }
		   };
}
...
```

舉例而言，以下這個物件 t1 實作了 R.test 中所需要的函數，因此我們就可以透過「 `o.D = t1;	t = R.test(o);` 這兩行指令呼叫 R.test 函數，完成單樣本的 t 檢定工作。

```javascript
var t1 = { // 單樣本 T 檢定 t = (X-mu)/(S/sqrt(n))
  h:function(o) { return "H0:mu"+o.op+o.mu; }, 
  o2q:function(o) {
    var x = o.x, n = x.length;
	// t=(X-mu)/(sd/sqrt(n))
    var t = (R.mean(x)-o.mu)/(R.sd(x)/Math.sqrt(n)); 
    return R.pt(t, n-1);
  },
  // P(X-mu/(S/sqrt(n))<t) = q ; 信賴區間 P(T<q)
  // P(mu > X-t*S/sqrt(n)) = q ; 這反而成了右尾檢定，所以左尾與右尾確實會反過來
  q2p:function(q, o) {
    var x = o.x, n = x.length;
    return R.mean(x) + R.qt(q, n-1) * R.sd(x) / Math.sqrt(n);
  },
  df:function(o) { return o.x.length-1; }
}
...
R.ttest = function(o) { 
  var t;
  if (typeof o.y === "undefined") {
    o.name = "ttest(X)";
    o.D = t1;
	t = R.test(o);
	t.mean = R.mean(o.x);
	t.sd   = R.sd(o.x);
  } else {
```

目前我們已經在 R.js 中加入了大部份的機率分布、還有基本的「有母數」檢定功能，之後會再加入「無母數檢定的功能」。

另外、我們也已經加入了基本的圖表繪製功能在 G.js 當中，於是形成了 jsLab 專案的基本架構，希望之後還能找到更多更棒的 JavaScript 科學計算函式庫，讓 JavaScript 語言也能 成為科學計算的良好平台。

### 參考文獻
* <https://github.com/jstat/jstat>
* <https://github.com/ccckmit/jslab/tree/gh-pages>
