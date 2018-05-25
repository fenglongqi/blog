## 高性能CSS3动画

高性能移动Web相较PC的场景需要考虑的因素也相对更多更复杂，我们总结为一下几点：**流量/功耗与流畅度**。

###流畅度
流畅度主要体现在前端动画中，在现有的前端动画体系中，通常由两种模式：JS动画与CSS3动画。JS动画是通过JS动态改写样式实现动画能力的一种方案。PC端兼容性上JS动画为推荐方案。在移动端，我们选择更优浏览器原生实现方案：CSS3动画。

然而，CSS3动画在移动端主要会有动画的卡顿与闪烁的问题

###提升性能方法

####尽可能多的利用硬件能力，如使用3D变形来开启GPU加速

	-webkit-transform: translate3d(0, 0, 0);
	-moz-transform: translate3d(0, 0, 0);
	-ms-transform: translate3d(0, 0, 0);
	transform: translate3d(0, 0, 0);

动画过程有闪烁(通常发生在动画开始的时候)，可以尝试下面的Hack：

	-webkit-backface-visibility: hidden;
	-moz-backface-visibility: hidden;
	-ms-backface-visibility: hidden;
	backface-visibility: hidden;
	
	-webkit-perspective: 1000;
	-moz-perspective: 1000;
	-ms-perspective: 1000;
	perspective: 1000;


注：3D变形会消耗更多的内存与功耗，应确实由性能问题时才去用它，

####尽可能少使用box-shadows与gradients

box-shadows与gradients往往都是页面的性能杀手，

####尽可能的让动画元素不在文档流中，以减少重排

	position:fixed;
	position:absolute;

###优化DOM layout性能

	// 触发两次 layout
	var newWidth = aDiv.offsetWidth + 10;   // Read
	aDiv.style.width = newWidth + 'px';     // Write
	var newHeight = aDiv.offsetHeight + 10; // Read
	aDiv.style.height = newHeight + 'px';   // Write
	
	// 只触发一次 layout
	var newWidth = aDiv.offsetWidth + 10;   // Read
	var newHeight = aDiv.offsetHeight + 10; // Read
	aDiv.style.width = newWidth + 'px';     // Write
	aDiv.style.height = newHeight + 'px';   // Write

所有可触发layout的操作都会被暂时放入layout-queue中，等到必须更新时，在计算整个队列中所有操作影响的结果，如此就可只进行一次layout，从而提升性能。

#### 关键一，可触发layout的操作

Webkit为例，Webkit主要通过Document::updateLayout与Document::updateLayoutlgnorePendingStylesheets两个方法

	void Document::updateLayout()
	{
	    ASSERT(isMainThread());
	
	    FrameView* frameView = view();
	    if (frameView && frameView->isInLayout()) {
	        ASSERT_NOT_REACHED();
	        return;
	    }
	
	    if (Element* oe = ownerElement())
	        oe->document()->updateLayout();
	
	    updateStyleIfNeeded();
	
	    StackStats::LayoutCheckPoint layoutCheckPoint;
	
	    if (frameView && renderer() && (frameView->layoutPending() || renderer()->needsLayout()))
	        frameView->layout();
	
	    if (m_focusedNode && !m_didPostCheckFocusedNodeTask) {
	        postTask(CheckFocusedNodeTask::create());
	        m_didPostCheckFocusedNodeTask = true;
	    }
	}
	
	
	void Document::updateLayoutIgnorePendingStylesheets()
	{
	    bool oldIgnore = m_ignorePendingStylesheets;
	
	    if (!haveStylesheetsLoaded()) {
	        m_ignorePendingStylesheets = true;
	
	        HTMLElement* bodyElement = body();
	        if (bodyElement && !bodyElement->renderer() && m_pendingSheetLayout == NoLayoutWithPendingSheets) {
	            m_pendingSheetLayout = DidLayoutWithPendingSheets;
	            styleResolverChanged(RecalcStyleImmediately);
	        } else if (m_hasNodesWithPlaceholderStyle)
	            recalcStyle(Force);
	    }
	
	    updateLayout();
	
	    m_ignorePendingStylesheets = oldIgnore;
	}


webkit实现中调用updateLayoutgnorePendingStylesheets的操作


* Element:clientHeight , clientLeft, clientTop, clientWidth, focus(), getBoundingClientRect(), getClientREcts(), innerText, offsetHeight , offsetLeft, offsetParent, offsetTop, offsetTop, offsetWidth, outerTex, scrollHeight, scrollIntoVeiew(), scrollIntoViewIfNeeded(), scrollLeft, scrollTop, scrollWidth

* Frame,HTMLImageElment:height,width

* Range:getBoundingClientRect(),getClientRects()

* SVGTextContent:getCharNumAtPosition(), getComputedTextLength(), getEndPositionOfChar(), getExtentOfChar(), getNumberOfChars(), getRotationOfChar(), getStartPositionOfChar(), getSubStringLength(), selectSubString()

* SVGUse:instanceRoot

*window:getComputedStyle(), scrollBy(), scrollTo(), scrollX, scrollY, webkitConvertPointFromNodeToPage(), webkitConvertPointFromPageToNode()
 





