---
layout:     post                    # 使用的布局（不需要改）
title:      RecycleView简单使用               # 标题 
subtitle:   Easyrecyclerview 使用 #副标题
date:       2020-9-11              # 时间
author:     JT                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - RecycleView
    - Easyrecyclerview
    - Android
---


[TOC]

###  easyrecyclerview

    implementation 'com.jude:easyrecyclerview:4.4.2'
    
    进阶查看[recycle实现filtermv筛选]

### 简单使用

    //初始化 EasyRecyclerView
    EasyRecyclerView topRecyclerView = new EasyRecyclerView(FilterMvActivity.this);
    ViewGroup.LayoutParams layoutParams = new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
    topRecyclerView.setLayoutParams(layoutParams);
    // 设置adapter,LayoutManager
    FilterAdapter filterAdapter = new FilterAdapter(FilterMvActivity.this, list);
    topRecyclerView.setLayoutManager(new LinearLayoutManager(FilterMvActivity.this, LinearLayoutManager.VERTICAL, false));
    topRecyclerView.setAdapter(filterAdapter);


### Adapter 

    public FilterAdapter(Context context, List<List<String>> objects) {
        super(context, objects);
    }
    
    @Override
    public BaseViewHolder OnCreateViewHolder(ViewGroup parent, int viewType) {
        return new FilterViewHolder(parent);
    }


### viewHolder

    public HitemViewHolder(ViewGroup parent) {
        super(parent, R.layout.filter_hitem_layout);
        textView = $(R.id.hitem_itemtitle);
    }
    
    @Override
    public void setData(String data) {
        //adpater 重写onbinderview走下面的方法
    }


    /**
     * 自定义方法 传入position,已达到状态更新的效果
     * 直接在这个界面设置position不行,只有从adapter中传过来的选中的position才行
     * @param data
     * @param position
     */
    public void setDataSelectPosition(String data, int position) {
        textView.setText(data);
        if (position == getDataPosition()) {
            textView.setBackgroundColor(Color.parseColor("#5CB4BB"));
        } else {
            textView.setBackgroundColor(Color.BLACK);
        }
    }
###  GridView添加头部View

```java

//创建gridLayoutManager,并设置gridview如果添加头部,把头部的几个列合并成一行,达到添加头部效果
 				GridLayoutManager gridLayoutManager = new GridLayoutManager(this, 5, GridLayoutManager.VERTICAL, false);
//RecyclerArrayAdapter封装好的gridviewmanager类型的添加头部
        RecyclerArrayAdapter<MainInfoBean>.GridSpanSizeLookup gridSpanSizeLookup = filterMainAdapter.obtainGridSpanSizeLookUp(5);
        gridLayoutManager.setSpanSizeLookup(gridSpanSizeLookup);
        easyRecyclerView.setLayoutManager(gridLayoutManager);
```

