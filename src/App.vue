<template>
  <div id="app" v-cloak>
    <van-nav-bar title="生日记录本" @click-right="onClickRight">
      <van-dropdown-menu slot='left'>
        <van-dropdown-item v-model="type" :options="option1" />
      </van-dropdown-menu>
      <van-icon name="plus" slot='right' />
    </van-nav-bar>
    
    <div class="data-list">
      <div>
        <van-swipe-cell v-for="(item, index) in showList" :key="index" class="van-hairline--bottom">
          <div class="list-content">
            <div class="user-info">
              <h3>{{item.name}} <span class="info-detail">属{{item.signZodiac}} {{item.constellation}}</span></h3>
              <p class="age">年龄: {{item.age}} 岁</p>
              <p>生日: {{item.birthday}} <span style="color: #417505;">{{item.isOld ? '(农历)' : '(新历)'}}</span></p>
            </div>
            <div class="birth-info">
              <div class="happy-birthday"  v-if="item.diffTime === 'happy'">
                <img src="./assets/img/cake.png" alt="">今天生日
              </div>
              <div v-else-if="item.diffTime > 0"><span class="day">{{item.diffTime}}</span>天后生日</div>
              <div class="out-date" v-else>
                <p>生日已过</p>
                <p class="next-birth">下个生日还有 <span class="day">{{item.nextDay}}</span>天</p>
              </div>
            </div>
          </div>
          <template #right>
            <van-button square type="primary" text="编辑" class="handle-btn" @click="editInfo(index)"/>
            <van-button square type="danger" text="删除" class="handle-btn" @click="deleteInfo(index)"/>
          </template>
        </van-swipe-cell>
        <div class="tips" v-show="showList.length === 0">暂时没有数据，快去添加吧~</div>
      </div>
    </div>
    
    <div class="input-area" v-show="isShow">
      <div class="title van-hairline--bottom">{{mode === 'add' ? '添加' : '编辑'}}生日</div>
      <div class="input-wrap">
        <div class="name">
          <label>
            <span>姓名：</span>
            <input type="text" placeholder="请输入姓名" v-model="name">
          </label>
        </div>
        <div class="date">
          <label>
            <span>生日：</span>
            <input type="text" placeholder="请选择生日" v-model="birthday" @click="openPicker" readonly>
          </label>
        </div>
        <div class="old-new">
          <van-radio-group v-model="radio" direction="horizontal">
            <van-radio name="1">新历</van-radio>
            <van-radio name="2">农历</van-radio>
          </van-radio-group>
        </div>
      </div>
      <van-button type="info" class="save-btn" @click="handleSave">保存</van-button>
      <div class="icon-wrap" @click="isShow = false">
        <van-icon name="close" />
      </div>
    </div>
    <div class="my-mask" v-show="isShow"></div>
    <van-popup position="bottom" v-model="dateShow" :style="{ height: '264px' }">
     <van-datetime-picker
        v-model="currentDate"
        type="date"
        title="请选择新历"
        :min-date="minDate"
        :max-date="maxDate"
        @confirm="choseTime"
        @cancel="cancelTime"
        v-if="radio === '1'"
      />
      <van-picker 
        show-toolbar
        title="请选择农历"
        :columns="lunarData" 
        @confirm="choseTime"
        @cancel="cancelTime"
        v-else
      />
    </van-popup>
  </div>
</template>

<script>
  import Vue from 'vue'
  import {calendar} from '@/assets/js/calendar.js'
  import BetterScroll from 'better-scroll';
  import { 
    NavBar, 
    Icon, 
    DropdownMenu, 
    DropdownItem, 
    SwipeCell, 
    Button, 
    Toast, 
    DatetimePicker, 
    Popup, 
    RadioGroup, 
    Radio,
    Dialog,
    Picker
  } from 'vant';
  
  Vue.use(NavBar);
  Vue.use(Icon);
  Vue.use(DropdownMenu);
  Vue.use(DropdownItem);
  Vue.use(SwipeCell);
  Vue.use(Button);
  Vue.use(Toast);
  Vue.use(DatetimePicker);
  Vue.use(Popup);
  Vue.use(RadioGroup);
  Vue.use(Radio);
  Vue.use(Dialog);
  Vue.use(Picker);
  
  export default {
      data() {
        return {
          option1: [
            { text: '全部', value: 1 },
            { text: '未过', value: 2 },
            { text: '已过', value: 3 },
            { text: '增序', value: 4 }
          ],
          isShow: false,
          list: [],
          name: '',
          birthday: '',
          type: 1,
          minDate: new Date(1910, 0, 1),
          maxDate: new Date(),
          currentDate: new Date(1995, 0, 1),
          dateShow: false,
          radio: "2",
          mode: 'add', //add是添加, edit是编辑
          editIndex: -1, //要编辑的索引
          lunarData: []
        };
      },
      
      computed: {
        showList() {
          const theList = this.list;
          switch (this.type) {
            case 1:
              return theList;
            break;
            case 2:
              return theList.filter(item => item.diffTime > 0 || item.diffTime === 'happy');
            break;
            case 3:
              return theList.filter(item => item.diffTime <= 0);
            break;
            case 4:
              const ret = theList.sort((fr, se) => fr.diffTime - se.diffTime);
              return ret.filter(item => item.diffTime > 0 || item.diffTime === 'happy');
            break;
          }
        }
      },
      
      methods: {
        onClickRight() {
            this.name = '';
            this.birthday = '';
            this.radio = '2';
            this.mode = 'add';
            this.isShow = true;
        },
        
        choseTime(_value) {
          const theTime = new Date(_value);
          const ret = theTime.getFullYear() + '-' + (theTime.getMonth() + 1) + '-' + theTime.getDate();
          this.birthday = ret;
          this.dateShow = false;
        },
        
        cancelTime() {
          this.editIndex = -1;
          this.dateShow = false;
        },
        
        /**
         * 打开时间选择
         */
        openPicker() {
          if (this.editIndex !== -1) {
            const birthday = (this.showList[this.editIndex].birthday).split('-');
            this.currentDate = new Date(birthday[0], birthday[1] - 1, birthday[2]);
          } else {
            this.currentDate = new Date(1995, 0, 1);
          }
          
          this.dateShow = true;
        },
        
        /**
         * 点击保存
         */
        handleSave() {
          if (this.mode === 'add') {
            this.addInfo();
          } else {
            this.saveEditInfo();
          }
        },
        
        /**
         * 计算年龄
         */
        calcAge(_day, _isOld) {
          const now = new Date();
          
          const year = now.getFullYear();
          const month = now.getMonth() + 1;
          const date = now.getDate();

          const theyTime = _day.split('-');
          
          let new_day_arr; //今年生日
          
          if (_isOld) { //如果是农历，转成阳历再计算
            const c_day = calendar.lunar2solar(year, theyTime[1], theyTime[2]);
            const new_day = c_day.date;
            new_day_arr = new_day.split('-');
          } else {
            new_day_arr = [year, theyTime[1], theyTime[2]];
          }
          
          const [t_year, t_month, t_date] = new_day_arr; //生日新历年月日
          
          const age = year - theyTime[0];
          
          if (t_year / 1 > year) {
            return age - 1;
          } else if (t_year / 1 < year) {
            return age;
          } else {
            if (month > t_month / 1) {
              return age;
            } else if (month < t_month / 1) {
              return age - 1;
            } else {
              if (date >= t_date / 1) {
                return age;
              } else if (month < t_date / 1) {
                return age - 1;
              }
            }
          }
        },
        
        /**
         * 计算时间差
         */
        calcDayByDate(_date, _isOld) {
          const now = new Date();
          
          const year = now.getFullYear();
          const month = now.getMonth() + 1;
          const date = now.getDate();
          const nowTime = now.getTime();
          
          const dateArr = _date.split('-');
          
          let c_day;
          
          if (_isOld) {
            c_day = calendar.lunar2solar(year, dateArr[1], dateArr[2]);
            c_day = c_day.date;
          } else {
            c_day = year + '-' + dateArr[1] + '-' + dateArr[2];
          }
          
          const thisDay = c_day.split('-');
          
          if (month === thisDay[1]*1 && date === thisDay[2]*1) {
            return {
              timeDay: 'happy',
              nextDay: ''
            };
          } else {
            const thatTime = new Date(c_day).getTime();
            let timeDay = Math.ceil((((((thatTime - nowTime) / 1000) / 60) / 60) / 24));
            let nextDay = ''; //下个生日
            
            if (timeDay < 0) { //如果生日过了就计算明年的
              if (_isOld) {
                c_day = calendar.lunar2solar(year + 1, dateArr[1], dateArr[2]);
                c_day = c_day.date;
              } else {
                c_day = (year + 1) + '-' + dateArr[1] + '-' + dateArr[2];
              }
              
              const nextTime = new Date(c_day).getTime();
              nextDay = Math.ceil((((((nextTime - nowTime) / 1000) / 60) / 60) / 24));
            }
            
            return {
              timeDay,
              nextDay
            };
          }
        },
        
        deleteInfo(_index) {
          const self = this;
          Dialog.confirm({
            title: '提示',
            message: '确认删除吗？',
          }).then(() => {
              self.list.splice(_index, 1);
              self.setStorage('dataList', self.list);
            }).catch(() => {
              
            });
        },
        
        /**
         * 编辑功能
         */
        editInfo(_index) {
          this.editIndex = _index;
          this.name = this.showList[_index].name;
          this.birthday = this.showList[_index].birthday;
          this.radio = this.showList[_index].isOld ? '2' : '1';
          this.mode = 'edit';
          this.isShow = true;
        },
        
        /**
         * 编辑保存
         */
        saveEditInfo() {
          if (!this.name || !this.birthday) {
            Toast('请先输入资料');
            return;
          }
          
          const data = {
            name: this.name,
            birthday: this.birthday,
            isOld: this.radio * 1 === 2 //是否是农历
          };
          
          Vue.set(this.list, this.editIndex, this.dataFormat(data));

          this.setStorage('dataList', this.list);
          
          this.isShow = false;
          this.name = '';
          this.birthday = '';
          this.mode = 'add';
          this.editIndex = -1;
          this.radio = '2';
          
          Toast.success('保存成功');
        },
        
        //添加数据转换
        dataFormat(item) {
          const date = item.birthday;
          const dateArr = date.split('-');
          
          let c_day;
          if (item.isOld) { //如果是农历
            c_day = calendar.lunar2solar(dateArr[0], dateArr[1], dateArr[2]);
          } else {
            c_day = calendar.solar2lunar(dateArr[0], dateArr[1], dateArr[2]);
          }
          
          const diffTime = this.calcDayByDate(item.birthday, item.isOld);
                     
          return {
            name: item.name,
            birthday: item.birthday,
            constellation: c_day.astro,
            signZodiac: c_day.Animal,
            age: this.calcAge(item.birthday, item.isOld),
            diffTime: diffTime.timeDay,
            nextDay: diffTime.nextDay,
            isOld: item.isOld
          };
        },
        
        addInfo() {
          if (!this.name || !this.birthday) {
            Toast('请先输入资料');
            return;
          }
          
          const data = {
            name: this.name,
            birthday: this.birthday,
            isOld: this.radio * 1 === 2 //是否是农历
          };
          
          this.list.push(this.dataFormat(data));
          this.setStorage('dataList', this.list);
          this.isShow = false;
          this.name = '';
          this.birthday = '';
          this.radio = '2';
          this.editIndex = -1;
          
          Toast.success('保存成功');
        },
        
        setStorage(_key, _val) {
          localStorage.setItem(_key, JSON.stringify(_val));
        },
        
        getStorage(_key) {
          return JSON.parse(localStorage.getItem(_key));
        },
        
        lunarDataInit() {
          const yearData = [];
          const monthData = [];
          const dateData = [];
          const nowYear = new Date().getFullYear();
          
          for (let i = 1910; i < nowYear + 1; i++) {
            yearData.push(i);
          }
          
          for (let i = 1; i < 13; i++) {
            monthData.push(i < 10 ? '0' + i : i);
          }
          
          for (let i = 1; i < 31; i++) {
            dateData.push(i < 10 ? '0' + i : i);
          }
          
          this.lunarData = [
            {
              values: yearData,
              defaultIndex: 80
            },
            {
              values: monthData,
              defaultIndex: 0
            },
            {
              values: dateData,
              defaultIndex: 0
            }
          ];
        }
      },
      
      mounted() {
        let dataList = this.getStorage('dataList');
        if (dataList) {
          dataList.forEach(item => {
            item = this.dataFormat(item);
          });
          this.list = dataList;
        }
        this.lunarDataInit();
        
        document.addEventListener("plusready", function() {
          plus.navigator.setStatusBarBackground("#8d8def");
        }, false);
        
        this.$nextTick(() => {
          new BetterScroll('.data-list');
        });
      }
  }
</script>

<style lang="less">
  #app {
    .van-nav-bar {
      background-color: #8d8def;
      .van-nav-bar__title {
        color: #fff;
      }
      .van-icon-plus {
        color: #fff !important;
        font-size: 20px;
      }
      .van-dropdown-menu {
        height: 46px;
        line-height: 46px;
        background-color: #8d8def;
        .van-dropdown-menu__title {
          color: #fff;
        }
      }
      .van-nav-bar__left {
        padding-left: 6px;
      }
    }
    
    .info-detail {
      color: #417505;
      font-size: 14px;
      font-weight: normal;
      margin-left: 6px;
    }
    
    .data-list {
      position: absolute;
      top: 46px;
      left: 0;
      right: 0;
      bottom: 0;
      height: auto;
      .tips {
        color: #666;
        line-height: 30px;
        text-align: center;
        padding: 20px 0;
        font-size: 16px;
       }
      .list-content {
        padding: 20px;
        height: 130px;
        box-sizing: border-box;
        display: flex;
        .out-date {
          padding-top: 10px;
          color: #888;
          font-size: 26px;
          font-weight: bold;
          .next-birth {
            margin-top: 6px;
            font-size: 14px;
            font-weight: normal;
            .day {
              font-size: 16px;
            }
          }
        }
        .happy-birthday {
          font-size: 26px;
          font-weight: bold;
          line-height: 60px;
          color: #f4b144;
          padding-top: 12px;
          img {
            position: relative;
            top: -5px;
            width: 50px;
            display: inline-block;
            vertical-align: middle;
          }
        }
        .user-info {
            width: 50%;
            p {
              line-height: 1em;
            }
            h3 {
              font-size: 20px;
              line-height: 1em;
              color: #4286F5;
              margin-bottom: 10px;
              margin-top: 6px;
            }
            .age {
              margin-bottom: 10px;
            }
        }
        .birth-info {
          width: 50%;
          text-align: center;
          .day {
            font-size: 46px;
            font-weight: bold;
            color: #34A853;
            margin-right: 6px;
          }
        }
      }
      .handle-btn {
        height: 100%;
      }
    }
    
    [class*=van-hairline]::after {
      border-color: #ccc;
    }
    
    .van-nav-bar__left [class*=van-hairline]::after {
      border-color: #8d8def !important;
    }
    
    .input-area {
      position: fixed;
      top: 26%;
      left: 5%;
      width: 90%;
      background-color: #fff;
      z-index: 999;
      padding: 10px 20px;
      padding-top: 0;
      box-sizing: border-box;
      border-radius: 6px;
      .title {
        text-align: center;
        font-size: 18px;
        line-height: 60px;
      }
      .input-wrap {
        padding: 0 10px;
        & > div {
          padding: 10px;
          border-bottom: 1px solid #eee;
          font-size: 16px;
          input {
            display: inline-block;
            line-height: 30px;
            font-size: 16px;
            width: 70%;
          }
        }
        .old-new {
          padding-top: 20px;
          border-bottom: 0;
          .van-radio-group {
            justify-content: center;
          }
        }
      }
      .save-btn {
        display: block !important;
        width: 50%;
        margin: 0 auto;
        margin-top: 30px;
        margin-bottom: 20px;
        height: 36px;
        line-height: 36px;
      }
      .icon-wrap {
        position: absolute;
        right: 6px;
        top: 6px;
        .van-icon-close {
          color: #ddd;
          font-size: 30px;
        }
      }
    }
    
    .van-datetime-picker {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      z-index: 99;
    }
  }
</style>
