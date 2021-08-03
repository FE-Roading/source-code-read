## jQuery的源码阅读

### [官网地址](https://jquery.com/)

### [Github](https://github.com/jquery/jquery)

### 1、_定义及导出
主要是利用自执行及闭包特性来完成jQuery定义的运行，并针对不同的运行环境进行特定的导出。
```js
// global 为全局对象，在浏览器或者webview环境下，指向window；Node环境指向global
// factory 为jQuery核心工厂函数：noGlobal表明是否向window中注入$、jQuery对象。 通过script导入时，会默认挂载
( function( global, factory ) {

	"use strict";
	// 支持CommonJS模块的导入导出规范「NODE OR WEBPACK」
	if ( typeof module === "object" && typeof module.exports === "object" ) {
		 // global.document 存在说明是WEBPACK环境「global->window」
		module.exports = global.document ?
			factory( global, true ) :
			// 在其余的没有window的环境下，导出一个函数，后期执行函数，如果可以传递一个window进来，也能正常使用，否则报错。
			function( w ) {
				if ( !w.document ) {
					throw new Error( "jQuery requires a window with a document" );
				}
				return factory( w );
			};
	} else {
		//浏览器或者webview中基于<script>直接导入的
		factory( global );
	}

// 如果是在浏览器或者webview环境下运行JS，则传入window对象；如果是在Node环境下执行，则传入global或者当前模块。
} )( typeof window !== "undefined" ? window : this, function( window, noGlobal ) {})
```

2、模块及全局变量的导出

```js
//factory 是jQuery闭包中传入函数的实参函数
function factory( window, noGlobal){
	// 	直接SCRIPT导入时， window:window  noGlobal:undefined
    // 	基于WEBPACK中的CommonJS/ES6Module规范导入的JQ	window:window  noGlobal:true
	"use strict";
	
	// 支持AMD模块化思想,不常用
    if ( typeof define === "function" && define.amd ) {
        define( "jquery", [], function() {
            return jQuery;
        } );
    }

    var _jQuery = window.jQuery, _$ = window.$;
    //多库共存解决权限冲突,导入jQuery（代码执行前）解决权限冲突
    jQuery.noConflict = function( deep ) {
        if ( window.$ === jQuery ) {
            window.$ = _$;
        }

        if ( deep && window.jQuery === jQuery ) {
            window.jQuery = _jQuery;
        }

        return jQuery;
    };

    //浏览器直接导入处理 => $() / jQuery() 都是把内部的jQuery方法执行
    if ( typeof noGlobal === "undefined" ) {
        window.jQuery = window.$ = jQuery;
    }

    // 基于WEBPACK处理 module.export = jQuery; 导入方式：import jQuery from 'jQuery';
    return jQuery;
};
```
