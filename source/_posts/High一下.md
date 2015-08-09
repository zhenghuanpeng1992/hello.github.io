title: High一下
date: 2015-06-27 12:49:07
category: Hexo
tag: [High, 动画]
thumbnail: http://rocko-blog.qiniudn.com/High一下-1.jpg?imageView2/2/w/400/h/300/q/100
banner: http://rocko-blog.qiniudn.com/High一下-1.jpg
---


发现博客访问不了，原来是 GitCafe Pages 又挂了，而且 A 记录也不能用了，所以又得把博客迁回 Github 了。顺便把 `High一下` 优化了下。基本是直接拿耗子叔的 [酷壳](http://coolshell.cn/) 上的 High 一下来用的，存在点击重复播放的问题，而且歌曲只有一首，所以得加以优化下：加了几首歌点击随机播放；修复点击重复执行播放的问题，重新点击后动画重新开始。原来的代码右键 High 一下 `审查元素` 就看得到了，修改后的代码如下：

<!--more-->

``` JavaScript
javascript:(function() {
	function c() {
		var e = document.createElement("link");
		e.setAttribute("type", "text/css");
		e.setAttribute("rel", "stylesheet");
		e.setAttribute("href", f);
		e.setAttribute("class", l);
		document.body.appendChild(e)
	}
 
	function h() {
		var e = document.getElementsByClassName(l);
		for (var t = 0; t < e.length; t++) {
			document.body.removeChild(e[t])
		}
	}
 
	function p() {
		var e = document.createElement("div");
		e.setAttribute("class", a);
		document.body.appendChild(e);
		setTimeout(function() {
			document.body.removeChild(e)
		}, 100)
	}
 
	function d(e) {
		return {
			height : e.offsetHeight,
			width : e.offsetWidth
		}
	}
 
	function v(i) {
		var s = d(i);
		return s.height > e && s.height < n && s.width > t && s.width < r
	}
 
	function m(e) {
		var t = e;
		var n = 0;
		while (!!t) {
			n += t.offsetTop;
			t = t.offsetParent
		}
		return n
	}
 
	function g() {
		var e = document.documentElement;
		if (!!window.innerWidth) {
			return window.innerHeight
		} else if (e && !isNaN(e.clientHeight)) {
			return e.clientHeight
		}
		return 0
	}
 
	function y() {
		if (window.pageYOffset) {
			return window.pageYOffset
		}
		return Math.max(document.documentElement.scrollTop, document.body.scrollTop)
	}
 
	function E(e) {
		var t = m(e);
		return t >= w && t <= b + w
	}
 
	function S() {
		var e = document.getElementById("audio_element_id");
		if(e === null){
			e = document.createElement("audio");
			e.setAttribute("class", l);
			e.id = "audio_element_id";
			e.loop = false;
			e.addEventListener("canplay", function() {
			setTimeout(function() {
				x(k)
			}, 500);
			setTimeout(function() {
				N();
				p();
				for (var e = 0; e < O.length; e++) {
					T(O[e])
				}
			}, 15500)
		}, true);
		e.addEventListener("ended", function() {
			N();
			h()
		}, true);
		e.innerHTML = " <p>If you are reading this, it is because your browser does not support the audio element. We recommend that you get a new browser.</p> <p>";
		document.body.appendChild(e);
		} else {
            N();
        }

		e.src = i;
		e.play()
	}
 
	function x(e) {
		e.className += " " + s + " " + o
	}
 
	function T(e) {
		e.className += " " + s + " " + u[Math.floor(Math.random() * u.length)]
	}
 
	function N() {
		var e = document.getElementsByClassName(s);
		var t = new RegExp("\\b" + s + "\\b");
		for (var n = 0; n < e.length; ) {
			e[n].className = e[n].className.replace(t, "")
		}
	}
	
	var e = 30;
	var t = 30;
	var n = 350;
	var r = 350;

	/* 简单粗暴随机选一首，注意把歌曲位置添加上 */
	var number = Math.floor(Math.random() * 4);
	if(number == 0)
		var i = "your music file path";
	else if(number == 1)
		var i = "your music file path";
	else 
		var i = "your music file path";

	var s = "mw-harlem_shake_me";
	var o = "im_first";
	var u = ["im_drunk", "im_baked", "im_trippin", "im_blown"];
	var a = "mw-strobe_light";
	/* harlem-shake-style.css，替换成你的位置，也可以直接使用：//s3.amazonaws.com/moovweb-marketing/playground/harlem-shake-style.css */
	var f = "//s3.amazonaws.com/moovweb-marketing/playground/harlem-shake-style.css";
	var l = "mw_added_css";
	var b = g();
	var w = y();
	var C = document.getElementsByTagName("*");
	var k = null;
	for (var L = 0; L < C.length; L++) {
		var A = C[L];
		if (v(A)) {
			if (E(A)) {
				k = A;
				break
			}
		}
	}
	if (A === null) {
		console.warn("Could not find a node of the right size. Please try a different page.");
		return
	}
	c();
	S();
	var O = [];
	for (var L = 0; L < C.length; L++) {
		var A = C[L];
		if (v(A)) {
			O.push(A)
		}
	}
	})()
```

相关源码已上传 Github：[High](https://github.com/zhengxiaopeng/High)。
Tips：浏览器新建个书签，把代码放到链接里，保存，然后在每个页面里（如：[Weibo](http://weibo.com/)）点击都会有效果啦。