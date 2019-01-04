---
title: '适配RecycleView的 ItemDecoration 的封装'
layout: post
tags:
    - android 
    - RecycleView
    - ItemDecoration
---
ItemDecoration 的封装
<!--more-->

## 1. ItemDecoration类的定义

__ItemDecoration-->RecycleView的item系统间距以及系统item绘制封装类__

    /**
     * ItemDecoration允许程序对适配器的数据源中的特殊的条目      
     * view 添加一些特别的图形和布局的偏移，这被用于在条目之      
     * 间画间隔线，高亮显示，视觉分组界限和更多。
     * 所有的ItemDecorations
     * <p>所有的ItemDecorations都是按照它们添加的顺序绘制的, 
     * 在所有的条目view绘制之前
    *  (in {@link ItemDecoration#onDraw(Canvas, RecyclerView, RecyclerView.State) onDraw()}
     * 在所有的条目view绘制之后 (in {@link ItemDecoration#onDrawOver(Canvas, RecyclerView,
     * RecyclerView.State)}.</p>
     */
    public abstract static class ItemDecoration {
       ...
       ...
    }

## 2. ItemDecoration的方法分析

__此类下的六个方法有三个为过时方法，只需要复写另外三个未过时的方法即可，谷歌对这三个方法的定义如下：__

* 在提供给RecyclerView的Canvas中绘制任何适当的装饰。 此方法绘制的任何内容将在绘制项目视图之前绘制， 因此将被视图覆盖.


      public void onDraw(Canvas c, RecyclerView parent, State state{
          onDraw(c, parent);
      }


* 在提供给RecyclerView的Canvas中绘制任何适当的装饰。 此方法绘制的任何内容将在绘制项目视图之后绘制， 因此将覆盖视图


      public void onDrawOver(Canvas c, RecyclerView parent, State state) {
            onDrawOver(c, parent);
      }

*  检索给定条目的所有偏移量， outRect 的每个字段指定条目视图应该被插入的像素大小，类似于填充或边距。默认实现将outRect的边界设置为0并返回。
    

      public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
            getItemOffsets(outRect, ((LayoutParams)view.getLayoutParams()).getViewLayoutPosition(), parent);
      }

> 注：onDraw 和 onDrawOver的区别在于，若未规定条目视图的左上右下的偏移量，在条目视图的基础上，使用canvas 所做的任何装饰，前者会被条目视图所覆盖，而后者方法则会在条目视图的基础上作画。

## 3. 封装 getItemOffsets

* 通过构造函数传入左上右下的间距
     

    private int   mSpaceHorizontal;
    private int   mSpaceVertical;
    private Paint mPaint;

    DividerDecoration(Activity activity, int spaceHorizontal, int spaceVertical) {
        mSpaceHorizontal = spaceHorizontal;
        mSpaceVertical = spaceVertical;
        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setAntiAlias(true);
        mPaint.setColor(ContextCompat.getColor(activity, android.R.color.transparent));
    }

    DividerDecoration(Activity activity, int color, int spaceHorizontal, int spaceVertical) {
        mSpaceHorizontal = spaceHorizontal;
        mSpaceVertical = spaceVertical;
        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setAntiAlias(true);
        mPaint.setColor(ContextCompat.getColor(activity, color));
    }

* 筛选RecycleView的布局管理器 ，对应做单独处理


    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
          super.getItemOffsets(outRect, view, parent, state);
          RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
          if (layoutManager instanceof GridLayoutManager) {
              layoutGridItem(outRect, view, parent, (GridLayoutManager) layoutManager);
          } else if (layoutManager instanceof LinearLayoutManager) {
              layoutLinearItem(outRect, view, parent, (LinearLayoutManager) layoutManager);
          } else if (layoutManager instanceof StaggeredGridLayoutManager) {
              layoutStaggeredItem(outRect, view, parent, (StaggeredGridLayoutManager) layoutManager);
          }
      }


* LinearLayoutManager 的布局偏移封装

    
    private void layoutLinearItemSpace(Rect outRect, View view, RecyclerView parent, LinearLayoutManager linearManager) {
        int viewPosition = parent.getChildAdapterPosition(view);

        if (linearManager.getOrientation() == LinearLayoutManager.VERTICAL) {
            outRect.left = mSpaceHorizontal;
            outRect.right = mSpaceHorizontal;
            if (viewPosition == 0) {
                outRect.top = mSpaceVertical;
                outRect.bottom = mSpaceVertical / 2;
            } else if (viewPosition == linearManager.getItemCount() - 1) {
                outRect.top = mSpaceVertical / 2;
                outRect.bottom = mSpaceVertical;
            } else {
                outRect.top = mSpaceVertical / 2;
                outRect.bottom = mSpaceVertical / 2;
            }
        } else {
            outRect.top = mSpaceVertical;
            outRect.bottom = mSpaceVertical;

            if (viewPosition == 0) {
                outRect.left = mSpaceHorizontal;
                outRect.right = mSpaceHorizontal / 2;
            } else if (viewPosition == linearManager.getItemCount() - 1) {
                outRect.left = mSpaceHorizontal / 2;
                outRect.right = mSpaceHorizontal;
            } else {
                outRect.left = mSpaceHorizontal / 2;
                outRect.right = mSpaceHorizontal / 2;
            }
        }
    }

* GridLayoutManager 的布局偏移封装


    private void layoutGridItemSpace(Rect outRect, View view, RecyclerView parent, GridLayoutManager gridManager) {
        int viewPosition = parent.getChildAdapterPosition(view);
        int spanCount = gridManager.getSpanCount();
        int index = viewPosition % spanCount;


        if (viewPosition < spanCount) {//第一行
            outRect.top = mSpaceVertical;
            outRect.bottom = mSpaceVertical / 2;
        } else if (viewPosition / spanCount == (gridManager.getItemCount() - 1) / spanCount) {//多宫格如何判断最后一行？
            outRect.top = mSpaceVertical / 2;
            outRect.bottom = mSpaceVertical;
        } else {
            outRect.top = mSpaceVertical / 2;
            outRect.bottom = mSpaceVertical / 2;
        }


        if (index == 0) {//第一列
            outRect.left = mSpaceHorizontal;
            outRect.right = mSpaceHorizontal / 2;

        } else if (index == spanCount - 1) {//最后一列
            outRect.left = mSpaceHorizontal / 2;
            outRect.right = mSpaceHorizontal;

        } else {
            outRect.left = mSpaceHorizontal / 2;
            outRect.right = mSpaceHorizontal / 2;
        }

    }


* StaggeredGridLayoutManager的布局偏移封装


    private void layoutStaggeredItemSpace(Rect outRect, View view, RecyclerView parent, StaggeredGridLayoutManager staggeredManager) {

    }

* LinearLayoutManager 的布局偏移颜色封装

    
    private void drawLinearItemSpace(Canvas c, RecyclerView parent, LinearLayoutManager linearManager) {

        if (linearManager.getOrientation() == LinearLayoutManager.VERTICAL) {
            for (int viewPosition = 0; viewPosition < linearManager.getItemCount(); viewPosition++) {
                View childView = parent.getChildAt(viewPosition);

                if (childView == null) {
                    continue;
                }

                RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) childView.getLayoutParams();

                int left;
                int top;
                int right;
                int bottom;
                left = childView.getLeft() - params.leftMargin;
                right = childView.getRight() + params.rightMargin;
                top = childView.getBottom() + params.bottomMargin;
                bottom = top + mSpaceVertical;
                c.drawRect(left, top, right, bottom, mPaint);
                if (viewPosition == 0) {
                    top = childView.getTop() - params.topMargin - mSpaceVertical;
                    bottom = top + mSpaceVertical;
                    c.drawRect(left, top, right, bottom, mPaint);
                }
            }
        } else {


            for (int viewPosition = 0; viewPosition < linearManager.getItemCount(); viewPosition++) {
                View childView = parent.getChildAt(viewPosition);

                if (childView == null) {
                    continue;
                }

                RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) childView.getLayoutParams();

                int left;
                int top;
                int right;
                int bottom;


                left = childView.getRight() + params.rightMargin;
                top = childView.getTop() - params.topMargin;
                right = left + mSpaceHorizontal;
                bottom = childView.getBottom() + params.bottomMargin;
                c.drawRect(left, top, right, bottom, mPaint);

                if(viewPosition == 0) {
                    left = childView.getLeft() - params.leftMargin - mSpaceHorizontal;
                    top = childView.getTop() - params.topMargin;
                    right = left + mSpaceHorizontal;
                    bottom = childView.getBottom() + params.bottomMargin;
                    c.drawRect(left, top, right, bottom, mPaint);
                }
            }
        }
    }


* GridLayoutManager 的布局颜色填充封装


    private void drawGridItemSpace(Canvas c, RecyclerView parent, GridLayoutManager gridManager) {
        for (int viewPosition = 0; viewPosition < gridManager.getItemCount(); viewPosition++) {
            View childView = parent.getChildAt(viewPosition);

            if (childView == null) {
                continue;
            }

            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) childView.getLayoutParams();

            int spanCount = gridManager.getSpanCount();
            int index = viewPosition % spanCount;

            int left;
            int top;
            int right;
            int bottom;

            left = childView.getLeft() - params.leftMargin;
            right = childView.getRight() + params.rightMargin + mSpaceHorizontal;
            if (viewPosition < spanCount) {//第一行
                top = childView.getTop() - params.topMargin - mSpaceVertical;
                bottom = top + mSpaceVertical;
                c.drawRect(left, top, right, bottom, mPaint);

                top = childView.getBottom() + params.bottomMargin;
                bottom = top + mSpaceVertical;
                c.drawRect(left, top, right, bottom, mPaint);
            } else {
                top = childView.getBottom() + params.bottomMargin;
                bottom = top + mSpaceVertical;
                c.drawRect(left, top, right, bottom, mPaint);
            }


            if (index == 0) {//第一列
                top = childView.getTop() - params.topMargin;
                bottom = childView.getBottom() + params.bottomMargin + mSpaceVertical;
                left = childView.getLeft() - params.leftMargin - mSpaceHorizontal;
                right = left + mSpaceHorizontal;
                c.drawRect(left, top, right, bottom, mPaint);

                top = childView.getTop() - params.topMargin;
                bottom = childView.getBottom() + params.bottomMargin;
                left = childView.getRight() + params.rightMargin;
                right = left + mSpaceHorizontal;
                c.drawRect(left, top, right, bottom, mPaint);

                if (viewPosition == 0) {
                    top = childView.getTop() - params.topMargin - mSpaceVertical;
                    bottom = top + mSpaceVertical;
                    left = childView.getLeft() - params.leftMargin - mSpaceHorizontal;
                    right = left + mSpaceHorizontal;
                    c.drawRect(left, top, right, bottom, mPaint);
                }

            } else {
                top = childView.getTop() - params.topMargin;
                bottom = childView.getBottom() + params.bottomMargin;
                left = childView.getRight() + params.rightMargin;
                right = left + mSpaceHorizontal;
                c.drawRect(left, top, right, bottom, mPaint);
            }
        }
    }


* StaggeredGridLayoutManager的布局偏移颜色填充封装



__StaggeredGridLayoutManager 相关的目前正在研究，后面会补充完善__






