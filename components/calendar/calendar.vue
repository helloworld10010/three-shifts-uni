<template>
    <view style="height: 100%">
        <view class="header space-between" v-if="showHeader">
            <view class="iconfont" data-id="0" @tap="dataBefor">
                <view class="left-color"></view>
            </view>
            <!-- <view class="btn flex-center" @tap="dateSelection">
                <view class="text">{{ selectDayFun }}???</view>
                <view class=""></view>
            </view> -->
            <view class="iconfont" data-id="1" @tap="dataBefor">
                <view class="right-color"></view>
            </view>
        </view>
        <view v-if="!calShow" :class="(isOpen ? '' : 'calendar-box') + ' ' + (dateShow ? 'active' : '')">
            <view :class="'calendar-wrapper ' + (dateShow ? 'active' : '')">
                <view class="calendar-panel">
                    <view v-if="sch.length > 0" class="checkalllbtn" @tap="checkall">
                        {{ sch }}
                    </view>
                    <view class="date-befor" data-id="0" data-type="month" @tap="dataBefor">
                        <view class="iconfont">
                            <view class="left-color"></view>
                        </view>
                    </view>
                    <view class="calendar-panel-box">
                        <view>{{ canlender.year }}年</view>
                        <view>{{ canlender.month }}月</view>
                    </view>
                    <view class="date-after" data-id="1" data-type="month" @tap="dataBefor">
                        <view class="iconfont">
                            <view class="right-color"></view>
                        </view>
                    </view>
                    <view class="backtoday" @tap="backtoday">回到今天</view>
                </view>
                <view :class="'calendar-header ' + (mini ? 'mini' : '')">
                    <view>日</view>
                    <view>一</view>
                    <view>二</view>
                    <view>三</view>
                    <view>四</view>
                    <view>五</view>
                    <view>六</view>
                </view>
                <view class="calendar-body">
                    <block v-for="(weeks, week) in canlender.weeks" :key="week">
                        <view class="calender-body-date-week">
                            <block v-for="(day, index) in weeks" :key="index">
                                <!-- <view wx:if="{{filters.strcontains(lockDay,canlender.year+'-'+filters.formatNumber(day.month) +'-'+filters.formatNumber(day.date))}}" class="date  date-lock">
              {{day.date}}
            </view> -->

                                <view
                                    :class="
                                        'date ' +
                                        (canlender.month == day.month && (nowMonth === day.month || day.month - nowMonth > 0) ? '' : 'placeholder') +
                                        ' ' +
                                        (day.checked && canlender.month == day.month ? 'date-current' : '')
                                    "
                                    :data-week="week"
                                    :data-index="index"
                                    :data-ischeck="canlender.month == day.month && ((nowMonth === day.month && day.date - nowDate >= 0) || day.month - nowMonth > 0)"
                                    @tap="selectDayFun"
                                >
                                    {{ day.date }}
                                    <view v-if="day.time <= nowDateTime && day.shouldsign" :class="'data-dot ' + (day.signed ? 'signed' : 'absent')"></view>
                                </view>
                            </block>
                        </view>
                    </block>
                </view>
                <!-- <view wx:if="{{!readonly}}" class="packup flex flex-between">
      <view class="packup-button" bindtap='clear'>清空</view>
      <view class="packup-button border-left" bindtap='packup'>确定</view>
    </view> -->
            </view>
        </view>
    </view>
</template>
// <script module="filters" lang="wxs">
// var strcontains = function (str, c) {
//   if (str.indexOf(c) >= 0) {
//     return true
//   } else {
//     return false
//   }
// }

// var formatNumber = function (n) {
//   n = n.toString()
//   return n[1] ? n : '0' + n
// }

// module.exports = {
//   formatNumber: formatNumber,
//   strcontains: strcontains
// }
// </script>
<script>
// components/calendar/calendar.js
const app = getApp();
export default {
    data() {
        return {
            calShow: true,

            // 日历组件是否打开
            dateShow: false,

            // 日期是否选择
            selectDay: '',

            // 当前选择日期
            selectDays: [],

            //多选日期
            canlender: {
                weeks: [],
                month: '',
                date: '',
                day: '',
                year: ''
            },

            currentMonth: null,
            nowMonth: new Date().getMonth() + 1,
            nowDate: new Date().getDate(),
            nowDateTime: new Date().getTime(),
            sch: '',
            week: '',
            weeks: [],

            day: {
                month: '',
                checked: '',
                date: 0,
                time: 0,
                shouldsign: '',
                signed: true
            },

        };
    },
    options: {
        styleIsolation: 'apply-shared',
        pureDataPattern: /^_/
    },
    props: {
        // date: {
        //     type: Number,
        //     default: Date.now()
        // },
        /**
         * 选中的日期
         */
        selected: {
            type: Array,
            default: () => []
        },
        /**
         * 锁定日期
         */
        lockDay: {
            type: String
        },
        /**
         * 是否默认展开, 可以调用open事件展开组件
         */
        isOpen: {
            type: Boolean,
            default: false
        },
        /**
         * 是否多选日期
         */
        multiple: {
            type: Boolean,
            default: true
        },
        /**
         * 是否展示头部
         */
        showHeader: {
            type: Boolean,
            default: false
        },
        /**
         * 是否不能修改
         */
        readonly: {
            type: Boolean,
            default: false
        },
        /**
         * 使用mini样式
         */
        mini: {
            type: Boolean,
            default: false
        },
        calendarSignInfoAndMark: {
            type: String,
            default: ''
        },
        anchorDateMS: {
            type: Number,
            default: 0
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    destroyed() {
        console.log('calendar detached');
    },
    /**
     * 组件的方法列表
     */
    methods: {
        ready() {
            console.log('calendar ready');
            let sdays = this.selected;
            sdays.forEach((v) => {
                this.selectDays.push(v);
            });
            this.getWeek(new Date());
            if (this.isOpen) {
                this.setData({
                    calShow: false,
                    dateShow: true
                });
            }
            this.backtoday();
        },

        onPageShow() {
            console.log('calendar show');
            this.backtoday(new Date());
        },

        onPageHide() {
            console.log('calendar hide');
        },

        attached() {
            console.log('calendar attached');
        },

        open: function (date) {
            if (date && date != '') {
                this.selectDays = []; //打开时候显示一个
                this.selectDays.push(date);
                this.setData({
                    selectDays: this.selectDays
                });
                this.getWeek(date);
            }
            this.dateSelection();
        },

        close: function () {
            this.packup();
        },

        clear: function () {
            this.setData({
                selectDays: []
            });
            let m;
            if (this.currentMonth == null) {
                m = this.canlender.month;
            } else {
                m = this.currentMonth;
            }
            let year = this.canlender.year + '-' + m + '-' + this.canlender.date;
            let _date = this.getDate(year, 0);
            this.getWeek(_date);
            this.$emit('clear', {
                detail: {}
            });
        },

        dateSelection() {
            if (this.isOpen) {
                return;
            }
            let self = this;
            if (self.calShow) {
                self.setData(
                    {
                        calShow: false
                    },
                    () => {
                        setTimeout(() => {
                            self.setData(
                                {
                                    dateShow: true
                                },
                                () => {
                                    self.$emit('show', {
                                        detail: {
                                            ischeck: !self.calShow
                                        }
                                    });
                                }
                            );
                        }, 100);
                    }
                );
            } else {
                self.setData(
                    {
                        dateShow: false
                    },
                    () => {
                        setTimeout(() => {
                            self.setData(
                                {
                                    calShow: true
                                },
                                () => {
                                    self.$emit('show', {
                                        detail: {
                                            ischeck: !self.calShow
                                        }
                                    });
                                }
                            );
                        }, 300);
                    }
                );
            }
        },

        //选择日期
        selectDayFun(e) {
			console.log("selectDayFun")
            if (this.readonly) {
                return;
            }
            let index = e.currentTarget.dataset.index;
            let week = e.currentTarget.dataset.week;
            let ischeck = e.currentTarget.dataset.ischeck;
            let canlender = this.canlender;
			let month,date;
            // if (!ischeck) return false;
            if (canlender.weeks[week][index].month < 10) {
                month = '0' + canlender.weeks[week][index].month;
            } else {
                month = canlender.weeks[week][index].month;
            }
            if (canlender.weeks[week][index].date < 10) {
                date = '0' + canlender.weeks[week][index].date;
            } else {
                date = canlender.weeks[week][index].date;
            }
            let datestr = canlender.year + '-' + month + '-' + date;
            let selectDays = this.selectDays;
            if (selectDays.indexOf(datestr) == -1) {
                if (this.multiple) {
                    //多选
                    selectDays.push(datestr);
                } else {
                    selectDays = []; //只保留最后选的
                    selectDays.push(datestr);
                }
            } else {
                let index = selectDays.indexOf(datestr);
                selectDays.splice(index, 1);
            }
            selectDays.sort(function (a, b) {
                let ds1 = a.split('-');
                let ds2 = b.split('-');
                let d1 = new Date(ds1[0], ds1[1], ds1[2]);
                let d2 = new Date(ds2[0], ds2[1], ds2[2]);
                return d1 - d2;
            });
            this.selectDays = selectDays;
            this.getWeek(datestr);
            let calendarSignInfoAndMarkObj = app.globalData.calendarSignInfoAndMarkObj;
            if (!(this.dateStrCutToMonthStr(datestr) in calendarSignInfoAndMarkObj)) {
                calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)] = {
                    shouldsigncount: 0,
                    signedcount: 0
                };
            }
            if (!(datestr in calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)])) {
                calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)][datestr] = {
                    signed: false,
                    mark: ''
                };
            }
            let dateinfoobj = calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)][datestr];
            let mark = dateinfoobj['mark'];
            this.$emit('getDate', {
                detail: {
                    selecteddate: datestr,
                    selecteddatemark: mark
                }
            });
        },

        //点击确定
        packup() {
            let self = this;
            self.$emit('select', {
                detail: {
                    selectDays: self.selectDays
                }
            });
            if (this.isOpen) {
                let year = self.canlender.year + '-' + self.canlender.month + '-' + self.canlender.date;
                let _date = self.getDate(year, 0);
                self.getWeek(_date);
                return;
            }
            self.setData(
                {
                    dateShow: false
                },
                () => {
                    setTimeout(() => {
                        self.setData(
                            {
                                calShow: true
                            },
                            () => {
                                let year = self.canlender.year + '-' + self.canlender.month + '-' + self.canlender.date;
                                let _date = self.getDate(year, 0);
                                self.getWeek(_date);
                            }
                        );
                    }, 300);
                }
            );
        },

        /**
         * 页面通过事件选中日期后调用refresh方法刷新日历组件
         */
        refresh: function (num) {
            this.setData({
                selectDays: num
            });
            let m;
            if (this.currentMonth == null) {
                m = this.canlender.month;
            } else {
                m = this.currentMonth;
            }
            let year = this.canlender.year + '-' + m + '-' + this.canlender.date;
            let _date = this.getDate(year, 0);
            this.getWeek(_date);
        },

        // 返回今天
        backtoday() {
            if (!this.readonly) {
                let datestr = this.dateStr(new Date());
                if (!this.multiple) {
                    this.selectDays = [];
                }
                this.selectDays.push(datestr);
                this.setData({
                    selectDays: this.selectDays
                });
                let calendarSignInfoAndMarkObj = app.globalData.calendarSignInfoAndMarkObj;
                console.log('当前日期数据：' + app.globalData.calendarSignInfoAndMarkObj);
                if (!(this.dateStrCutToMonthStr(datestr) in calendarSignInfoAndMarkObj)) {
                    calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)] = {
                        shouldsigncount: 0,
                        signedcount: 0
                    };
                }
                let monthObj = calendarSignInfoAndMarkObj[this.dateStrCutToMonthStr(datestr)];
                if (!(datestr in monthObj)) {
                    monthObj[datestr] = {
                        signed: true,
                        mark: ''
                    };
                }
                let dateinfoobj = monthObj[datestr];
                let mark = dateinfoobj['mark'];
                this.$emit('getDate', {
                    detail: {
                        selecteddate: datestr,
                        selecteddatemark: mark,
                        shouldsigncount: monthObj['shouldsigncount'],
                        signedcount: monthObj['signedcount']
                    }
                });
            }
            this.getWeek(new Date());
        },

        // 前一天|| 后一天
        dataBefor(e) {
            let num = 0;
            let types = e.currentTarget.dataset.type;
            if (e.currentTarget.dataset.id === '0') {
                num = -1;
            } else {
                num = 1;
            }
            let year = this.canlender.year + '-' + this.canlender.month + '-' + this.canlender.date;
            let _date = this.getDate(year, num, types === 'month' ? 'month' : 'day');
            this.getWeek(_date);
        },

        // 获取日历内容
        getWeek(dateData) {
            var cobj = this;
            let selected = this.selected;
            let selectDays = this.selectDays;
            // 判断当前是 安卓还是ios ，传入不容的日期格式
            if (typeof dateData !== 'object') {
                dateData = dateData.replace(/-/g, '/');
            }
            let _date = new Date(dateData);
            let year = _date.getFullYear(); //年
            let month = _date.getMonth() + 1; //月
            let date = _date.getDate(); //日
            let day = _date.getDay(); // 天
            let canlender = [];
            console.log('当前日历内容：year:' + year + ' month:' + month + ' date:' + date);
            let dates = {
                firstDay: new Date(year, month - 1, 1).getDay(),
                lastMonthDays: [],
                // 上个月末尾几天
                currentMonthDys: [],
                // 本月天数
                nextMonthDays: [],
                // 下个月开始几天
                endDay: new Date(year, month, 0).getDay(),
                weeks: []
            };
            console.log('dates.firstDay:' + dates.firstDay);
            console.log('dates.endDay:' + dates.endDay);
            // 循环上个月末尾几天添加到数组
            for (let i = dates.firstDay; i > 0; i--) {
                let dd = new Date(year, month - 1, -(i - 1));
                let checked = false;
                selectDays.forEach((v) => {
                    let selDate = v.split('-');
                    let ddstr = this.dateStr(dd);
                    if (ddstr == v) {
                        checked = true;
                    }
                });
                dates.lastMonthDays.push({
                    date: dd.getDate() + '',
                    month: month - 1,
                    time: dd.getTime(),
                    signed: true,
                    shouldsign: false,
                    checked
                });
            }
            // 循环本月天数添加到数组
            let calendarSignInfoAndMarkObj = app.globalData.calendarSignInfoAndMarkObj;
            if (!(cobj.monthStr(new Date(year, month - 1)) in calendarSignInfoAndMarkObj)) {
                calendarSignInfoAndMarkObj[cobj.monthStr(new Date(year, month - 1))] = {
                    shouldsigncount: 0,
                    signedcount: 0
                };
            }
            let monthObj = calendarSignInfoAndMarkObj[cobj.monthStr(new Date(year, month - 1))];
            monthObj['shouldsigncount'] = 0;
            monthObj['signedcount'] = 0;
            for (let i = 1; i <= new Date(year, month, 0).getDate(); i++) {
                let checked = false;
                // for (let j = 0; j < selected.length; j++) {
                //   let selDate = selected[j].split('-');
                //   if (Number(year) === Number(selDate[0]) && Number(month) === Number(selDate[1]) && Number(i) === Number(selDate[2])) {
                //     have = true;
                //   }
                // }
                selectDays.forEach((v) => {
                    let selDate = v.split('-');
                    if (Number(year) === Number(selDate[0]) && Number(month) === Number(selDate[1]) && Number(i) === Number(selDate[2])) {
                        checked = true;
                    }
                });
                let everyDate = new Date(year, month - 1, i);
                let diff = app.globalData.getDaysBetween(new Date(this.anchorDateMS), everyDate);
                if (checked) {
                    cobj.setData({
                        sch: app.globalData.getSchedulesStr(diff)
                    });
                    console.log('当前选中时间差：' + diff + '天');
                    console.log('最终班次结果index：' + (diff + 3 + app.globalData.anchor));
                }
                let shouldsign = (diff + 3 + app.globalData.anchor) % 3 != 2;
                if (!(cobj.dateStr(everyDate) in monthObj)) {
                    monthObj[cobj.dateStr(everyDate)] = {
                        signed: true,
                        mark: ''
                    };
                }
                let sp = monthObj[cobj.dateStr(everyDate)];
                let signed = sp['signed'];
                if (shouldsign) {
                    monthObj['shouldsigncount'] = monthObj['shouldsigncount'] + 1;
                }
                if (signed) {
                    monthObj['signedcount'] = monthObj['signedcount'] + 1;
                }
                dates.currentMonthDys.push({
                    date: i + '',
                    month: month,
                    time: everyDate.getTime(),
                    signed: signed,
                    shouldsign: (diff + 3 + app.globalData.anchor) % 3 != 2,
                    // have,
                    checked
                });
            }
            this.$emit('getMonth', {
                detail: {
                    // 月度信息
                    shouldsigncount: monthObj['shouldsigncount'],
                    signedcount: monthObj['signedcount']
                }
            });

            // 循环下个月开始几天 添加到数组
            for (let i = 1; i < 7 - dates.endDay; i++) {
                dates.nextMonthDays.push({
                    date: i + '',
                    month: month + 1,
                    // 用于显示的月份都得加一
                    time: new Date(year, month, i).getTime(),
                    signed: true,
                    shouldsign: false
                });
            }
            canlender = canlender.concat(dates.lastMonthDays, dates.currentMonthDys, dates.nextMonthDays);
            // 拼接数组  上个月开始几天 + 本月天数+ 下个月开始几天
            for (let i = 0; i < canlender.length; i++) {
                if (i % 7 == 0) {
                    dates.weeks[parseInt(i / 7)] = new Array(7); // 第几行
                }

                dates.weeks[parseInt(i / 7)][i % 7] = canlender[i];
            }
            // 渲染数据
            this.setData({
                selectDay: month + '月' + date + '日',
                'canlender.weeks': dates.weeks,
                'canlender.month': month,
                'canlender.date': date,
                'canlender.day': day,
                'canlender.year': year
            });
            month = month < 10 ? '0' + month : month;
            date = date < 10 ? '0' + date : date;
            // let localstr = year+"-"+month+"-"+date;
            // this.triggerEvent('getdate', {
            //   year,
            //   month,
            //   date,
            //   localstr
            // })
        },

        checkall() {
            let date = new Date();
            let m;
            let d;
            let day;
            let dayNum;
            let days = [];
            if (this.currentMonth == null) {
                m = parseInt(this.nowMonth);
            } else {
                m = parseInt(this.currentMonth);
            }
            switch (m) {
                case 1:
                case 3:
                case 5:
                case 7:
                case 8:
                case 10:
                case 12:
                    dayNum = 31;
                    break;
                case 4:
                case 6:
                case 9:
                case 11:
                    dayNum = 30;
                    break;
                case 2:
                    if ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0) {
                        dayNum = 29;
                    } else {
                        dayNum = 28;
                    }
                    break;
            }
            for (d = 1; d <= dayNum; d++) {
                date.setMonth(m - 1, d);
                day = date.getDay();
                let str = this.format(date);
                if (date.getMonth() == new Date().getMonth()) {
                    if (date.getDate() - new Date().getDate() >= 0) {
                        days.push(str);
                    }
                } else if (date.getMonth() > new Date().getMonth()) {
                    days.push(str);
                }
            }
            this.refresh(days);
            this.$emit('checkAll', {
                detail: {
                    days
                }
            });
        },

        /**
         * 时间计算
         */
        getDate(date, AddDayCount, str = 'day') {
            if (typeof date !== 'object') {
                date = date.replace(/-/g, '/');
            }
            let dd = new Date(date);
            switch (str) {
                case 'day':
                    dd.setDate(dd.getDate() + AddDayCount); // 获取AddDayCount天后的日期
                    break;
                case 'month':
                    dd.setMonth(dd.getMonth() + AddDayCount); // 获取AddDayCount天后的日期
                    break;
                case 'year':
                    dd.setFullYear(dd.getFullYear() + AddDayCount); // 获取AddDayCount天后的日期
                    break;
            }
            let y = dd.getFullYear();
            let m = dd.getMonth() + 1 < 10 ? '0' + (dd.getMonth() + 1) : dd.getMonth() + 1; // 获取当前月份的日期，不足10补0
            let d = dd.getDate() < 10 ? '0' + dd.getDate() : dd.getDate(); // 获取当前几号，不足10补0
            this.setData({
                currentMonth: m
            });
            return y + '-' + m + '-' + d;
        },

        monthStr(calendar) {
            let year = calendar.getFullYear();
            let month = calendar.getMonth() + 1;
            month = month < 10 ? '0' + month : month;
            let monthstr = year + '-' + month;
            return monthstr;
        },

        dateStrCutToMonthStr(dateStr) {
            if (dateStr < 10) {
                return;
            }
            return dateStr.substr(1, 7);
        },

        //日期转字符串
        dateStr(calendar) {
            let year = calendar.getFullYear();
            let month = calendar.getMonth() + 1;
            let date = calendar.getDate();
            month = month < 10 ? '0' + month : month;
            date = date < 10 ? '0' + date : date;
            let datestr = year + '-' + month + '-' + date;
            return datestr;
        },

        format(dd) {
            let y = dd.getFullYear();
            let m = dd.getMonth() + 1 < 10 ? '0' + (dd.getMonth() + 1) : dd.getMonth() + 1;
            let d = dd.getDate() < 10 ? '0' + dd.getDate() : dd.getDate();
            return y + '-' + m + '-' + d;
        }
    },
    created: function () {},
    watch: {
        selected: {
            handler: function (newVal, oldVal) {
                this.getWeek(new Date());
            },

            immediate: true,
            deep: true
        }
    }
};
</script>
<style>
@import './calendar.css';
</style>
