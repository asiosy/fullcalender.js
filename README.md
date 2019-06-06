# fullcalender.js

FullCalendar作为一款功能强大的日历插件，用途十分广泛。今天我们用它来实现一个简单的节假日设置功能，代码非常简洁，有助于理解插件的调用方式。本文代码需用到SpringMVC和Mysql。

![在这里插入图片描述](https://img-blog.csdn.net/20181018222446490?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM2MjM5Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
**1、在官网下载fullCalendar.js（LZ的js和样式都是改过的，可以在文末下载）**
**2、创建页面**
  - 2.1定义一个div：
```
 <div id="calendar" style="width: 600px"></div>
```
-  2.2在Script中调用fullCalendar插件，脚本如下：
```
 function calendarInit(){
    $('#calendar').fullCalendar({
        theme: false,
        header: {
            left: 'prev,next',
            center: 'title',
            right: 'today'//,agendaWeek,agendaDay
        },
        monthNames: ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"],
        monthNamesShort: ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"],
        dayNames: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
        dayNamesShort: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
        today: ["今天"],
        firstDay: 1,
        buttonText: {
            today: '本月',
            month: '月',
            week: '周',
            day: '日',
            prev: '<<上一月',
            next: '下一月>>'
        },
        buttonIcons:false,
        viewDisplay: function (view) {//按照月份动态查询数据
            $("td .fc-widget-content").css('background-color', 'transparent');
            $("#hidDay").val('');
            viewStart = $.fullCalendar.formatDate(view.start, "yyyy-MM-dd");
            viewEnd = $.fullCalendar.formatDate(view.end, "yyyy-MM-dd");
            getData();//获取当前页面节假日数据
        },
        editable: false,//判断该日程能否拖动
        dayClick: function (date, allDay, jsEvent, view) {//日期点击事件
            changeColor(date,this);//改变背景颜色
        }
    });
}
```

**3、初始化节假日，默认初始化近五年**
![在这里插入图片描述](https://img-blog.csdn.net/20181018222639879?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM2MjM5Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 - 3.1年份下拉框脚本： 
```
for (var i = 1990; i <= 2050; i++) {
    selStartYear.add(new Option(i + "年", i));
    selEndYear.add(new Option(i + "年", i));
}

```
 - 3.2前后端数据交互脚本：
```
function initData() {
    vDialogInitData.hide();
    showMask(1000);
    var postData = "iSYear=" + $("#selStartYear").val() + "&iEYear=" + $("#selEndYear").val();
    $.post("InitHolidayData", postData, function (response) {
        if (response) {
            getData();
            alert("初始化成功！");
        }
        else {
            alert("初始化失败！");
        }
        hideMask();
    })
}
```

 - 3.3 Java初始化节假日代码：
```
public boolean initHoliday(Integer iSYear, Integer iEYear){
    afwfdayoffDao.deleteInitModel(iSYear,iEYear);//删除已有数据
    Date dStart=Function.stringToDate(iSYear+"-01-01","yyyy-MM-dd");//格式化起始时间
    Date dEnd=Function.stringToDate(iEYear+"-12-31","yyyy-MM-dd");//格式化结束时间
    while(dStart.compareTo(dEnd)<0){//循环遍历待初始化的时间日期
        Afwfdayoff model=new Afwfdayoff();//节假日数据模型
        Calendar cal = Calendar.getInstance();
        cal.setTime(dStart);
        if(cal.get(Calendar.DAY_OF_WEEK) == Calendar.SATURDAY || cal.get(Calendar.DAY_OF_WEEK) == Calendar.SUNDAY){
           model.name="周末";
           model.sort=1;
        }
        if(cal.get(Calendar.DAY_OF_MONTH) ==1){ //判断节假日类型
           switch (cal.get(Calendar.MONTH)){
               case 0:
                   model.name="元旦";
                   model.sort=2;
                   break;
               case 4:
                   model.name="劳动节";
                   model.sort=2;
                   break;
               case 9:
                   model.name="国庆节";
                   model.sort=2;
                   break;
           }
        }
        if(model.sort!=null) {//保存节假日数据
            model.dater = dStart;
            afwfdayoffDao.insert(model);
        }
        cal.add(Calendar.DATE,1);//当前日期加1
        dStart=cal.getTime();
    }
     return true;
}
```

 - 3.4数据库中初始化结果示例：
 ![在这里插入图片描述](https://img-blog.csdn.net/201810182228452?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM2MjM5Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**4、节假日设置**
 ![在这里插入图片描述](https://img-blog.csdn.net/20181018222916992?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM2MjM5Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
public boolean SetWorkday(String dates,Integer iType,String sName) {
    String[] arrDay = dates.split(",");//获取待设置的时间数组
    int iLength = arrDay.length;
    if (iLength > 0) {
        for (int ii = 0; ii < iLength; ii++) {//循环遍历
            if (arrDay[ii].length() > 0) {
                afwfdayoffDao.deleteByDays(arrDay[ii]);//删除已有数据
                if (iType != 0) {
                    Afwfdayoff model = new Afwfdayoff();
                    if (sName.trim().length() == 0) {
                        if (iType == 2) {
                            sName = "节假日";
                        } else {
                            sName = "调休";
                        }
                    }
                    model.name = sName;
                    model.dater = Function.stringToDate(arrDay[ii], "yyyy-MM-dd");
                    model.sort = iType;
                    afwfdayoffDao.insert(model);//保存节假日信息
                }
            }
        }
    }
    return true;
}
```

**最终效果图：**
 ![在这里插入图片描述](https://img-blog.csdn.net/20181018222944159?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM2MjM5Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Ps：该代码设置好节假日数据，也可以用于日程管理，人事考勤，项目周期计算等应用中。

