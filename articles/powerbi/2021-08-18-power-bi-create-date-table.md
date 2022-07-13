# Power BI创建时间表

[![Screenshot of Date table](images/power-bi-date-table.png)](images/power-bi-date-table.png#lightbox)

**需求**： 为更好地从时间维度来分析数据，我们需要创建一个没有间隔的时间序列表，这个文章手把手完成时间序列表Date表格的创建。
首先，新建表，表命名为Date.
### Date列
创建Date列，要求日期序列没有间隔。例如，创建从7/1/2018到6/30/2022的时间序列，新建列，使用CALENDAR函数的DAX语法：
```
Date = CALENDAR("7/1/2018", "6/30/2022")
```
实际工作中，生成Calendar日期的区间可能由某个表的某列决定， 这个是时候就不需要hard code起始和结束日期了， 而是由以下表达式完成:
```
Date = CALENDAR(Min('Table1'[Released Date]), Max('Table1'[Released Date]))
```
### Year列
创建Year列，新建列，使用Year函数：
```
Year = YEAR('Date'[Date])
```

### Month列
创建Month列，新建列，使用Month函数:
```DAX
Month = MONTH('Date'[Date])
```

### Day列
创建Day列，新建列，使用Day函数：
```DAX
Day = DAY('Date'[Date])
```

### Quarter列
创建Quarter列，新建列，使用Quarter函数：
```DAX
Quarter = QUARTER('Date'[Date])
```

### Week Number列
创建Week Number列，新建列，使用WEEKNUM函数，函数第二个参数指定哪天为一周的第一天， 1 - 星期天为第一天， 2 - 星期一为第一天：
```DAX
Week Number = WEEKNUM('Date'[Date],2)
```

### Week Day Number列
创建Week Day Number列，新建列，使用WEEKDAY函数，函数第二个参数指定哪天为一周的第一天，1 - 星期天为第一天， 2 - 星期一为第一天：
```DAX
Week Day Number = WEEKDAY('Date'[Date],2)
```

### Day Number列
创建Day Number列，新建列，使用DATEDIFF函数计算当前日期与当年第一天的日间隔，别忘记最后+1:
```DAX
Day Number = 
DATEDIFF(DATE('Date'[Year],1,1),'Date'[Date],DAY)+1
```

### Week Day Name (English)列
创建Week Day Name (English) 列，新建列，使用SWITCH函数，确保表达式为TRUE()，然后在值和结果中输入不同条件下的结果：
```DAX
Week Day Name (English) = 
SWITCH(TRUE(),
'Date'[Week Day Number]=1,"Monday",
'Date'[Week Day Number]=2, "Tuesday",
'Date'[Week Day Number]=3, "Wednesday",
'Date'[Week Day Number]=4, "Thursday",
'Date'[Week Day Number]=5, "Friday",
'Date'[Week Day Number]=6, "Saturday",
'Date'[Week Day Number]=7, "Sunday")
```

### Week Day Name (Chinese)列
创建Week Day Name (Chinese)列， 新建列，同上Week Day Name (English)列：
```DAX
Week Day Name (Chinese) = 
SWITCH(TRUE(),
'Date'[Week Day Number]=1,"星期一",
'Date'[Week Day Number]=2, "星期二",
'Date'[Week Day Number]=3, "星期三",
'Date'[Week Day Number]=4, "星期四",
'Date'[Week Day Number]=5, "星期五",
'Date'[Week Day Number]=6, "星期六",
'Date'[Week Day Number]=7, "星期天")
```

### Month Name (English)列
创建Month Name (English)列，新建列，使用SWITCH函数，确保表达式为TRUE()，然后在值和结果中输入不同条件下的结果：
```DAX
Month Name (English) = 
SWITCH(true(),
'Date'[Month]=1,"January",
'Date'[Month]=2, "February",
'Date'[Month]=3, "March",
'Date'[Month]=4, "April",
'Date'[Month]=5, "May",
'Date'[Month]=6, "June",
'Date'[Month]=7, "July",
'Date'[Month]=8, "August",
'Date'[Month]=9, "September",
'Date'[Month]=10, "October",
'Date'[Month]=11,"November",
'Date'[Month]=12,"December")
```

### Month Name (Chinese)列
创建Month Name (Chinese)列，新建列，同上Month Name (English)列：
```DAX
Month Name (Chinese) = 
SWITCH(true(),
'Date'[Month]=1,"一月",
'Date'[Month]=2, "二月",
'Date'[Month]=3, "三月",
'Date'[Month]=4, "四月",
'Date'[Month]=5, "五月",
'Date'[Month]=6, "六月",
'Date'[Month]=7, "七月",
'Date'[Month]=8, "八月",
'Date'[Month]=9, "九月",
'Date'[Month]=10, "十月",
'Date'[Month]=11,"十一月",
'Date'[Month]=12,"十二月")
```

### Fiscal Year列
创建Fiscal Year列，在很多公司，财年开始于7月，即7/1/2019的Fiscal Year是FY20, 7/1/2019的Fiscal Quarter是FY20 Q1，新建列，使用IF函数来判断当前日期月份是否大于等于7，为真则FY Year为当前Year+1, 否则FY Year为当前Year, 再使用Right函数取年份后2位：
```DAX
Fiscal Year = 
IF('Date'[Month]>=7,
"FY"&RIGHT('Date'[Year]+1,2),
"FY"&RIGHT('Date'[Year],2))
```

也可以定义变量进行计算：
```dotnetcli
Fiscal Year = 
var YearNumber = 
SWITCH(true(),
[Month] >= 7, [Year]+1,
[Month] >= 1, [Year])
Return "FY"& Right(YearNumber,2)
```

### Fiscal Quarter列
创建Fiscal Month列，在很多公司，财年开始于7月，即：

|Quarter|	Fiscal Quarter|
|---|---|
|1|	FQ3|
|2|	FQ4|
|3|	FQ1|
|4|	FQ2|

新建列，使用SWITCH函数，确保表达式为TRUE()，然后在值和结果中输入不同条件下的结果：
```DAX
Fiscal Quarter = 
var QuarterNumber = 
SWITCH(true(),
'Date'[Quarter]=1,"Q3",
'Date'[Quarter]=2,"Q4",
'Date'[Quarter]=3,"Q1",
'Date'[Quarter]=4,"Q2")
return [Fiscal Year]&" "&QuarterNumber
```

也可通过Month进行判断：
```dotnetcli
Fiscal Quarter = 
var QuarterNumber = 
SWITCH(true(),
[Month] >= 10, "Q2",
[Month] >= 7, "Q1",
[Month] >= 4, "Q4",
[Month] >= 1, "Q3")
return [Fiscal Year]&" "&QuarterNumber
```

## 在视觉控件中让数据按照Fiscal Year, Quarter, Month正序显示
表格中的数据需要按月显示当月销售数据和当月YTD数据， 这个时候需要让时间列能够自然排序。有个敲门就是让月份显示的复杂一点，让Month字符串以如下格式显示：
`FY22 Q4 5-May`
修改Month Name(English)列的表达式如下：
```dotnetcli
Month Name(English) = 
var MonthName = 
SWITCH(true(),
[Month]=1, "Jan",
[Month]=2, "Feb",
[Month]=3, "Mar",
[Month]=4, "Apr",
[Month]=5, "May",
[Month]=6, "Jun",
[Month]=7, "Jul",
[Month]=8, "Aug",
[Month]=9, "Sep",
[Month]=10, "Oct",
[Month]=11, "Nov",
[Month]=12, "Dec")
RETURN [Fiscal Quarter] &" "&[Month]&"-"&MonthName
```
## DateKey列
`DateKey`列用于建立表之间关系， 可以精确到天。在Calendar表和事实表中都依据某个日期列建立DateKey序列列。
```
DateKey = FORMAT([Date],"yyyyMMdd")
```