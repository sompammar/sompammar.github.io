---
layout: post
title:  "Generate CRON Expression"
date:   2018-09-01 18:41:34 -0700
categories: quartz cron scheduler expression builder
---
Let's say you want to build cron expression. You want to take input from the user on when to start a job. It could be a future date. And from there you want it to repeat on a daily/weekly/monthly/quarterly basis.

e.g. If you select as your starting date as Jan 1, 2018, Monday 9:35:02 AM, and you want to repeat it weekly from there onwards. The CRON expression for that would be `2 35 9 9/7 1/1 ? 2018/1`

Here's a sample program to achieve this.

```java
    public static String toCronExpression(ScheduleDetail scheduleDetail) {
        Date startDate = scheduleDetail.getStartTime();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(startDate);
        long dayOfMonth = calendar.get(Calendar.DAY_OF_MONTH);
        long month = calendar.get(Calendar.MONTH) + 1; //starting at 1
        long seconds = calendar.get(Calendar.SECOND);
        long minutes = calendar.get(Calendar.MINUTE);
        long hours = calendar.get(Calendar.HOUR_OF_DAY);
        long dayOfWeek = calendar.get(Calendar.DAY_OF_WEEK);
        long year = calendar.get(Calendar.YEAR);

        ScheduleDetail.Frequency frequency = scheduleDetail.getRepeatFrequency();

        StringBuilder secondsPart = new StringBuilder();
        StringBuilder minutesPart = new StringBuilder();
        StringBuilder hoursPart = new StringBuilder();
        StringBuilder dayOfMonthPart = new StringBuilder();
        StringBuilder monthPart = new StringBuilder();
        StringBuilder dayOfWeekPart = new StringBuilder();
        StringBuilder yearPart = new StringBuilder();


        secondsPart.append(seconds);
        minutesPart.append(minutes);

        if (ScheduleDetail.Frequency.HOURLY == frequency) {
            hoursPart.append('*');
        } else {
            hoursPart.append(hours);
        }

        if (ScheduleDetail.Frequency.WEEKLY == frequency) {
            dayOfWeekPart.append('?');
            dayOfMonthPart.append(dayOfMonth).append("/7"); //every 7 days
            monthPart.append(month).append("/1");
            yearPart.append(year).append("/1");

        } else if (ScheduleDetail.Frequency.DAILY == frequency) {
            dayOfMonthPart.append('*');
            monthPart.append('*');
            dayOfWeekPart.append('?');
        } else {
            dayOfMonthPart.append(dayOfMonth);
        }

        if (ScheduleDetail.Frequency.MONTHLY == frequency) {
            monthPart.append(month).append("/1");
            yearPart.append(year).append("/1");
            dayOfWeekPart.append('?');
        } else if (ScheduleDetail.Frequency.QUARTERLY == frequency) {
            monthPart.append(month).append("/3");
            yearPart.append(year).append("/1");
            dayOfWeekPart.append('?');

        }
        StringBuilder cronExpression = new StringBuilder();
        cronExpression.append(secondsPart).append(' ').append(minutesPart).append(' ').append(hoursPart).append(' ').append(dayOfMonthPart).append(' ').append(monthPart).append(' ').append(dayOfWeekPart).append(' ').append(yearPart);
        return cronExpression.toString().trim();
    }
```

```java
public class ScheduleConverterTest {

    @Test
    public void testCron1() throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat("MM-dd-yyyy hh:mm:ss");
        Date inputDate = dateFormat.parse("01-01-2018 09:35:02");

        ScheduleDetail scheduleDetail = new ScheduleDetail(inputDate, ScheduleDetail.Frequency.DAILY);
        String cron = ScheduleConverter.toCronExpression(scheduleDetail);
        Assert.assertEquals(cron, "2 35 9 * * ?");
    }

    @Test
    public void testCron2() throws ParseException {

        DateFormat dateFormat = new SimpleDateFormat("MM-dd-yyyy hh:mm:ss");
        Date inputDate = dateFormat.parse("01-02-2018 09:35:02"); //Tuesday

        ScheduleDetail scheduleDetail = new ScheduleDetail(inputDate, ScheduleDetail.Frequency.WEEKLY);
        String cron = ScheduleConverter.toCronExpression(scheduleDetail);
        Assert.assertEquals(cron, "2 35 9 2/7 1/1 ? 2018/1");

    }

    @Test
    public void testCronMidWeek() throws ParseException {

        DateFormat dateFormat = new SimpleDateFormat("MM-dd-yyyy hh:mm:ss");
        Date inputDate = dateFormat.parse("01-09-2018 09:35:02"); //Tuesday

        ScheduleDetail scheduleDetail = new ScheduleDetail(inputDate, ScheduleDetail.Frequency.WEEKLY);
        String cron = ScheduleConverter.toCronExpression(scheduleDetail);
        Assert.assertEquals(cron, "2 35 9 9/7 1/1 ? 2018/1");

    }


    @Test
    public void testCron3() throws ParseException {

        DateFormat dateFormat = new SimpleDateFormat("MM-dd-yyyy hh:mm:ss");
        Date inputDate = dateFormat.parse("01-01-2018 09:35:02");

        ScheduleDetail scheduleDetail = new ScheduleDetail(inputDate, ScheduleDetail.Frequency.MONTHLY);
        String cron = ScheduleConverter.toCronExpression(scheduleDetail);
        Assert.assertEquals(cron, "2 35 9 1 1/1 ? 2018/1");
    }


    @Test
    public void testQuarterly() throws ParseException {

        DateFormat dateFormat = new SimpleDateFormat("MM-dd-yyyy hh:mm:ss");
        Date inputDate = dateFormat.parse("01-01-2018 09:35:02");

        ScheduleDetail scheduleDetail = new ScheduleDetail(inputDate, ScheduleDetail.Frequency.QUARTERLY);
        String cron = ScheduleConverter.toCronExpression(scheduleDetail);
        Assert.assertEquals(cron, "2 35 9 1 1/1 ? 2018/1");
    }
}

```