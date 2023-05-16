<template>
    <!-- index.wxml -->
    <view class="container">
        <text class="desc-text">{{ nowDateStr }} 今天你该：</text>
        <text class="current-text">{{ current }}</text>
        <text class="tip-text" @tap.stop.prevent="actionFix">如显示班次错误，点击</text>

        <view :class="'absent-btn ' + (signed ? 'signed-btn' : 'absent-btn')" @tap.stop.prevent="signIn">
			<text style="color: azure">{{getActionsBtnText}}</text>
        </view>

        <!-- <view style="position: fixed;bottom: 50rpx; width: 100%;height: auto">
    <button type="primary" style="width: 300rpx;font-size: small;font-weight: 500;" catchtap="actionFix">修正</button>
  </view> -->
    </view>
</template>

<script>
// index.js
const app = getApp();
let currentDate = new Date();
var arr = ['白班', '夜班', '休息'];
function getDaysBetween(date1, date2) {
    var tdate1 = date1;
    tdate1.setHours(0);
    tdate1.setMinutes(0);
    tdate1.setSeconds(0);
    tdate1.setMilliseconds(0);
    var tdate2 = date2;
    tdate2.setHours(0);
    tdate2.setMinutes(0);
    tdate2.setSeconds(0);
    tdate2.setMilliseconds(0);
    if (tdate1 > tdate2) {
        uni.showToast({
            title: '不支持向前查询',
            duration: 2000
        });
        return 0;
    }
    var days = (tdate2 - tdate1) / (1440 * 60 * 1000);
    console.log('时间差' + days + '天');
    return Math.trunc(days);
}
export default {
    data() {
        return {
            current: '第一次使用请先修正',
            nowDateStr: '',
            signed: true,
            anchorDate: '',
        };
    },
	computed:{
		getActionsBtnText(){
			return this.signed?'翘班':'休息咯'
		}
	},
    onReady() {
        this.init();
    },
    onShow() {
        console.log('indexonShow.........................................');
    },
    methods: {
        signIn: function () {
            if (!this.signed) {
                uni.showToast({
                    title: '无需继续操作'
                });
            } else {
                this.setData({
                    signed: false
                });
                let calendarSignInfoAndMarkObj = app.globalData.calendarSignInfoAndMarkObj;
                let monthStr = app.globalData.monthStr(new Date());
                if (!(app.globalData.monthStr(new Date()) in calendarSignInfoAndMarkObj)) {
                    calendarSignInfoAndMarkObj[monthStr] = {
                        shouldsigncount: 0,
                        signedcount: 0
                    };
                }
                let monthObj = calendarSignInfoAndMarkObj[monthStr];
                if (!(app.globalData.dateStr(new Date()) in monthObj)) {
                    monthObj[app.globalData.dateStr(new Date())] = {
                        signed: false,
                        mark: ''
                    };
                }
                var dateObj = monthObj[app.globalData.dateStr(new Date())];
                dateObj['signed'] = false;
                uni.showToast({
                    title: '翘班成功',
                    icon: 'success'
                });
            }
        },

        actionFix: function () {
            var that = this;
            uni.showActionSheet({
                itemList: ['白班', '夜班', '休息'],
                success(res) {
                    console.log(res.tapIndex);
                    uni.setStorageSync('anchorDate', new Date());
                    uni.setStorageSync('anchor', res.tapIndex);
                    app.globalData.dataInit(); // 需要重新初始化数据

                    that.init();
                },
                fail(res) {
                    // wx.showToast({
                    //   title: '未知错误',
                    //   icon:'error',
                    //   duration: 2000
                    // })
                    console.log(res.errMsg);
                }
            });
        },

        init: function () {
            console.log('init start....................');
            try {
                var anchorDate = uni.getStorageSync('anchorDate');
                var anchor = uni.getStorageSync('anchor');
                console.log('anchorDate:' + anchorDate);
                console.log('anchor:' + anchor);
                if (anchorDate) {
                    var anchorDate = new Date(anchorDate);
                    var now = new Date();
                    var diff = getDaysBetween(anchorDate, now);
                    console.log('时间差为：' + diff);
                    console.log('current:' + ((diff + anchor) % 3));
                    let dateObj = app.globalData.getDateDataObj(new Date());
                    this.setData({
                        current: arr[(diff + anchor) % 3],
                        nowDateStr: now.getFullYear() + '年' + (now.getMonth() + 1) + '月' + now.getDate() + '日',
                        anchorDate: anchorDate,
                        signed: (diff + anchor) % 3 == 2 || dateObj['signed']
                    });
                } else {
                    this.setData({
                        current: '首次使用请先修正'
                    });
                }
            } catch (e) {
                console.log('CatchClause', e);
                console.log('CatchClause', e);
                console.log(e);
                uni.showToast({
                    title: '未知错误',
                    icon: 'error',
                    duration: 2000
                });
            }
        }
    }
};
</script>
<style>
@import './index.css';
</style>
