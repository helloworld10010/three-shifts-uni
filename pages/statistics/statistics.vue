<template>
    <!-- pages/statistics/statistics.wxml -->
    <view>
        <calendar
            id="calendar"
            :calendarSignInfoAndMark="calendarSignInfoAndMark"
            :anchorDateMS="anchorDateMS"
            :multiple="false"
            :isOpen="true"
            :lockDay="lockday"
            @get-date="selectedDateHandle()"
            @get-month="selectedMonthHandle()"
            @check-all="checkall()"
            @update-data="updateData()"
        ></calendar>

        <view style="display: flex">
            <view class="card1">
                <view class="text-align-center">
                    <text style="font-size: small; color: #39b54a">本日备注</text>
                </view>

                <textarea
                    placeholder="点击输入.."
                    class="textarea small-text"
                    :value="selectedDateMark"
                    @input="textareaInput"
                    @confirm="textareaConfirm"
                    :show-confirm-bar="false"
                    maxlength="30"
                ></textarea>
            </view>

            <view class="card2">
                <view class="text-align-center">
                    <text style="font-size: small; color: #39b54a">本月统计</text>
                </view>
                <text style="display: block; font-size: smaller">应出勤：{{ shouldsigncount }}</text>
                <text style="display: block; font-size: smaller">已出勤：{{ signedcount }}</text>
            </view>
        </view>
    </view>
</template>

<script>
import calendar from '@/components/calendar/calendar';
// pages/statistics/statistics.js
const app = getApp();
export default {
    components: {
        calendar
    },
    data() {
        return {
            calendarSignInfoAndMark: '{}',
            anchorDateMS: 0,
            selectedDateStr_var: null,
            selectedDateMark: '',
            shouldsigncount: 0,
            signedcount: 0,
            lockday: ''
        };
    },
    options: {
        pureDataPattern: /^_/
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
        this.setData({
            anchorDateMS: app.globalData.anchorDateMS,
            calendarSignInfoAndMark: app.globalData.calendarSignInfoAndMark
        });
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady() {
        console.log('statistics onReady anchorDateMS:' + this.anchorDateMS);
        console.log('statistics app anchorDateMS' + app.globalData.anchorDateMS);
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow() {
        console.log("statistics onShow")
        let calendarSignInfoAndMarkObj = app.globalData.calendarSignInfoAndMarkObj
        if(!(app.globalData.monthStr(new Date()).substr(1,7) in calendarSignInfoAndMarkObj)){
          calendarSignInfoAndMarkObj[app.globalData.monthStr(new Date()).substr(1,7)] = {'shouldsigncount':0,'signedcount':0}
        }
        let monthdata = calendarSignInfoAndMarkObj[app.globalData.monthStr(new Date()).substr(1,7)]
        console.log("monthData:"+JSON.stringify(monthdata))
        this.setData({
          shouldsigncount:monthdata['shouldsigncount'],
          signedcount:monthdata['signedcount']
        })
    },
    /**
     * 生命周期函数--监听页面隐藏
     */
    onHide() {},
    methods: {
        selectedDateHandle: function (e) {
            // ---处理dataset begin--- */
            // this.handleDataset(e, _dataset);
            // ---处理dataset end---
            console.log('selectedDateHandle::' + JSON.stringify(e.detail));
            this.setData({
                selectedDateStr_var: e.detail.selecteddate,
                selectedDateMark: e.detail.selecteddatemark
            });
        },

        selectedMonthHandle: function (e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            console.log('selectedMonthHandle::' + JSON.stringify(e.detail));
            this.setData({
                shouldsigncount: e.detail.shouldsigncount,
                signedcount: e.detail.signedcount
            });
        },

        textareaConfirm(event) {
            // console.log("textarea 确定输入："+event.detail.value)
        },

        textareaInput(event) {
            console.log('textarea 输入东西：' + event.detail.value);
            var markobj = app.globalData.calendarSignInfoAndMarkObj; // 两边数据不同步
            var markobjdate = markobj[this._selectedDateStr.substr(1, 7)][this._selectedDateStr];
            markobjdate['mark'] = event.detail.value;
            this.setData({
                calendarSignInfoAndMark: JSON.stringify(markobj)
            });
            app.globalData.calendarSignInfoAndMark = JSON.stringify(markobj);
        },

        checkall(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            console.log('占位：函数 checkall 未声明');
        },

        updateData(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            console.log('占位：函数 updateData 未声明');
        }
    }
};
</script>
<style>
@import './statistics.css';
</style>
