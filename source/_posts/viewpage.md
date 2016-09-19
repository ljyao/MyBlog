---
title: viewpage
date: 2016-07-28 12:07:53
category: android
tags: view
---
# ViewPager
Layout manager that allows the user to flip left and right through pages of data. You supply an implementation of a PagerAdapter to generate the pages that the view shows.
<!-- more -->
ViewPager is most often used in conjunction with Fragment, which is a convenient way to supply and manage the lifecycle of each page. There are standard adapters implemented for using fragments with the ViewPager, which cover the most common use cases. These are FragmentPagerAdapter and FragmentStatePagerAdapter; each of these classes have simple code showing how to build a full user interface with them.

# PagerAdapter

Base class providing the adapter to populate pages inside of a ViewPager. You will most likely want to use a more specific implementation of this, such as FragmentPagerAdapter or FragmentStatePagerAdapter.

When you implement a PagerAdapter, you must override the following methods at minimum:

instantiateItem(ViewGroup, int)
destroyItem(ViewGroup, int, Object)
getCount()
isViewFromObject(View, Object)

# FragmentPagerAdapter

Implementation of PagerAdapter that represents each page as a Fragment that is persistently kept in the fragment manager as long as the user can return to the page.

This version of the pager is best for use when there are a handful of typically more static fragments to be paged through, such as a set of tabs. The fragment of each page the user visits will be kept in memory, though its view hierarchy may be destroyed when not visible. This can result in using a significant amount of memory since fragment instances can hold on to an arbitrary amount of state. For larger sets of pages, consider FragmentStatePagerAdapter.

适合于 Fragment数量不多的情况。当某个页面不可见时，该页面对应的View可能会被销毁，但是所有的Fragment都会一直存在于内存中。如果Fragment需要保存的状态较多时，会导致占用内存较大，因此对于Fragment数量较多的情况，建议使用FragmentStatePagerAdapter。

# FragmentStatePagerAdapter
Implementation of PagerAdapter that uses a Fragment to manage each page. This class also handles saving and restoring of fragment's state.

This version of the pager is more useful when there are a large number of pages, working more like a list view. When pages are not visible to the user, their entire fragment may be destroyed, only keeping the saved state of that fragment. This allows the pager to hold on to much less memory associated with each visited page as compared to FragmentPagerAdapter at the cost of potentially more overhead when switching between pages.

适合于Fragment数量较多的情况。当页面不可见时， 对应的Fragment实例可能会被销毁，但是Fragment的状态会被保存。因此每个Fragment占用的内存会更少，但是页面切换会引起较大开销。