#android #testing #viewPager1

There is a problem with Espresso under Robolectric environment not waiting for the smooth scroll finishing. This causes a false negative result of assertion reporting invisible draw rect for the view queried with `ViewInteraction`.

The solution is to override default tab listener with one that performs selection without smooth animation.

```kotlin
/**  
 * Because of the smooth scroll animation espresso returns the empty rectangle for the currently selected tab. */private fun disableScrollAnimationOnTabLayout(activity: FragmentActivity) {  
    // We are relying on multiple instances of navhost fragment, we need to pick the right one just after the activity is shown  
    val fragmentManager = activity.supportFragmentManager.primaryNavigationFragment?.childFragmentManager.shouldNotBeNull()  
    val listener = object : FragmentManager.FragmentLifecycleCallbacks() {  
        override fun onFragmentViewCreated(fm: FragmentManager, f: Fragment, v: View, savedInstanceState: Bundle?) {  
            super.onFragmentViewCreated(fm, f, v, savedInstanceState)  
            if (f is ViewPagerFragment) {  
                disableScrollAnimationOnTabLayout(v)  
                fragmentManager.unregisterFragmentLifecycleCallbacks(this)  
            }  
        }  
  
        private fun disableScrollAnimationOnTabLayout(view: View) {  
            val tabLayout = view.findViewById<TabLayout>(R.id.tabLayout).shouldNotBeNull()  
            val viewPager = view.findViewById<ViewPager>(R.id.viewPager).shouldNotBeNull()  
  
            // Yes we want to get rid of any other listener to suppress smooth scroll side effect  
            @Suppress("DEPRECATION")  
            tabLayout.setOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {  
                override fun onTabSelected(tab: TabLayout.Tab) {  
                    viewPager.setCurrentItem(tab.position, false)  
                }  
  
                override fun onTabUnselected(tab: TabLayout.Tab?) = Unit  
                override fun onTabReselected(tab: TabLayout.Tab?) = Unit  
            })  
        }  
    }  
    fragmentManager.registerFragmentLifecycleCallbacks(listener, false)  
}
```