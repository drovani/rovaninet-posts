---
title: "Recurring Gifts — Part 1: Database Schema"
category: Rovani in C♯
date: 2015-02-13
---

There are many posts out there for how to configure a database schema to handle scheduling events. The part that I found difficult to find was an algorithm for calculating future dates for each type of recurring schedule. I have started this series on Recurring Gifts as a way to document the process I went through to build the current system. Some features include:

- Ability to scheduled gifts by daily intervals, weekly intervals, monthly (by date) intervals, and monthly intervals by relative point. From there, it is simple math to do Quarterly, Yearly, or any other multiple.
- Ability to place gift generation On Hold until a certain date.
- Ability to Cancel gift generation on or after a certain date.
- Ability to makes changes that apply on a certain date.
- Easy to answer the question of "why was this gift created" because all gifts are linked to the schedule that generated it.


A few limitations that may constraint its universe utility:

- Only one active schedule can be created. Can only do "first Friday of every Month", not "first Friday of every Month, and third Tuesday of every other Month."
- Cannot schedule by the phase of the moon (not even joking ? this was an actual request) ? though it could be fudged by using a Daily schedule.
- Cannot schedule based on National Holidays (also an actual request).
- Schedules can only pause and restart once (via the Header.HoldUntilDate).

## Database Schema

Instead of inventing a database schema from nothing, I did what I do best — stand on the shoulders of giants, and Microsoft's SQL Server team is a pretty huge giant. Turning to their documentation on the [SQL Server Agent job schedules](https://msdn.microsoft.com/en-us/library/ms178644.aspx), I opened up the msdb.dbo.sysschedules table. I adjusted some of the definitions to fit my need and built my schema from there.

## Schedules Data Schema

There were three major changes that I made to the SQL schedule.

1. Frequency Interval, for weekly recurrences, is a bit flag in the SQL Agent schema. Since there is already an enum in the mscorlib, I changed the schema to use the System.DayOfWeek enum values.
1. Frequency Interval, for monthly relative recurrences, uses 1-7 for Sunday-Saturday, 8 = Day, 9 = Weekday, 10 = Weekend day. I only need the days of the week, so it made sense to also adjust this to use the System.DayOfWeek enum values.
1. I dropped all notion of "sub day" scheduling. There is no need to have gifts created at multiple points in a day.

A common approach to handling querying the event dates is to store a list of all future dates. I can also use this to determine when
the gift was actually created, and use this as an audit for gift creation. By tying the Audit to the Schedule, it is very easy to find
out exactly why an Audit was created.

![Recurring Schema](/images/recurring-schema.png)

## Evolution Path

There were two mistakes I made in the original implementation of this schema. First, I had the End conditions on the Schedule. The thought was that this would be mildly helpful with knowing why the generation of gifts might have started and stopped. Potentially, a termination point would be reached, gifts would stop being generated, then a new schedule would be added, thus resuming creation. However, the actual hit rate for this situation was zero. All it did was ridiculously complicate the algorithm for parsing the schedules.

Additionally, I previously had the Start Date on the Schedule. This also was confusing, as two schedules could be created and cause a gap in the timeline. It is much more easier to implement one "pause until" setting than try and handle skips. My use case was exactly zero for needing multiple start and stops, which is why I have pulled back on the complexity a bit for this second round.

## Up Next

The next post is going to start to demonstrate how I implemented various parts of the C♯ algorithm to generate the dates in a schedule. These collections then are combined to provide for a final list of all dates.