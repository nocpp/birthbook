<template>
  <div id="app">
    <van-nav-bar title="生日记录本" @click-right="onClickRight">
      <van-dropdown-menu slot='left'>
        <van-dropdown-item v-model="type" :options="option1" />
      </van-dropdown-menu>
      <van-icon name="plus" slot='right' />
    </van-nav-bar>
    
    <div class="data-list">
      <div>
        <van-swipe-cell v-for="(item, index) in showList" :key="item.name" class="van-hairline--bottom">
          <div class="list-content">
            <div class="user-info">
              <h3>{{item.name}} <span class="info-detail">属{{item.signZodiac}} {{item.constellation}}</span></h3>
              <p class="age">年龄: {{item.age}} 岁</p>
              <p>生日: {{item.birthday}}</p>
            </div>
            <div class="birth-info">
              <div v-if="item.diffTime > 0"><span class="day">{{item.diffTime}}</span>天后生日</div>
              <div class="out-date" v-else>生日已过</div>
            </div>
          </div>
          <template #right>
            <van-button square type="danger" text="删除" class="handle-btn" @click="deleteInfo(index)"/>
          </template>
        </van-swipe-cell>
        <div class="tips" v-show="list.length === 0">暂时没有数据，快去添加吧~</div>
      </div>
    </div>
    
    <div class="input-area" v-show="isShow">
      <div class="title van-hairline--bottom">添加生日</div>
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
            <input type="text" placeholder="请选择生日" v-model="birthday" @click="dateShow = true" readonly>
          </label>
        </div>
      </div>
      <van-button type="info" class="save-btn" @click="addInfo">保存</van-button>
      <div class="icon-wrap" @click="isShow = false">
        <van-icon name="close" />
      </div>
    </div>
    <div class="my-mask" v-show="isShow"></div>
    <van-popup position="bottom" v-model="dateShow" :style="{ height: '264px' }">
     <van-datetime-picker
        v-model="currentDate"
        type="date"
        :min-date="minDate"
        :max-date="maxDate"
        @confirm="choseTime"
        @cancel="cancelTime"
      />
    </van-popup>
  </div>
</template>

<script>
  import Vue from 'vue';
  import {calendar} from '@/assets/js/calendar.js'
  import BetterScroll from 'better-scroll';
  import { NavBar, Icon, DropdownMenu, DropdownItem, SwipeCell, Button, Toast, DatetimePicker, Popup} from 'vant';
  
  Vue.use(NavBar);
  Vue.use(Icon);
  Vue.use(DropdownMenu);
  Vue.use(DropdownItem);
  Vue.use(SwipeCell);
  Vue.use(Button);
  Vue.use(Toast);
  Vue.use(DatetimePicker);
  Vue.use(Popup);
  
  export default {
      data() {
        return {
          option1: [
            { text: '全部', value: 1 },
            { text: '未过', value: 2 },
            { text: '已过', value: 3 },
          ],
          isShow: false,
          list: [],
          name: '',
          birthday: '',
          type: 1,
          minDate: new Date(1910, 0, 1),
          maxDate: new Date(),
          currentDate: new Date(1995, 0, 1),
          dateShow: false
        };
      },
      
      computed: {
        showList() {
          const theList = this.list.map(item => {
            const date = item.birthday;
            const dateArr = date.split('-');
            const c_day = calendar.lunar2solar(dateArr[0], dateArr[1], dateArr[2]);
           
            return {
              name: item.name,
              birthday: item.birthday,
              constellation: c_day.astro,
              signZodiac: c_day.Animal,
              age: this.calcAge(item.birthday),
              diffTime: this.calcDayByDate(item.birthday)
            }
          });
          
          if (this.type === 1) {
            return theList;
          } else if (this.type === 2) {
            return theList.filter(item => item.diffTime > 0);
          } else {
            return theList.filter(item => item.diffTime <= 0);
          }
        }
      },
      
      methods: {
        onClickRight() {
            this.name = '';
            this.birthday = '';
            this.isShow = true;
        },
        
        choseTime(_value) {
          const theTime = new Date(_value);
          const ret = theTime.getFullYear() + '-' + (theTime.getMonth() + 1) + '-' + theTime.getDate();
          this.birthday = ret;
          this.dateShow = false;
        },
        
        cancelTime() {
          this.dateShow = false;
        },
        
        /**
         * 计算年龄
         */
        calcAge(_day) {
          const now = new Date();
          
          const year = now.getFullYear();
          const month = now.getMonth() + 1;
          const date = now.getDate();

          const theyTime = _day.split('-');
          
          const c_day = calendar.lunar2solar(year, theyTime[1], theyTime[2]);
          const new_day = c_day.date;
          const new_day_arr = new_day.split('-');
          
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
        calcDayByDate(_date) {
          const now = new Date();
          
          const year = now.getFullYear();
          const month = now.getMonth() + 1;
          const date = now.getDate();
          const nowTime = now.getTime();
          
          const dateArr = _date.split('-');
          
          const c_day = calendar.lunar2solar(year, dateArr[1], dateArr[2]);
          
          const thatTime = new Date(c_day.date).getTime();
          
          let timeDay = Math.ceil((((((thatTime - nowTime) / 1000) / 60) / 60) / 24));
          
          return timeDay;
        },
        
        deleteInfo(_index) {
          this.list.splice(_index, 1);
          this.setStorage('dataLst', this.list);
        },
        
        addInfo() {
          if (!this.name || !this.birthday) {
            Toast('请先输入资料');
            return;
          }
          
          const data = {
            name: this.name,
            birthday: this.birthday
          };
          
          this.list.push(data);
          this.setStorage('dataLst', this.list);
          this.isShow = false;
          this.name = '';
          this.birthday = '';
          
          Toast.success('保存成功');
        },
        
        setStorage(_key, _val) {
          localStorage.setItem(_key, JSON.stringify(_val));
        },
        
        getStorage(_key) {
          return JSON.parse(localStorage.getItem(_key));
        }
      },
      
      mounted() {
        const dataLst = this.getStorage('dataLst');
        if (dataLst) {
          this.list = dataLst;
        }
        
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
          color: #888;
          font-size: 26px;
          font-weight: bold;
          line-height: 60px;
        }
        .user-info {
            width: 50%;
            p {
              line-height: 1em;
            }
            h3 {
              font-size: 22px;
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
          .day {
            font-size: 50px;
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
