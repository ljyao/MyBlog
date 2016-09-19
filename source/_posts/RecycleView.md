---
title: RecycleView
date: 2016-01-26 10:36:58
category: android
tags: view
---
    个人觉得RecycleView很强大，很好的扩展性，稳定，将会逐渐代替gridview和listview。
# GridLayoutManager
   实现gridview功能。

   使用GridLayoutManager给RecycleView加headview或footview，总共有3列，spansize表示表示该子view占几个位置。
```
 protected RecyclerView.LayoutManager getLayoutManager() {
        GridLayoutManager gridLayoutManager = new GridLayoutManager(getActivity(), 3);
        gridLayoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                int spanSize;
                switch (adapter.getItemViewType(position)) {
                    case ShowThumbnailData.LIVE:
                    case ShowThumbnailData.PHOTO:
                    case ShowThumbnailData.PLAYBACK:
                        spanSize = 1;
                        break;
                    case ShowThumbnailData.HEADER:
                        spanSize = 3;
                        break;
                    case ShowThumbnailData.LIVEV2:
                        spanSize = 3;
                    default:
                        spanSize = 1;
                        break;
                }
                return spanSize;
            }
        });
        return gridLayoutManager;
    }
```
# StaggeredGridLayoutManager
