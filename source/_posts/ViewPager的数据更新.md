---
title: ViewPager的数据更新
date: 2016-09-19 11:11:52
tags: view
---
ViewPager的更新通过的PagerAdapter中的notifyDataSetChanged方法，可以增加、删除或排序，但不可以像RecycleView更新view。
**官方文档：**
PagerAdapter supports data set changes. Data set changes must occur on the main thread and must end with a call to {@link #notifyDataSetChanged()} similar to AdapterView adapters derived from {@link android.widget.BaseAdapter}. A data set change may involve pages being added, removed, or changing position. The ViewPager will keep the current page active provided the adapter implements the method {@link #getItemPosition(Object)}.

更新后ViewPager是否保持当前页存活，通过adapter实现getItemPosition方法进行判断。
```
    /**
     * Called when the host view is attempting to determine if an item's position
     * has changed. Returns {@link #POSITION_UNCHANGED} if the position of the given
     * item has not changed or {@link #POSITION_NONE} if the item is no longer present
     * in the adapter.
     *
     * <p>The default implementation assumes that items will never
     * change position and always returns {@link #POSITION_UNCHANGED}.
     *
     * @param object Object representing an item, previously returned by a call to
     *               {@link #instantiateItem(View, int)}.
     * @return object's new position index from [0, {@link #getCount()}),
     *         {@link #POSITION_UNCHANGED} if the object's position has not changed,
     *         or {@link #POSITION_NONE} if the item is no longer present.
     */
    public int getItemPosition(Object object) {
        return POSITION_UNCHANGED;
    }
```
POSITION_UNCHANGED不更新，POSITION_NONE更新。
所以重写getItemPosition方法，返回POSITION_NONE，可以先实现通过notifyDataSetChanged进行页面更新。
<!-- more -->
# 源码分析：
PagerAdapter notifyDataSetChanged方法
```
    /**
     * This method should be called by the application if the data backing this adapter has changed
     * and associated views should update.
     */
    public void notifyDataSetChanged() {
        synchronized (this) {
            if (mViewPagerObserver != null) {
                mViewPagerObserver.onChanged();
            }
        }
        mObservable.notifyChanged();
    }
```
mViewPagerObserver来自viewpage.
```
 public void setAdapter(PagerAdapter adapter) {
    .....
    if (mAdapter != null) {
            if (mObserver == null) {
                mObserver = new PagerObserver();
            }
            mAdapter.setViewPagerObserver(mObserver);
    ......
```
PagerObserver为viewPage内部类，继承了DataSetObserver抽象类。
```
private class PagerObserver extends DataSetObserver {
        @Override
        public void onChanged() {
            dataSetChanged();
        }
        @Override
        public void onInvalidated() {
            dataSetChanged();
        }
    }
```

ViewPager dataSetChanged方法
```
    void dataSetChanged() {
        // This method only gets called if our observer is attached, so mAdapter is non-null.

        final int adapterCount = mAdapter.getCount();
        mExpectedAdapterCount = adapterCount;
        boolean needPopulate = mItems.size() < mOffscreenPageLimit * 2 + 1 &&
                mItems.size() < adapterCount;
        int newCurrItem = mCurItem;

        boolean isUpdating = false;
        for (int i = 0; i < mItems.size(); i++) {
            final ItemInfo ii = mItems.get(i);
            //获取是否更新
            final int newPos = mAdapter.getItemPosition(ii.object);

            if (newPos == PagerAdapter.POSITION_UNCHANGED) {
                continue;
            }

            if (newPos == PagerAdapter.POSITION_NONE) {
                mItems.remove(i);
                i--;

                if (!isUpdating) {
                    mAdapter.startUpdate(this);
                    isUpdating = true;
                }

                mAdapter.destroyItem(this, ii.position, ii.object);
                needPopulate = true;

                if (mCurItem == ii.position) {
                    // Keep the current item in the valid range
                    newCurrItem = Math.max(0, Math.min(mCurItem, adapterCount - 1));
                    needPopulate = true;
                }
                continue;
            }

            if (ii.position != newPos) {
                if (ii.position == mCurItem) {
                    // Our current item changed position. Follow it.
                    newCurrItem = newPos;
                }

                ii.position = newPos;
                needPopulate = true;
            }
        }

        if (isUpdating) {
            mAdapter.finishUpdate(this);
        }

        Collections.sort(mItems, COMPARATOR);

        if (needPopulate) {
            // Reset our known page widths; populate will recompute them.
            final int childCount = getChildCount();
            for (int i = 0; i < childCount; i++) {
                final View child = getChildAt(i);
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                if (!lp.isDecor) {
                    lp.widthFactor = 0.f;
                }
            }

            setCurrentItemInternal(newCurrItem, false, true);
            requestLayout();
        }
    }
```
从上面我们看到，它会判断adapter的getItemPosition方法的返回值，只有当返回值是POSITION_NONE时候，才会调用item的remove方法以及startUpdate和destroyItem方法，进而去更新数据。默认返回值是POSITION_UNCHANGED，不执行任何操作。

