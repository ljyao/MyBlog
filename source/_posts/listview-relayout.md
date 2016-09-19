---
title: 关于listview子view的绘制
date: 2016-01-12 19:31:52
category: android
tags: view
---
在listview的getview中setData(),在子view的setdata(),onlayout(),onmeasure()方法中写上log,
发现setdata(),最先执行，onmeasure()执行了2次，onlayout()执行了一次，通过看listview源码找到原因。
- getHeightForPosition()方法中view.measure（）
- obtainView()方法中child.setLayoutParams(lp)，而setLayoutParams()又会执行requestLayout()
```
log
data->com.nice.main.data.enumerable.Show@17239164setdata
data->com.nice.main.data.enumerable.Show@17239164onMeasure
data->com.nice.main.data.enumerable.Show@17239164onMeasure
data->com.nice.main.data.enumerable.Show@17239164onLayout
```
<!-- more -->
# Listview和GridView中的子view的onlayout(),onmeasure()执行次数
## 在AbsListView.java类中getHeightForPosition()，onMeasure执行一次
```
   /**
     * Returns the height of the view for the specified position.
     *
     * @param position the item position
     * @return view height in pixels
     */
    int getHeightForPosition(int position) {
        final int firstVisiblePosition = getFirstVisiblePosition();
        final int childCount = getChildCount();
        final int index = position - firstVisiblePosition;
        if (index >= 0 && index < childCount) {
            // Position is on-screen, use existing view.
            final View view = getChildAt(index);
            return view.getHeight();
        } else {
            // Position is off-screen, obtain & recycle view.
            final View view = obtainView(position, mIsScrap);
            view.measure(mWidthMeasureSpec, MeasureSpec.UNSPECIFIED);
            final int height = view.getMeasuredHeight();
            mRecycler.addScrapView(view, position);
            return height;
        }
    }
```
## 在AbsListView.java类中obtainView()，onMeasure,onlayout各执行一次
```
 /**
     * Get a view and have it show the data associated with the specified
     * position. This is called when we have already discovered that the view is
     * not available for reuse in the recycle bin. The only choices left are
     * converting an old view or making a new one.
     *
     * @param position The position to display
     * @param isScrap Array of at least 1 boolean, the first entry will become true if
     *                the returned view was taken from the scrap heap, false if otherwise.
     *
     * @return A view displaying the data associated with the specified position
     */
    View obtainView(int position, boolean[] isScrap) {

        .................
        final View scrapView = mRecycler.getScrapView(position);
        .................
        setItemViewLayoutParams(child, position);
        .................
        return child;
    }
  private void setItemViewLayoutParams(View child, int position) {
        final ViewGroup.LayoutParams vlp = child.getLayoutParams();
        LayoutParams lp;
        if (vlp == null) {
            lp = (LayoutParams) generateDefaultLayoutParams();
        } else if (!checkLayoutParams(vlp)) {
            lp = (LayoutParams) generateLayoutParams(vlp);
        } else {
            lp = (LayoutParams) vlp;
        }

        if (mAdapterHasStableIds) {
            lp.itemId = mAdapter.getItemId(position);
        }
        lp.viewType = mAdapter.getItemViewType(position);
        child.setLayoutParams(lp);
    }
```
View.java
```
 /**
     * Set the layout parameters associated with this view. These supply
     * parameters to the <i>parent</i> of this view specifying how it should be
     * arranged. There are many subclasses of ViewGroup.LayoutParams, and these
     * correspond to the different subclasses of ViewGroup that are responsible
     * for arranging their children.
     *
     * @param params The layout parameters for this view, cannot be null
     */
    public void setLayoutParams(ViewGroup.LayoutParams params) {
        if (params == null) {
            throw new NullPointerException("Layout parameters cannot be null");
        }
        mLayoutParams = params;
        resolveLayoutParams();
        if (mParent instanceof ViewGroup) {
            ((ViewGroup) mParent).onSetLayoutParams(this, params);
        }
        requestLayout();
    }

```
# RecyclerView中子view的onlyaout，onMeasure次数
RecyclerView会多次对子view.measure
```
data->com.nice.main.data.enumerable.Show@183bd061setdata
data->com.nice.main.data.enumerable.Show@183bd061onMeasure
data->com.nice.main.data.enumerable.Show@183bd061onMeasure
data->com.nice.main.data.enumerable.Show@183bd061onMeasure
data->com.nice.main.data.enumerable.Show@183bd061onMeasure
data->com.nice.main.data.enumerable.Show@183bd061onLayout
```
## RecyclerView的bindViewToPosition引起子view重绘
RecyclerView.java
```
   /**
         * Binds the given View to the position. The View can be a View previously retrieved via
         * {@link #getViewForPosition(int)} or created by
         * {@link Adapter#onCreateViewHolder(ViewGroup, int)}.
         * <p>
         * Generally, a LayoutManager should acquire its views via {@link #getViewForPosition(int)}
         * and let the RecyclerView handle caching. This is a helper method for LayoutManager who
         * wants to handle its own recycling logic.
         * <p>
         * Note that, {@link #getViewForPosition(int)} already binds the View to the position so
         * you don't need to call this method unless you want to bind this View to another position.
         *
         * @param view The view to update.
         * @param position The position of the item to bind to this View.
         */
        public void bindViewToPosition(View view, int position) {
            ViewHolder holder = getChildViewHolderInt(view);
            ............
            final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            final LayoutParams rvLayoutParams;
            if (lp == null) {
                rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else if (!checkLayoutParams(lp)) {
                rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else {
                rvLayoutParams = (LayoutParams) lp;
            }

            rvLayoutParams.mInsetsDirty = true;
            rvLayoutParams.mViewHolder = holder;
            rvLayoutParams.mPendingInvalidate = holder.itemView.getParent() == null;
        }

```
## RecyclerView中的对子view的measure
```
        /**
         * Measure a child view using standard measurement policy, taking the padding
         * of the parent RecyclerView and any added item decorations into account.
         *
         * <p>If the RecyclerView can be scrolled in either dimension the caller may
         * pass 0 as the widthUsed or heightUsed parameters as they will be irrelevant.</p>
         *
         * @param child Child view to measure
         * @param widthUsed Width in pixels currently consumed by other views, if relevant
         * @param heightUsed Height in pixels currently consumed by other views, if relevant
         */
        public void measureChild(View child, int widthUsed, int heightUsed) {
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();

            final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);
            widthUsed += insets.left + insets.right;
            heightUsed += insets.top + insets.bottom;

            final int widthSpec = getChildMeasureSpec(getWidth(),
                    getPaddingLeft() + getPaddingRight() + widthUsed, lp.width,
                    canScrollHorizontally());
            final int heightSpec = getChildMeasureSpec(getHeight(),
                    getPaddingTop() + getPaddingBottom() + heightUsed, lp.height,
                    canScrollVertically());
            child.measure(widthSpec, heightSpec);
        }

        /**
         * Measure a child view using standard measurement policy, taking the padding
         * of the parent RecyclerView, any added item decorations and the child margins
         * into account.
         *
         * <p>If the RecyclerView can be scrolled in either dimension the caller may
         * pass 0 as the widthUsed or heightUsed parameters as they will be irrelevant.</p>
         *
         * @param child Child view to measure
         * @param widthUsed Width in pixels currently consumed by other views, if relevant
         * @param heightUsed Height in pixels currently consumed by other views, if relevant
         */
        public void measureChildWithMargins(View child, int widthUsed, int heightUsed) {
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();

            final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);
            widthUsed += insets.left + insets.right;
            heightUsed += insets.top + insets.bottom;

            final int widthSpec = getChildMeasureSpec(getWidth(),
                    getPaddingLeft() + getPaddingRight() +
                            lp.leftMargin + lp.rightMargin + widthUsed, lp.width,
                    canScrollHorizontally());
            final int heightSpec = getChildMeasureSpec(getHeight(),
                    getPaddingTop() + getPaddingBottom() +
                            lp.topMargin + lp.bottomMargin + heightUsed, lp.height,
                    canScrollVertically());
            child.measure(widthSpec, heightSpec);
        }


        /**
         * Returns the measured width of the given child, plus the additional size of
         * any insets applied by {@link ItemDecoration ItemDecorations}.
         *
         * @param child Child view to query
         * @return child's measured width plus <code>ItemDecoration</code> insets
         *
         * @see View#getMeasuredWidth()
         */
        public int getDecoratedMeasuredWidth(View child) {
            final Rect insets = ((LayoutParams) child.getLayoutParams()).mDecorInsets;
            return child.getMeasuredWidth() + insets.left + insets.right;
        }

        /**
         * Returns the measured height of the given child, plus the additional size of
         * any insets applied by {@link ItemDecoration ItemDecorations}.
         *
         * @param child Child view to query
         * @return child's measured height plus <code>ItemDecoration</code> insets
         *
         * @see View#getMeasuredHeight()
         */
        public int getDecoratedMeasuredHeight(View child) {
            final Rect insets = ((LayoutParams) child.getLayoutParams()).mDecorInsets;
            return child.getMeasuredHeight() + insets.top + insets.bottom;
        }
```
