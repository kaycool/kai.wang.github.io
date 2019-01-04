---
title: 'FlowLayout 形式的indicator'
layout: post
tags:
    - android 
    - 自定义View
    - indicator
    - FlowLayout
---
这是自定义的一个indicator，用于和viewpager绑定的指示器，可以横向滑动和纵向切换，内部再带处理nestscroll滑动机制，与CoordinatorLayout 交互可实现悬浮置顶效果。
<!--more-->

已经很久没心情去写东西了，勿忘初心。

![效果展示](https://upload-images.jianshu.io/upload_images/6370809-1140e7e46d0ba770.gif?imageMogr2/auto-orient/strip)

控件的一些属性展示如下：

MultiFlowIndicator->指示器

|attr|desc|
|:----------:|:------------:|
|multi_text_selected_color | 选中文本颜色->对应adapter中的selectedTextColor| 
|multi_icon_selected_color | 选中icon颜色->对应adapter中的selectedIconColor| 
|multi_text_selected_size | 选中文本字体大小->选中文本字体大小| 
|multi_text_unselected_color | 未选中文本颜色->对应adapter中的unSelectedTextColor| 
|multi_icon_unselected_color | 未选中icon颜色->对应adapter中的unSelectedIconColor| 
|multi_text_unselected_size | 未选中文本字体大小->未选中文本字体大小| 
|multi_indicator_height | 指示器高度| 
|multi_indicator_equal_title |指示器宽度与文本对齐| 
|multi_indicator_radius | 配合multi_indicator_style rectangle 属性使用，表示矩形的圆角| 
|multi_indicator_style | normal  和 rectangle| 
|multi_max_height | indicator最大高度| 
|multi_max_lines | indicator最大行数，float类型，multi_max_height 和 multi_max_lines 二者选其一| 

MultiFlowLayout->自定义FlowLayout

|attr|desc|
|:----------:|:------------:|
|multi_max_selected_count | 最大可选数| 
|multi_max_selected_Tips | 最大可选数提示| 
|multi_flow_padding_horizontal | 水平间距| 
|multi_flow_padding_vertical | 竖直间距| 
|multi_flow_max_height | 布局最大高度| 
|multi_flow_max_lines | 布局最大行数| 


github项目地址: https://github.com/kaycool/MultiIndicator

欢迎各位需要的人疯狂star，您可以在此基础之上定制您需要的控件效果，后期会维护和整理，当然也得看心情了，生活不易，所以任性些...

在下告辞！