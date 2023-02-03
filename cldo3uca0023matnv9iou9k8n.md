# Android: How to create Custom Calendar with RecyclerView

> Post inspired by [RecyclerCalendarAndroid Lib](https://github.com/tejpratap46/RecyclerCalendarAndroid)

### RecyclerView

The theory is simple, we need each day of the calendar rendered as one cell of the RecyclerView.

To achieve this, we need to first prepare list of all dates, but there is a problem, we are missing a few things:

1. Header Cell for Month with span size of 7.
    
2. Preceding empty cell (if the month start from Wednesday, we need to add an empty cell with a span of 2)
    
3. Succeeding empty cell (if the month ends on a Friday, then we need to add an empty cell with a span of 2)
    

Now we need to achieve this,

we need to iterate from our start date to the end date:

```kotlin
while (startCalendar.time.before(endCalendar.time) || startCalendar.time == endCalendar.time) {
    //...
}
```

Now, inside out while loop, we check if the date is the first date of the month, we add our empty cell before we add an actual cell with a date.

```kotlin
val calendarEmptyHeader: RecyclerCalenderViewItem =
    RecyclerCalenderViewItem(
        date = startCalendar.time,
        spanSize = 7,
        isEmpty = false,
        isHeader = true
    )
calendarItemList.add(calendarEmptyHeader)

val calendarEmptyWeek: RecyclerCalenderViewItem =
    RecyclerCalenderViewItem(
        date = startCalendar.time,
        spanSize = 1,
        isEmpty = true,
        isHeader = false
    )
if (dayOfWeek - 1 < 0) {
    dayOfWeek = 7 + (dayOfWeek - 1)
    calendarEmptyWeek.spanSize = dayOfWeek
} else {
    calendarEmptyWeek.spanSize = dayOfWeek - 1
}
if (calendarEmptyWeek.spanSize > 0) {
    // Is span size is greater then 0, then add empty cell
    calendarItemList.add(calendarEmptyWeek)
}
```

Now, after you added an empty cell, just add the actual date and increment the start date by 1 day

```kotlin
val calendarDateItem: RecyclerCalenderViewItem =
    RecyclerCalenderViewItem(
        date = startCalendar.time,
        spanSize = 1,
        isEmpty = false,
        isHeader = false
    )
calendarItemList.add(calendarDateItem)
startCalendar.add(Calendar.DATE, 1)
```

After doing all that in the while loop, we now have a list with all the dates, headers and empty cells.

Use this list as input to your `RecyclerView` and render each date, month and empty cell with different views, customize them as per your need.

# Or

Simply use this library, customize your views as per your need, supports events, and add additional views and animations, the possibilities are endless.

%[https://github.com/tejpratap46/RecyclerCalendarAndroid]