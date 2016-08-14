# T3
T3是一個JavaScript UI framework，主要的功能是讓程式碼更結構化。如果網站內的程式碼很雜亂的話，很適合用來整理散落在各處的程式碼(尤其是針對大型網站)。而整理方式就是將程式碼分成幾個部份：Application、Module、Serveice、Behavior等來處理。

## Modules
將頁面功能切成模組來運作，例如：頁面上的模組「test-module」，可能需要針對其他模組的反應來做出回應，因此有Message Handling；不同模組但有重覆的事情要做，則可用Behaviors；模組內的事件處理可使用onclick等。然後非UI而是資料處理的事情，則丟給Serveice即可。

	Box.Application.addModule('test-module', function(context) {
	  return {
	    messages: ['statechanged', 'statecompleted'],
	    onmessage: function(name, data) {
	      switch(name) {
	        case 'statechanged':
	          console.log('statechanged');
	          break;
	        case 'searchcomplete':
	          console.log('statecompleted: ' + data.numResults + ' tasks completed.');
	          break;
	      }
	    },
	    behaviors: ['get-info'],
	    init: function(){
	      console.log('module init');
	    },
	    onclick: function(event, element, elementType){
	      console.log('click');
		  context.getService('connectSomething').connect();
	    }
	  }
	});

## Serveice
Module與Behavior只要負責處理好UI，而資料的處理則交給Service，Serveice提供一個與伺服器間溝通的介面。例如：例如使用ajax取資料。例如：「getInfo」這個Service負責與Server端溝通，並將結果回傳給Behavior「getSearchResult」。

	Box.Application.addService('getInfo', function(application) {
	  return {
	    getSearchResultByKeyword: function(query) {
	      $.ajax({
	        url: '/getSearchResult',
	        type: 'post',
	        data: {
	          query: query
	        },
	        dataType: 'json',
	        success: function (response) {
			  return response.data;
	        },
	        error: function (xhr) {
	          alert('噢噢！發生錯誤了！請重新再試一次～');
	        }
	      });
	    }
	  };
	});

	Box.Application.addBehavior('getSearchResult', function(context) {
	  var query = "日式料理";
	  return {
	    init: function() {
	      var data = context.getService('getInfo').getSearchResultByKeyword(query);
		  console.log(data);
	    }
	  };
	});

	Box.Application.init();

## Behavior
不同模組但有重覆的事情要做，則可在模組中使用用Behavior。

	Box.Application.addModule('module-test-1', function(context) {
	  return {
	    behaviors: ['element-button'],
	    init: function(){
	      console.log('init');
	    }
	  }
	});

	Box.Application.addModule('module-test-2', function(context) {
	  return {
	    behaviors: ['element-button'],
	    init: function(){
	      console.log('init');
	    }
	  }
	});
	
	Box.Application.addBehavior('element-button', function(context) {
	  return {
	    init: function() {
	      console.log('add behavior');
	      context.getService('connectSomething').connect();
	    }
	  };
	});
	
	Box.Application.addService('connectSomething', function(application) {
	  return {
	    connect: function() {
	      console.log('connect something...');
	    }
	  };
	});
	
	Box.Application.init();

## DOMEventDelegate
委派。

### HTML

	<button data-module="module-test-domeventdelegate">
	  Test DOMEventDelegate
	</button>

### JS

	Box.Application.addModule('module-test-domeventdelegate', function(context) {
		var element = context.getElement();
		var delegate = new Box.DOMEventDelegate(element, {
			onclick: function(event) {
				console.log('DOMEventDelegate: ' + event.type);
			}
		});

	  return {
	    init: function(){
	    	var element = context.getElement();
	      delegate.attachEvents(); //DOMEventDelegate
	    },
	    onclick: function(event, element, elementType){
	      console.log('click!');
	    }
	  }
	});

	Box.Application.init();

## Context
### broadcast
模組訂閱事件，當事件發生時會通知模組。

#### HTML

	<div data-module="module-broadcast-1">1</div>
	<div data-module="module-broadcast-2">2</div>

#### JS

	Box.Application.addModule('module-broadcast-1', function(context) {
	  return {
	    messages: ['broadcast-a', 'broadcast-b'],
	    onmessage: function(name, data) {
	      switch(name) {
	        case 'broadcast-a':
	          console.log('broadcast-a');
	          Box.Application.broadcast('broadcast-b');
	          break;
	        case 'broadcast-b':
	          console.log('broadcast-b');
	          break;
	      }
	    }
	  }
	});

	Box.Application.addModule('module-broadcast-2', function(context) {
	  return {
	    messages: ['broadcast-a', 'broadcast-c'],
	    onmessage: function(name, data) {
	      switch(name) {
	        case 'broadcast-a':
	          console.log('broadcast-a');
	          break;
	        case 'broadcast-c':
	          console.log('broadcast-c');
	          break;
	      }
	    },
	  }
	});

	Box.Application.init();
	Box.Application.broadcast('broadcast-a');
	Box.Application.broadcast('broadcast-c');

### getGlobal
取得window。

	<div data-module="module-test">test</div>

	Box.Application.addModule('module-test', function(context) {
	  return {
	    init: function(){
	      var navigator = context.getGlobal('navigator');
	      console.log(window.navigator.userAgent === navigator.userAgent);
	    }
	  }
	});

	Box.Application.init();

### getGlobalConfig
取得 `Box.Application.init` 中的設定物件。

	<div data-module="module-test">test</div>

	Box.Application.addModule('module-test', function(context) {
	  return {
	    init: function(){
	      console.log(context.getGlobalConfig('userName'));
	    }
	  }
	});

	Box.Application.init({
		userName: 'Bob'
	});


[範例程式碼](https://github.com/cythilya/t3-example)
[T3 JavaScript Framework - T3 JavaScript Framework](http://t3js.org/)