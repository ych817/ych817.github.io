# Hugo FixIt 主题配置


<!-- {{< music server="netease" type="album" id="250926226" autoplay="true" loop="all" list-folded=false >}} -->

## 数学公式 MathJax 配置

### 配置流程

下述操作参考链接[在此](https://github.com/hugo-fixit/FixIt/issues/574)[^1] 。

1. 首先将 `./themes/FixIt/layouts/partials/assets.html` 
   复制到 `./layouts/partials/assets.html` ,   
   然后将其中与 KaTeX 中的内容删除 , 并把下述内容粘贴进该文件中 : 
   
   ```html {data-open=false}
   {{- /* MathJax */ -}}
   {{- $math := .Scratch.Get "math" -}}
   {{- if $math.enable -}}

   <script>
   MathJax = {
       loader: {
       load: ['[custom]/xypic.js'],
       paths: { custom: 'https://cdn.jsdelivr.net/gh/sonoisa/XyJax-v3@3.0.1/build/' }
       },
       tex: {
       packages: { '[+]': ['xypic'] },
       inlineMath: [['\\(', '\\)'], ['$', '$']],
       displayMath: [['\\[', '\\]'], ['$$', '$$']],
       processEscapes: true,
       // processEnvironments: true,
       useLabelIds: true
       },
       options: {
       skipHtmlTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
       enableMenu: false,
       }
   };
   </script>   
   <script type="text/javascript" id="MathJax-script" async
   src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml-full.js"></script>
   <style>
      mjx-container {
         display: inline-grid;
         overflow-x: auto;
         overflow-y: hidden;
         max-width: 100%;
      }
      mjx-container svg {
         display: inline-grid !important;
         overflow-x: auto;
         overflow-y: hidden;
         max-width: 100%;
      }
   </style>
   {{- end -}}
   ```

2. 为使行内公式能正确加载 , 我们还需允许 [`passthrough render hooks`](https://gohugo.io/render-hooks/passthrough/)[^2] 。
   具体配置参考[官方文档](https://gohugo.io/render-hooks/passthrough/)[^2] 。


### 测试结果

-  行间公式测试 :

   $$
   \rm\LaTeX\text{ definitions are here.}
   \let\ph=\phantom
   \let\vph=\vphantom
   \def\pila{\vph{fg}}
   $$

-  行内公式测试 : 
   
   $c = \pm\sqrt{a^2 + b^2}$ 和 
   $\displaystyle f(x)=\int_{-\infty}^{\infty} \hat{f}(\xi) e^{2 \pi i \xi x} d \xi$  以及 
   $x^2 +y^2=1$ 、
   $
   \begin{array}{|l|l|}\hline
   \cellcolor{LightGreen} \rm Hello & \rm World\pila \\\hline
   \end{array}
   $ 。
   $
   % \begin{array}{l}
   % \qquad~~\begin{xy}0;p+/r+1cm/:
   % (-1,+.75)*++<3pt>[F-:<10pt>:Gray][=stateOld]{}="MaI1";
   % (-1,-.75)*[stateOld]{}="MaIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MaL"*[gray]!L{\small I_a};"MaL".
   % "MaIn"."MaI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaI";
   % (+1,+.75)*++<3pt>[F=:<10pt>:Gray][=stateOldFin]{}="MaF1";
   % (+1,-.75)*[stateOldFin]{}="MaFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MaR"*[gray]!R{\small F_a};"MaR".
   % "MaFn"."MaF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaF";
   % "MaL";"MaR"**@{}?(.5)*[gray]{\small M_a}.
   % "MaF"."MaI"*+<4pt>[F-:<5pt>]\frm{}="Ma";
   % % -----------------------------------------------------------
   % "MaI1"!L;"MaIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@[gray]@{->}@`{c+/l+1.5pc/,c}c};
   % % -----------------------------------------------------------
   % (5.15,0);p+/r+1cm/:
   % % -----------------------------------------------------------
   % (-1,+.75)*[stateOld]{}="MbI1";
   % (-1,-.75)*[stateOld]{}="MbIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MbL"*[gray]!L{\small I_b};"MbL".
   % "MbIn"."MbI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbI";
   % (+1,+.75)*[stateOldFin]{}="MbF1";
   % (+1,-.75)*[stateOldFin]{}="MbFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MbR"*[gray]!R{\small F_b};"MbR".
   % "MbFn"."MbF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbF";
   % "MbL";"MbR"**@{}?(.5)*[gray]{\small M_b}.
   % "MbF"."MbI"*+<4pt>[F-:<5pt>]\frm{}="Mb";
   % % -----------------------------------------------------------
   % "MbI1"!L;"MbIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@[gray]@{->}@`{c+/l+1.5pc/,c}c};
   % \end{xy}
   % \\
   % \begin{xy}p+/r+1cm/:p+/u+1cm/::
   % (-2.5,-1.375)*++<3pt>[F-:<10pt>][=stateNew]{}="p",
   % {\ar@{.>}@`{c+/l+3pc/,"p"}"p"};
   % % -------------------------------------------------------
   % (-1,+.75)*++<3pt>[F-:<10pt>:Gray][=stateOld]{}="MaI1";
   % (-1,-.75)*[stateOld]{}="MaIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MaL"*[gray]!L{\small I_a};"MaL".
   % "MaIn"."MaI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaI";
   % (+1,+.75)*++<3pt>[F=:<10pt>:Gray][=stateOldFin]{}="MaF1";
   % (+1,-.75)*[stateOldFin]{}="MaFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MaR"*[gray]!R{\small F_a};"MaR".
   % "MaFn"."MaF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaF";
   % "MaL";"MaR"**@{}?(.5)*[gray]{\small M_a}.
   % "MaF"."MaI"*+<4pt>[F-:<5pt>]\frm{}="Ma";
   % % -----------------------------------------------------------
   % (0,-2.75);p+/r+1cm/:
   % (-1,+.75)*[stateOld]{}="MbI1";
   % (-1,-.75)*[stateOld]{}="MbIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MbL"*[gray]!L{\small I_b};"MbL".
   % "MbIn"."MbI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbI";
   % (+1,+.75)*[stateOldFin]{}="MbF1";
   % (+1,-.75)*[stateOldFin]{}="MbFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MbR"*[gray]!R{\small F_b};"MbR".
   % "MbFn"."MbF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbF";
   % "MbL";"MbR"**@{}?(.5)*[gray]{\small M_b}.
   % "MbF"."MbI"*+<4pt>[F-:<5pt>]\frm{}="Mb";
   % % -----------------------------------------------------------
   % "MaI1"!L;"MaIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@{<.}@`{c+/l+2pc/,"p"}_(.25){\scriptstyle \epsilon}"p"};
   % "MbI1"!L;"MbIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@{<.}@`{c+/l+2pc/,"p"}^(.25){\scriptstyle \epsilon}"p"};
   % \end{xy} \qquad \begin{xy}p+/r+1cm/:p+/u+1cm/::
   % (+2.5,0)*++<3pt>[F-:<10pt>][=stateNew]{}="p";
   % % -----------------------------------------------------------
   % (-1,+.75)*++<3pt>[F-:<10pt>:Gray][=stateOld]{}="MaI1";
   % (-1,-.75)*[stateOld]{}="MaIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MaL"*[gray]!L{\small I_a};"MaL".
   % "MaIn"."MaI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaI";
   % (+1,+.75)*++<3pt>[F=:<10pt>:Gray][=stateOldFin]{}="MaF1";
   % (+1,-.75)*[stateOldFin]{}="MaFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MaR"*[gray]!R{\small F_a};"MaR".
   % "MaFn"."MaF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaF";
   % "MaL";"MaR"**@{}?(.5)*[gray]{\small M_a}.
   % "MaF"."MaI"*+<4pt>[F-:<5pt>]\frm{}="Ma";
   % % -----------------------------------------------------------
   % (5,0);p+/r+1cm/:
   % (-1,+.75)*[stateOld]{}="MbI1";
   % (-1,-.75)*[stateOld]{}="MbIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MbL"*[gray]!L{\small I_b};"MbL".
   % "MbIn"."MbI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbI";
   % (+1,+.75)*[stateOldFin]{}="MbF1";
   % (+1,-.75)*[stateOldFin]{}="MbFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MbR"*[gray]!R{\small F_b};"MbR".
   % "MbFn"."MbF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MbF";
   % "MbL";"MbR"**@{}?(.5)*[gray]{\small M_b}.
   % "MbF"."MbI"*+<4pt>[F-:<5pt>]\frm{}="Mb";
   % % -----------------------------------------------------------
   % "MaI1"!L;"MaIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@[gray]@{->}@`{c+/l+1.5pc/,c}c};
   % % -----------------------------------------------------------
   % "MaF1"!R;"MaFn"!R**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@{.>}@`{c+/r+1pc/,"p"}^(.25){\scriptstyle \epsilon}"p"};
   % "MbI1"!L;"MbIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{\ar@{<.}@`{c+/l+1pc/,"p"}^(.25){\scriptstyle \epsilon}"p"};
   % % Fig 2 : ---------------------------------------------------
   % 0;p+/r+1cm/:(0,-2.75);
   % p+/r+1cm/:p+/u+1cm/::
   % (0,-2)*++<3pt>[F=:<10pt>][=stateNewFin]{}="p",
   % {\ar@[gray]@{->}@`{"p"+/d+3pc/,"p"}"p"};
   % % -----------------------------------------------------------
   % (-1,+.75)*++<3pt>[F-:<10pt>:Gray][=stateOld]{}="MaI1";
   % (-1,-.75)*[stateOld]{}="MaIn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}}
   % ?(.5)="MaL"*[gray]!L{\small I_a};"MaL".
   % "MaIn"."MaI1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaI";
   % (+1,+.75)*++<3pt>[F=:<10pt>:Gray][=stateOldFin]{}="MaF1";
   % (+1,-.75)*[stateOldFin]{}="MaFn"**@{}
   % @={?(.35),?(.5),?(.65)} @@{*{\cdot}},
   % ,?(.5)="MaR"*[gray]!R{\small F_a};"MaR".
   % "MaFn"."MaF1"*+<8pt,5pt>[F:<3pt>]\frm{}="MaF";
   % "MaL";"MaR"**@{}?(.5)*[gray]{\small M_a}.
   % "MaF"."MaI"*+<4pt>[F-:<5pt>]\frm{}="Ma";
   % % -----------------------------------------------------------
   % "MaF1"!R;"MaFn"!R**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{
   % \PATH ~={**@{.}}~<{^>>*{\scriptstyle \epsilon}}~>{|>*\dir{>}}
   % `r/5pt c+/r1.5pc/`d/5pt "p" "p"
   % };
   % "MaI1"!L;"MaIn"!L**@{}
   % @={?(0),?(.25),?(.5),?(.75),?(1)} 
   % @@{
   % \PATH ~={**@{.}}~<{|<*\dir{<}_>>*{\scriptstyle \epsilon}}
   % `l^/5pt c+/l1.5pc/`d/5pt "p" "p"
   % };
   % \end{xy}
   % \end{array}
   $

## 短代码 `svg-file` 配置

该文件便于在博文中插入 `.svg` 文件 。与普通插入图片的方式有所不同的是 : 该方法可允许 `.svg` 文件在黑暗模式下正常显示 。

### 配置流程

1. 创建文件 `./layouts/shortcodes/svg-file.html` 并将下述内容粘贴进文件中即可 。

   ```html {title="svg-file.html", data-open=false}
   {{ $svg := .Get "src" }}
   {{ $svgPath := path.Join "static" $svg }}
   {{ if fileExists $svgPath }}
     <div class="svg-container">
       {{ readFile $svgPath | safeHTML }}
     </div>
     <style>
       .svg-container {
         display: inline-block;
         width: 100%;
         max-width: {{ with .Get "max-width" }}{{ . }}{{ else }}100%{{ end }};
         height: auto;
         overflow-x: auto;
         overflow-y: hidden;
       }
       .svg-container svg {
         width: auto;
         height: auto;
         {{ with .Get "fill" }} fill: {{ . }};{{ end }}
         {{ with .Get "stroke" }} stroke: {{ . }};{{ end }}
       }
     </style>
   {{ else }}
     {{ errorf "SVG file not found: %s" $svgPath }}
   {{ end }}
   ```

### 测试结果

<!-- {{< svg-file src="ConfigSoftware/test.svg" >}} -->

{{< svg-file src="EOPL/Xer2.11.svg" >}}

## 短代码 `jsxgraph` 配置

一个类似 Geogebra 的小工具 。

### 配置流程

1. 首先在 `./layouts/partials/assets.html` 中加入下述代码 : 

   ```html {data-open=false}
   {{- /* JSXGraph */ -}}
   {{- if .Store.Get "hasJSXGraph" -}}
     <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/jsxgraph/distrib/jsxgraph.css" />
     {{- $source := $cdn.jsxgraph | default "https://cdn.jsdelivr.net/npm/jsxgraph/distrib/jsxgraphcore.js" -}}
     {{- dict "Source" $source "Fingerprint" $fingerprint "Defer" true | dict "Scratch" .Scratch "Data" | partial "scratch/script.html" -}}
   {{- end -}}
   ```

2. 然后创建 `./layouts/partials/shortcodes/jsxgraph.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="jsxgraph.html", data-open=false}
   {{- $options := .Get "width" | default (.Get 0) | dict "width" -}}
   {{- $options = .Get "height" | default (.Get 1) | dict "height" | merge $options -}}
   {{- $options = .Get "renderer" | default "svg" | dict "renderer" | merge $options -}}
   {{- dict "Options" $options "Inner" .Inner | partial "plugin/jsxgraph.html" -}}
   {{- .Page.Store.Set "hasJSXGraph" true -}}
   ```

3. 接着创建 `./layouts/partials/plugin/jsxgraph.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="jsxgraph.html", data-open=false}
   {{- /* JSXGraph support for code fences extended and shortcodes. */ -}}
   {{- $width := .Options.width | default "100%" -}}
   {{- $height := .Options.height | default "20rem" -}}
   {{- $id := printf "jsxgraph-%d" now.UnixNano -}}
   {{- $attrs := printf `id="%s" style="width: %v; height: %v; margin: 2rem 0; border: 1px solid; border-radius: 10px; overflow: hidden; page-break-inside: avoid; break-inside: avoid;"` $id $width $height -}}
   <div class="jsxgraph" {{ $attrs | safeHTMLAttr }}></div>
   <script>
   (function(){
     var container = document.getElementById('{{ $id }}');
   
     function runUserScript() {
       // 替换掉 initBoard(...) 中写死的 ID，不管是 box / jxgbox / myboard
       {{ .Inner | replaceRE "initBoard\\(\\s*['\"][^'\"]+['\"]" (printf "initBoard('%s'" $id) | safeJS }}
     }
   
     if (document.readyState === 'loading') {
       document.addEventListener("DOMContentLoaded", runUserScript);
     } else {
       runUserScript();
     }
   })();
   </script>
   ```

4. 最后创建 `./layouts/_default/_markup/render-codeblock-jsxgraph.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="render-codeblock-jsxgraph.html", data-open=false}
   {{- dict "Options" .Attributes "Inner" .Inner | partial "plugin/jsxgraph.html" -}}
   {{- .Page.Store.Set "hasJSXGraph" true -}}
   ```

### 测试结果

> [!tip]
>
> 如果比例失调 , 可以考虑在 `JSXGraph.initBoard` 中加入 `keepaspectration: true` 。比如
> 
> ```javascript {data-open=false}
> var board = JXG.JSXGraph.initBoard('jxgbox', {
>   boundingbox: [-1, 1, 1, -1], 
>   keepaspectratio: true
> });
> ```

{{< raw >}}
<p>Function term in <em>(x,y)</em>:
  <br>
  <input type"text" id="plot_input" value="sin(x * y * r / 4)" style="width: 10em; margin-top: 1rem;">
  <br>
  <input type="button" id="plot_start" value="plot" onClick="start_plot();" style="width: 4em; margin-top: 1rem;">
  <input type="button" id="plot_reset" value="reset" onClick="reset_plot();" style="width: 4em; margin-top: 1rem;">
  <br><br>
  (The function might also depend on the slider value <em>r</em>.)
</p>
{{< /raw >}}

```jsxgraph {}
// Define the id of your board in 3dplotter

// Define some colors, optimized for color blinds
var colors = [JXG.palette.blue,
        JXG.palette.red,
        JXG.palette.green,
        JXG.palette.black,
        JXG.palette.purple,
        JXG.palette.yellow,
        JXG.palette.skyblue
    ],
    cnt = 0;

// We make board a global variable.
// This is necessary just on this platform.
window.board = JXG.JSXGraph.initBoard('3dplotter', {
    boundingbox: [-8, 8, 8, -8],
    keepaspectratio: true,
    axis: false,
    pan: {
        enabled: false
    }
});

var box = [-5, 5];
var view = board.create('view3d', [
    [-6, -3],
    [8, 8],
    [box, box, box]
], {
    xPlaneRear: {visible: false}, yPlaneRear: {visible: false}
});

// Create a slider which might be used in the function term
// using the name 'r'
var slider = board.create('slider', [[-7, -6], [5, -6], [-3, 1, 4]], { name: 'r' });

// Handler for button 'plot'
window.start_plot = function() {
    var txt = document.getElementById('plot_input').value,
        f = board.jc.snippet(txt, true, 'x,y', true), // JessieCode parsing
        el;

    el = view.create('functiongraph3d', [f, box, box], {
        strokeColor: colors[cnt],
        stepsU: 50,
        stepsV: 50,
        strokeWidth: 0.5
    });
    board.update();
    // Cycle through the color array
    cnt = (cnt + 1) % colors.length;
}

// Handler for button 'reset'
window.reset_plot = function() {
    // Complete restart of the construction
    JXG.JSXGraph.freeBoard(board);
    board = JXG.JSXGraph.initBoard('3dplotter', {
        boundingbox: [-8, 8, 8, -8],
        keepaspectratio: true,
    });
    view = board.create('view3d', [
        [-6, -3],
        [8, 8],
        [box, box, box]
    ]);

    slider = board.create('slider', [
        [-7, -6],
        [5, -6],
        [-3, 1, 4]
    ], {
        name: 'r'
    });
    cnt = 0;
}
```

```jsxgraph {}
var brd = JXG.JSXGraph.initBoard('box', {boundingbox: [-2, 2, 2, -2], keepaspectratio: true});
solveQ2 = function(x1,x2,x3,off) {
    var a, b, c, d;
    a = 0.5;
    b = -(x1+x2+x3);
    c = x1*x1+x2*x2+x3*x3-0.5*(x1+x2+x3)*(x1+x2+x3)-off;
    d = b*b-4*a*c;
    if (Math.abs(d)<0.00000001) d = 0.0;
    return [(-b+Math.sqrt(d))/(2.0*a),(-b-Math.sqrt(d))/(2.0*a)];
}   
    
a = brd.create('segment',[[0,0],[2,0]],{visible:false});
p1 = brd.create('glider',[1.3,0,a],{name:'Drag me'});
b0 = -0.5;

c0 = brd.create('circle',[[0,0],Math.abs(1.0/b0)],{strokeWidth:1});
c1 = brd.create('circle',[p1,function(){return 2-p1.X();}],{strokeWidth:1});

c2 = brd.create('circle',[[function(){return p1.X()-2;},0],function(){return p1.X();}],{strokeWidth:1});
c0.curvature = function(){ return b0;}; // constant
c1.curvature = function(){ return 1/(2-p1.X());};
c2.curvature = function(){ return 1/(p1.X());};

thirdCircleX = function() {
    var b0,b1,b2,x0,x1,x2, b3,bx3;
    b0 = c0.curvature();
    b1 = c1.curvature();
    b2 = c2.curvature();
    x0 = c0.midpoint.X();
    x1 = c1.midpoint.X();
    x2 = c2.midpoint.X();

    b3 = solveQ2(b0,b1,b2,0);
    bx3 = solveQ2(b0*x0,b1*x1,b2*x2,2);
    return bx3[0]/b3[0];
}
thirdCircleY = function() {
    var b0,b1,b2,y0,y1,y2, b3,by3;
    b0 = c0.curvature();
    b1 = c1.curvature();
    b2 = c2.curvature();
    y0 = c0.midpoint.Y();
    y1 = c1.midpoint.Y();
    y2 = c2.midpoint.Y();

    b3 = solveQ2(b0,b1,b2,0);
    by3 = solveQ2(b0*y0,b1*y1,b2*y2,2);
    return by3[0]/b3[0];
}
thirdCircleRadius = function() {
    var b0,b1,b2, b3,bx3,by3;
    b0 = c0.curvature();
    b1 = c1.curvature();
    b2 = c2.curvature();
    b3 = solveQ2(b0,b1,b2,0);
    return 1.0/b3[0];
}

c3 = brd.create('circle',[[thirdCircleX,thirdCircleY],thirdCircleRadius],{strokeWidth:1});
c3.curvature = function(){ return 1.0/this.radius;};

otherCirc = function(circs,level) {
    var c, fx,fy,fr;
    if (level<=0) return;
    fx = function() {
        var b,x,i;
        b = [];
        x = [];
        for (i=0;i<4;i++) {
            b[i] = circs[i].curvature();
            x[i] = circs[i].midpoint.X();
        }
    
        b[4] = 2*(b[0]+b[1]+b[2])-b[3];
        x[4] = (2*(b[0]*x[0]+b[1]*x[1]+b[2]*x[2])-b[3]*x[3])/b[4];
        return x[4];
    }
    fy = function() {
        var b,y,i;
        b = [];
        y = [];
        for (i=0;i<4;i++) {
            b[i] = circs[i].curvature();
            y[i] = circs[i].midpoint.Y();
        }
    
        b[4] = 2*(b[0]+b[1]+b[2])-b[3];
        y[4] = (2*(b[0]*y[0]+b[1]*y[1]+b[2]*y[2])-b[3]*y[3])/b[4];
        return y[4];
    }
    fr = function() {
        var b,i;
        b = [];
        for (i=0;i<4;i++) {
            b[i] = circs[i].curvature();
        }
        b[4] = 2*(b[0]+b[1]+b[2])-b[3];
        if (isNaN(b[4])) {
            return 1000.0;
        } else {
            return 1/b[4];
        }
    }
    c = brd.create('circle',[[fx,fy],fr],{strokeWidth:1,name:'',
                 fillColor:JXG.hsv2rgb(50*level,0.8,0.8),highlightFillColor:JXG.hsv2rgb(50*level,0.5,0.8),fillOpacity:0.5,highlightFillOpacity:0.5});
    c.curvature = function(){ return 1/this.radius;};

    // Recursion
    otherCirc([circs[0],circs[1],c,circs[2]],level-1);
    otherCirc([circs[0],circs[2],c,circs[1]],level-1);
    otherCirc([circs[1],circs[2],c,circs[0]],level-1);
    return c;
}

//-------------------------------------------------------
brd.suspendUpdate();
level = 4;
otherCirc([c0,c1,c2,c3],level);
otherCirc([c3,c1,c2,c0],level);
otherCirc([c0,c2,c3,c1],level);
otherCirc([c0,c1,c3,c2],level);
brd.unsuspendUpdate();
```

## 短代码 `mathbox` 配置

高配版的数学绘图工具 。

> [!warning]
>
> 由于用到了 `three.js` , 有些图像可能无法正常加载 。

### 配置流程

1. 首先在 `./layouts/partials/assets.html` 中加入下述代码 : 
   
   ```html {data-open=false}
   {{- /* MathBox */ -}}
   {{- if .Store.Get "hasMathBox" -}}
     <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.5.1/katex.min.css"/>
     <script src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.5.1/katex.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/three@0.137.0/build/three.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/three@0.137.0/examples/js/controls/OrbitControls.js"></script>
     {{- $source := $cdn.mathboxJS | default "https://cdn.jsdelivr.net/npm/mathbox@latest/build/bundle/mathbox.js" -}}
     <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/mathbox@latest/build/mathbox.css" />
     {{- dict "Source" $source "Fingerprint" $fingerprint "Defer" false | dict "Scratch" .Scratch "Data" | partial "scratch/script.html" -}}
   {{- end -}}
   ```

2. 然后创建 `./layouts/partials/shortcodes/mathbox.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="mathbox.html", data-open=false}
   {{- $options := .Get "width" | default (.Get 0) | dict "width" -}}
   {{- $options = .Get "height" | default (.Get 1) | dict "height" | merge $options -}}
   {{/*  {{- $options = .Get "renderer" | default "svg" | dict "renderer" | merge $options -}}  */}}
   {{- dict "Options" $options "Inner" .Inner | partial "plugin/mathbox.html" -}}
   {{- .Page.Store.Set "hasMathBox" true -}}
   ```

3. 接着创建 `./layouts/partials/plugin/mathbox.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="mathbox.html", data-open=false}
   {{- /* MathBox support for code fences extended and shortcodes. */ -}}
   {{- $width := .Options.width | default "100%" -}}
   {{- $height := .Options.height | default "20rem" -}}
   {{- $id := printf "mathbox-%d" now.UnixNano -}}
   
   <div id="{{ $id }}" class="mathbox" style="width: {{ $width }}; height: {{ $height }}; margin: 2rem 0; border: 1px solid; border-radius: 10px; overflow: hidden; position: relative; page-break-inside: avoid; break-inside: avoid;"></div>
   
   <script>
   (function(){
     const container = document.getElementById('{{ $id }}');
   
     const userScript = function(container) {
       // 注入监控：等待用户 new dat.GUI() 并把它移到容器内
       const oldAppendChild = document.body.appendChild;
       document.body.appendChild = function (el) {
         if (el.classList?.contains('dg')) {
           container.appendChild(el);  // 强制把 GUI 控件放进你想要的地方
           el.style.position = 'absolute';
           el.style.top = '1rem';
           el.style.right = '1rem';
           return el;
         }
         return oldAppendChild.call(this, el);
       };
       {{ .Inner | replaceRE "MathBox\\.mathBox\\s*\\(\\s*\\{" (
        printf "MathBox.mathBox({ element: container,"
        ) | safeJS }}
     };
   
     if (document.readyState === 'loading') {
       document.addEventListener("DOMContentLoaded", () => userScript(container));
     } else {
       userScript(container);
     }
   })();
   </script>
   ```

4. 最后创建 `./layouts/_default/_markup/render-codeblock-mathbox.html` 并将下述代码粘贴进文件中即可 。

   ```html {title="render-codeblock-mathbox.html", data-open=false}
   {{- dict "Options" .Attributes "Inner" .Inner | partial "plugin/mathbox.html" -}}
   {{- .Page.Store.Set "hasMathBox" true -}}
   ```

### 测试结果

{{< raw >}}
<script type="application/glsl" id="rotate4D">
uniform float cos1;
uniform float sin1;
uniform float cos2;
uniform float sin2;
uniform float cos3;
uniform float sin3;
uniform float cos4;
uniform float sin4;

vec4 getRotate4D(vec4 xyzw, inout vec4 stpq) {
  xyzw.xy = xyzw.xy * mat2(cos1, sin1, -sin1, cos1);
  xyzw.zw = xyzw.zw * mat2(cos2, sin2, -sin2, cos2);
  xyzw.xz = xyzw.xz * mat2(cos3, sin3, -sin3, cos3);
  xyzw.yw = xyzw.yw * mat2(cos4, sin4, -sin4, cos4);

  return xyzw;
}
</script>
{{< /raw >}}

```mathbox
mathbox = MathBox.mathBox({
  plugins: ["core", "controls", "cursor"],
  controls: {
    klass: THREE.OrbitControls,
  },
});
three = mathbox.three;

three.camera.position.set(2.3, 1, 2);
three.renderer.setClearColor(new THREE.Color(0xffffff), 1.0);

time = 0;
angle = 0;
deform = 0;
skew = 0;

fa = 6;
fb = 4;

three.on("update", function () {
  time = three.Time.clock * 0.6;
  angle = wobbler(time / 5) * 0.6 + MathBox.π / 4 + 0.2;
  skew = wobbler(time / 16.1) * 0.25;
  deform = wobbler(time / 7.1);

  if (deform == 0) {
    fa = (Math.ceil(Math.random() * 2) + 1) * 2;
    fb = (Math.ceil(Math.random() * 2) + 1) * 2;
  }

  deform *= cosineEase(time - 2);
});

clamp = function (x, a, b) {
  return Math.max(a, Math.min(b, x));
};

cosineEase = function (t) {
  return 0.5 - 0.5 * Math.cos(clamp(t, 0, 1) * MathBox.π);
};

wobbler = function (t) {
  t = Math.sin(
    (Math.min(1, Math.max(-1, 0.7 * Math.asin(Math.cos(MathBox.π * t)))) *
      MathBox.τ) /
      4
  );
  return t * 0.5 - 0.5;
};

hopf = function (emit, ϕ, θ, γ) {
  η =
    θ / 2 +
    (Math.cos(ϕ * 2 + time) * skew + Math.cos(ϕ * fa)) * 0.15 * deform;
  ξ1 = ϕ + Math.cos(γ * fb + time) * 0.25 * deform;
  ξ2 = γ;

  sum = ξ1 + ξ2;
  diff = ξ1 - ξ2;

  s = Math.sin(η);
  c = Math.cos(η);

  x = Math.cos(sum) * s;
  y = Math.sin(sum) * s;
  z = Math.cos(diff) * c;
  w = Math.sin(diff) * c;

  emit(z, y, w, x);
};

view = mathbox
  .unit({
    scale: 500,
  })
  .stereographic4({
    range: [
      [-4, 4],
      [-4, 4],
      [-4, 4],
      [-1, 1],
    ],
    scale: [4, 4, 4, 1],
  });

view.area({
  rangeX: [-MathBox.π, MathBox.π],
  rangeY: [-MathBox.π / 2, MathBox.π / 2],
  width: 127,
  height: 16,
  centeredX: false,
  centeredY: true,
  expr: function (emit, x, y, i, j) {
    ϕ = y;
    θ = angle;
    hopf(emit, ϕ, θ, x);
  },
  channels: 4,
});
view.line({
  color: 0x3080ff,
  zBias: 10,
  width: 3,
});

view.area({
  rangeX: [-MathBox.π, MathBox.π],
  rangeY: [-0.6, 0.6],
  width: 127,
  height: 63,
  expr: function (emit, x, y, i, j) {
    ϕ = y + time;
    θ = angle;
    hopf(emit, ϕ, θ, x);
  },
  channels: 4,
});
view.surface({
  shaded: true,
  color: 0x3080ff,
  zBias: -5,
});

view.area({
  rangeX: [-MathBox.π, MathBox.π],
  rangeY: [-MathBox.π / 2, MathBox.π / 2],
  width: 127,
  height: 63,
  expr: function (emit, x, y, i, j) {
    ϕ = y;
    θ = angle;
    hopf(emit, ϕ, θ, x);
  },
  channels: 4,
});
view.surface({
  shaded: true,
  color: 0xffffff,
  opacity: 0.5,
  zBias: -10,
});
```

```mathbox
var π = Math.PI;
var mathbox = MathBox.mathBox({
  plugins: ["core", "controls", "cursor"],
  controls: {
    klass: THREE.OrbitControls,
  },
});

mathbox.three.camera.position.set(0, 0, 3);
mathbox.three.renderer.setClearColor(new THREE.Color(0xffffff), 1.0);

var red = 0xdf2000;
var green = 0x20a000;
var blue = 0x3090ff;

view = mathbox
  .set({
    scale: 500,
  })
  .clock({
    speed: 0.25,
  })
  .stereographic4()
  .shader(
    {
      code: "#rotate4D",
    },
    {
      cos1: function (t) {
        return Math.cos(t * 0.111);
      },
      sin1: function (t) {
        return Math.sin(t * 0.111);
      },
      cos2: function (t) {
        return Math.cos(t * 0.151 + 1);
      },
      sin2: function (t) {
        return Math.sin(t * 0.151 + 1);
      },
      cos3: function (t) {
        return Math.cos(t * 0.071 + Math.sin(t * 0.081));
      },
      sin3: function (t) {
        return Math.sin(t * 0.071 + Math.sin(t * 0.081));
      },
      cos4: function (t) {
        return Math.cos(t * 0.053 + Math.sin(t * 0.066) + 1);
      },
      sin4: function (t) {
        return Math.sin(t * 0.053 + Math.sin(t * 0.066) + 1);
      },
    }
  )
  .vertex();

var q1 = new THREE.Quaternion();
var q2 = new THREE.Quaternion();

view.area({
  rangeX: [-π / 2, π / 2],
  rangeY: [0, 2 * π],
  width: 129,
  height: 65,
  expr: function (emit, θ, ϕ, i, j) {
    q1.set(0, 0, Math.sin(θ), Math.cos(θ));
    q2.set(0, Math.sin(ϕ), 0, Math.cos(ϕ));
    q1.multiply(q2);
    emit(q1.x, q1.y, q1.z, q1.w);
  },
  live: false,
  channels: 4,
});
view.line({
  color: blue,
});

// ===========================================================================

view.area({
  rangeX: [-π / 2, π / 2],
  rangeY: [0, 2 * π],
  width: 129,
  height: 65,
  expr: function (emit, θ, ϕ, i, j) {
    q1.set(0, Math.sin(θ), 0, Math.cos(θ));
    q2.set(Math.sin(ϕ), 0, 0, Math.cos(ϕ));
    q1.multiply(q2);
    emit(q1.x, q1.y, q1.z, q1.w);
  },
  live: false,
  channels: 4,
});
view.line({
  color: green,
});

// ===========================================================================

view.area({
  rangeX: [-π / 2, π / 2],
  rangeY: [0, 2 * π],
  width: 129,
  height: 65,
  expr: function (emit, θ, ϕ, i, j) {
    q1.set(Math.sin(θ), 0, 0, Math.cos(θ));
    q2.set(0, 0, Math.sin(ϕ), Math.cos(ϕ));
    q1.multiply(q2);
    emit(q1.x, q1.y, q1.z, q1.w);
  },
  live: false,
  channels: 4,
});
view.line({
  color: red,
});

// ===========================================================================
```

## 将博文导出为带书签的 PDF

首先博文在导出为 PDF 的时候 , echarts , jsxgraph 和 mathbox 并不能跟随打印纸张的宽度而自适应 。

其次浏览器默认的打印功能不支持为导出的 PDF 添加书签 , 因此借鉴了一下了知乎上两篇分别关于 [Typora](https://zhuanlan.zhihu.com/p/652509474)[^4] 和 [Obsidian](https://zhuanlan.zhihu.com/p/653165462)[^5] 导出带书签 PDF 的文章 。另外也可以参考这篇 [Github Issue](https://github.com/Hopding/pdf-lib/issues/127)[^6] 。

### 配置流程

1. 创建文件 `./layouts/partials/single/print-to-pdf.html` 并将下述内容粘贴进文件中 :

   ```html {data-open=false}
   <!-- 引入 pdf-lib -->
   <script src="https://cdn.jsdelivr.net/npm/pdf-lib/dist/pdf-lib.min.js"></script>
   
   <div id="print-controls" style="text-align: center; margin: 2rem 0;">
     <button id="upload-btn" style="
       background-color: #444;
       color: white;
       padding: 0.6rem 1.2rem;
       border-radius: 8px;
       border: none;
       cursor: pointer;
       font-size: 0.95rem;
     ">点此上传本博文对应的 PDF 以自动为 PDF 添加书签</button>
     <input type="file" id="pdf-upload" accept="application/pdf" style="display: none;" />
   </div>
   
   <script>
     window.addEventListener("DOMContentLoaded", () => {
       document.getElementById("upload-btn").addEventListener("click", () => {
         document.getElementById("pdf-upload").click();
       });
       document.getElementById("pdf-upload").addEventListener("change", handlePdfUpload);
     });
   
     window.addEventListener("beforeprint", () => {
       const headers = document.querySelectorAll(".single-title, .content h2, .content h3, .content h4");
       headers.forEach((el, i) => {
         if (el.querySelector("a.md-print-anchor")) return;
         const anchor = document.createElement("a");
         anchor.href = `af://n${i}`;
         anchor.className = "md-header-anchor md-print-anchor";
         anchor.style.display = "inline-block";
         anchor.style.height = "0.1pt";
         anchor.style.width = "0.1pt";
         anchor.style.overflow = "hidden";
         anchor.style.color = "transparent";
         el.insertBefore(anchor, el.firstChild);
       });
     });
   
     window.addEventListener("afterprint", () => {
       document.querySelectorAll(".md-print-anchor").forEach(a => a.remove());
     });
   
     async function handlePdfUpload(e) {
       const file = e.target.files[0];
       if (!file) return alert("请先选择一个 PDF 文件");


       const arrayBuffer = await file.arrayBuffer();

       const pdfDoc = await PDFLib.PDFDocument.load(arrayBuffer);
       const pages = pdfDoc.getPages();
       const context = pdfDoc.context;
   
       const markerMap = new Map();
       for (let i = 0; i < pages.length; i++) {
         const annots = pages[i].node.Annots?.();
         if (!annots) continue;
         for (let j = 0; j < annots.size(); j++) {
           try {
             const ann = annots.lookup(j, PDFLib.PDFDict);
             const subtype = ann.get(PDFLib.PDFName.of("Subtype"));
             if (subtype?.asString() !== "/Link") continue;
             const action = ann.get(PDFLib.PDFName.of("A"));
             const uri = action?.get(PDFLib.PDFName.of("URI"))?.asString();
             const rect = ann.get(PDFLib.PDFName.of("Rect"))?.asRectangle();
             const match = /^af:\/\/(.+)$/.exec(uri);
             if (!match) continue;
             const anchorId = match[1];
             const y = rect.y;
             markerMap.set(anchorId, { pageIndex: i, y });
           } catch (err) {
             console.warn("annotation parse failed", err);
           }
         }
       }
   
       const headings = Array.from(document.querySelectorAll(".single-title, .content h2, .content h3, .content h4"))
         .map((el, i) => ({
           level: el.matches(".single-title") ? 1 : parseInt(el.tagName[1]),
           text: el.innerText.trim(),
           marker: `n${i}`
         }))
         .filter(h => markerMap.has(h.marker));
   
       if (headings.length === 0) {
         alert("未在 PDF 中找到任何 af:// 书签标记，请确认已插入锚点并打印。");
         return;
       }
   
       const outlineNodes = [];
       headings.forEach(({ text, level, marker }) => {
         const { pageIndex, y } = markerMap.get(marker);
         const page = pages[pageIndex];
         const destArray = context.obj([
           page.ref,
           PDFLib.PDFName.of("XYZ"),
           PDFLib.PDFNumber.of(0),
           PDFLib.PDFNumber.of(y),
           PDFLib.PDFNull
         ]);
         const dict = context.obj({
           Title: PDFLib.PDFHexString.fromText(text),
           Dest: destArray
         });
         const ref = context.register(dict);
         outlineNodes.push({ dict, ref, level, children: [] });
       });
   

       const stack = [], topLevel = [];
       outlineNodes.forEach(node => {
         while (stack.length > 0 && stack[stack.length - 1].level >= node.level) stack.pop();
         if (stack.length === 0) topLevel.push(node);
         else stack[stack.length - 1].children.push(node);
         stack.push(node);
       });
   
       function linkSiblings(nodes) {
         for (let i = 0; i < nodes.length; i++) {
           const current = nodes[i];
           if (i > 0) current.dict.set(PDFLib.PDFName.of("Prev"), nodes[i - 1].ref);
           if (i < nodes.length - 1) current.dict.set(PDFLib.PDFName.of("Next"), nodes[i + 1].ref);
           if (current.children.length > 0) {
             current.dict.set(PDFLib.PDFName.of("First"), current.children[0].ref);
             current.dict.set(PDFLib.PDFName.of("Last"), current.children[current.children.length - 1].ref);
             current.dict.set(PDFLib.PDFName.of("Count"), PDFLib.PDFNumber.of(current.children.length));
             linkSiblings(current.children);
           }
         }
       }
   
       linkSiblings(topLevel);
   
       const outlineDict = context.obj({
         Type: PDFLib.PDFName.of("Outlines"),
         First: topLevel[0].ref,
         Last: topLevel[topLevel.length - 1].ref,
         Count: PDFLib.PDFNumber.of(topLevel.length)
       });
   
       pdfDoc.catalog.set(PDFLib.PDFName.of("Outlines"), context.register(outlineDict));
       pdfDoc.catalog.set(PDFLib.PDFName.of("PageMode"), PDFLib.PDFName.of("UseOutlines"));
   
       const pdfBytes = await pdfDoc.save();
       const blob = new Blob([pdfBytes], { type: "application/pdf" });
       const link = document.createElement("a");
       link.href = URL.createObjectURL(blob);
       const originalName = file.name.replace(/\.pdf$/i, '');
       link.download = `${originalName}-带书签.pdf`;
   
       link.click();
     }
   </script>
   ```
   
2. 将文件 `./themes/FixIt/layouts/posts/single.html` 复制到 `./layouts/posts/single.html` 并将 `Custom block before post footer` 处修改为如下 : 

   ```html {data-open=false}
   {{- /* Custom block before post footer */ -}}
   {{- block "custom-post__footer:before" . }}
   {{ partial "single/print-to-pdf.html" . }} {{ end -}}
   ```

### 测试结果

页面底部会出现一个按钮 。

## 博客发布

[这里的链接](https://jianzhnie.github.io/post/hugo_site/)[^3]写的很清楚 。

[^1]: https://github.com/hugo-fixit/FixIt/issues/574

[^2]: https://gohugo.io/render-hooks/passthrough/

[^3]: https://jianzhnie.github.io/post/hugo_site/

[^4]: https://zhuanlan.zhihu.com/p/652509474

[^5]: https://zhuanlan.zhihu.com/p/653165462

[^6]: https://github.com/Hopding/pdf-lib/issues/127

---

> 作者: 沉积岩  
> URL: http://localhost:1313/posts/config-hugo_fixit/  

