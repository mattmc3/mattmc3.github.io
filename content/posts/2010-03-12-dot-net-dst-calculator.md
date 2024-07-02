+++
title = '.NET DST Calculator'
date = 2010-03-12T11:58:00-05:00
+++

I have a common .NET library written in C# that I use everywhere. I call it `mattmc3.Util`, and it contains helper methods (static classes or extension methods) that correspond to Microsoft's BCL. So, for example, I have a `DateTimeHelper` with static methods that corresponds to the `System.DateTime` class. The `DateTime` class is one of the most anemic in the .NET library, mainly because MS has to deal with international customers, so methods like `DateTime GetMemorialDay(int year)` are too regional and clutter the BCL, but are essential in the American business world.

In honor of Daylight Savings Time beginning this weekend, I decided to post some code from my `DateTimeHelper` class.

First off, a little bit of helper code.

```csharp
public enum Month {
    January = 1,
    February = 2,
    March = 3,
    April = 4,
    May = 5,
    June = 6,
    July = 7,
    August = 8,
    September = 9,
    October = 10,
    November = 11,
    December = 12
}

public enum WeekPlacement {
    First = 1,
    Second = 2,
    Third = 3,
    Fourth = 4,
    Last = 5
}

public static DateTime GetLastDayOfMonth(DateTime theDate) {
    int daysInMonth = DateTime.DaysInMonth(theDate.Year, theDate.Month);
    return new DateTime(theDate.Year, theDate.Month, daysInMonth);
}
```

Then, the workhorse method that does the calculations.

```csharp
/// <summary>
/// This method calculates a date for you. For example, Thanksgiving falls on the 4th Thursday
/// of November, so you would call GetCalculatedDate(WeekPlacement.Fourth, DayOfWeek.Thursday, Month.November, DateTime.Now.Year)
/// </summary>
public static DateTime GetCalculatedDate(WeekPlacement weekPlacement, DayOfWeek dayOfWeek, Month month, int year) {
    var result = DateTime.MinValue;
    var intMonth = (int)month;

    if (weekPlacement == WeekPlacement.Last) {
        result = GetLastDayOfMonth(new DateTime(year, intMonth, 1));
        while (result.DayOfWeek != dayOfWeek) {
            result = result.AddDays(-1);
        }
    }
    else {
        result = new DateTime(year, intMonth, 1);
        while (result.DayOfWeek != dayOfWeek) {
            result = result.AddDays(1);
        }

        var weeksToAdd = (int)weekPlacement - 1;
        result = result.AddDays(7 * weeksToAdd);
    }

    // Post condition
    if (result.DayOfWeek != dayOfWeek || result.Month != intMonth || result.Year != year) {
        result = DateTime.MinValue;
    }

   return result;
}
```

And finally, some of the methods that allow me to calculate holidays and other calendar events that are important for our business applications.

```csharp
public static DateTime GetNewYearsDay(int year) {
    return new DateTime(year, 1, 1);
}

public static DateTime GetMemorialDay(int year) {
    return GetCalculatedDate(WeekPlacement.Last, DayOfWeek.Monday, Month.May, year);
}

public static DateTime GetIndependanceDay(int year) {
    return new DateTime(year, 7, 4);
}

public static DateTime GetLaborDay(int year) {
    return GetCalculatedDate(WeekPlacement.First, DayOfWeek.Monday, Month.September, year);
}

public static DateTime GetThanksgivingDay(int year) {
    return GetCalculatedDate(WeekPlacement.Fourth, DayOfWeek.Thursday, Month.November, year);
}

public static DateTime GetChristmasDay(int year) {
    return new DateTime(year, 12, 25);
}

public static DateTime GetDaylightSavingsTimeStart(int year) {
    if (year >= 2007) {
        // Second Sunday in March
        return GetCalculatedDate(WeekPlacement.Second, DayOfWeek.Sunday, Month.March, year);
   }
   else if (year >= 1987) {
        // First Sunday in April
        return GetCalculatedDate(WeekPlacement.First, DayOfWeek.Sunday, Month.April, year);
   }
   else {
        return DateTime.MinValue;
   }
}

public static DateTime GetDaylightSavingsTimeEnd(int year) {
    if (year >= 2007) {
        // First Sunday in November
        return GetCalculatedDate(WeekPlacement.First, DayOfWeek.Sunday, Month.November, year);
    }
    else if (year >= 1987) {
        // Last Sunday in October
        return GetCalculatedDate(WeekPlacement.Last, DayOfWeek.Sunday, Month.October, year);
   }
   else {
        return DateTime.MinValue;
   }
}
```

Enjoy!  Spring is nearly here!
