
#### <center>**ALGORITHM**</center>
###### binary algorithm
二分查找是针对于有序数组的,以下是二分查找的find方法:
```
override fun find(searchKey: Long): Int {
        //binary algorithm
        var lowerItemIndex = 0
        var upperItemIndex = nElems - 1
        var targetItemIndex: Int
        while (true) {
            targetItemIndex = (lowerItemIndex + upperItemIndex) / 2
            if (a[targetItemIndex]==searchKey) {
                return targetItemIndex
            } else if (lowerItemIndex > upperItemIndex) { // no find item
                return nElems
            } else {
                if (a[targetItemIndex]!! < searchKey) {
                    lowerItemIndex = targetItemIndex + 1
                } else {
                    upperItemIndex = targetItemIndex - 1
                }
            }
        }
    }
```
