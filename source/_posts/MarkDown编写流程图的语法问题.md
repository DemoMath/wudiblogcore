---
layout: post
title:  "Markdown编写流程图的语法问题"
date:   2017-03-27
categories: [Markdown,流程图]
tags: [Text]
---
画流程图的步骤

>好气哦！我的编辑器不支持解析！ :(

>后头搞一下，未完待续...

1. 第一段用来定义元素
	
	语法：
		
		tag=>type:content:>url
		
	tag就是一个标签，下面我们在定义连接元素时用type是这个标签的类型
	
	基本类型：
	
		start
		end
		operation
		subroutine
		condition
		input/output
		
	开始：
		
		st=>start:开始
		
	操作流程：
	
		st->op->cond
		
	条件：
	
		cond=>condition:确认?
		
	结束：
	
		e=>end:结束
		
	示例：


<textarea id="code" style="width: 100%;" rows="11">
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
</textarea>


		st=>start: Start
		e=>end
		op=>operation: My Operation
		cond=>condition: Yes or No?

		st->op->cond
		cond(yes)->e
		cond(no)->op	

2. 第二段用来连接元素

	st=>start:开始
	st->op->cond
	
	
***
	
	flow
	st=>start: Start
	op=>operation: Your Operation
	cond=>condition: Yes or No?
	e=>end
	st->op->cond
	cond(yes)->e
	cond(no)->op
	
～～～flow
st=>start: Start:>http://alfred-sun.github.io
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end:>https://github.com/adrai/flowchart.js

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
～～～

	st=>start: 开始:>http://www.google.com[blank]
	e=>end:    结束:>http://www.google.com
	op1=>operation: 操作
	sub1=>subroutine: 子程序
	cond=>condition: Yes 
	or No?:>http://www.google.com
	io=>inputoutput: 输入输出

	st->op1->cond
	cond(yes)->io->e
	cond(no)->sub1(right)->op1



<flow>
st=>start: Start:>http://alfred-sun.github.io
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end:>https://github.com/adrai/flowchart.js

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
</flow>



