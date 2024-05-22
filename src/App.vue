<template>
  <div class="app">
    <div class="my-calendar">
      <!-- 头部日期 -->
      <div class="calendar-header">
        <div class="month-change">
          <img src="./assets/darrow.png" alt="" @click="changeYear(-1)">
          <img src="./assets/arrow.png" mode="" @click="changeMonth(-1)" class="change-arrow">
          <text class="date-text">{{getYearMonth()}}</text>
          <img src="./assets/arrow.png" mode="" @click="changeMonth(1)" class="change-arrow arrow-rotate">
          <img src="./assets/darrow.png" alt="" @click="changeYear(1)" class="arrow-rotate d-arrow-right">
        </div>
      </div>
      <div class="calendar-weekdays">
        <text class="weekday" v-for="(item, index) in weekdays" :key="index">{{item}}</text>
      </div>
      <!-- 日历 -->
      <div class="calendar-month" @mouseover="onMouseOver">
        <div class="day" v-for="item in dayLists" :key="item.id">
          <template v-if="item.id">
            <div
              :class="[
                'day-num',
                { 'range-day': isRange(item)},
                { 'range-start-day': rangeDate[0] == item.date },
                { 'range-end-day': rangeDate[1] == item.date},
                { 'range-one-day': rangeDate[0] && !rangeDate[1]}
              ]"
              @click="selectRange(item)"
            >
              <div :class="[
                { 'cur-day': yearMonthCopy == yearMonth && curDay == item.id },
                { 'range-select-day': rangeStart == item.date || rangeEnd == item.date || rangeEndMove == item.date }
              ]"
              style="width: 90%;height:90%;display:flex;align-items:center;justify-content:center">
                {{ item.day }}
              </div>
            </div>
          </template>
        </div>
      </div>
      <!-- 底部开始时间 结束时间 -->
      <div class="time-picker">
        <div class="start-time">
          <span>开始时间</span>
          <div>
            <span class="time-num">23</span>
            <span> : </span>
            <span class="time-num">36</span>
          </div>
        </div>
        <div class="start-time">
          <span>结束时间</span>
          <div>
            <span class="time-num">23</span>
            <span> : </span>
            <span class="time-num">36</span>
          </div>
        </div>
      </div>
    </div>
  </div> 
</template>
<script setup>
import dayjs from 'dayjs'
import { ref, onMounted } from 'vue'
    let weekdays = ref(['S', 'M', 'T', 'W', 'T', 'F', 'S'])
    let yearMonth = ref(''); //获取当前日期
		let yearMonthCopy = ref(''); //获取当前日期备份
		let dayLists = ref([]); //日历day列表
		let curDay= ref('');
    let rangeStart = ref(); //日期范围开始时间
    let rangeEnd = ref(); //日期范围结束时间
    let rangeEndMove = ref(); //鼠标移动时的日期范围结束时间
    let rangeDate = ref([]) //日期范围
    //获取 年-月-01
		const getDayFirst = () => {
			return `${yearMonth.value}-01`
		}
		//获取xxxx年xx月xx日
		const getYearMonth = () => {
      let arr = yearMonth.value.split('-')
      return `${arr[0]}年 ${arr[1]}月`
		}
		const getMonth = () => {
			//当前月份的天数
			let days = dayjs(yearMonth.value).daysInMonth()

			//获取1号为星期几
			let week = new Date(getDayFirst()).getDay()
			dayLists.value = [...new Array(week)].map(() => ({}))
			//设置当前月份天数
			for (let i = 0; i < days; i++) {
				let str = i + 1
				dayLists.value.push({
					id: str.toString(),
					day: str.toString(),
					date: `${yearMonth.value}-${str.toString()}`
				})
			}
		}
		//月份切换
		const changeMonth = (change) => {
			yearMonth.value = dayjs(yearMonth.value)
				.add(change, 'month')
				.format('YYYY-MM')
			//获取当前月天数
		  getMonth()
		}
    //年切换
    const changeYear = (change) => {
      let arr = yearMonth.value.split('-')
      yearMonth.value = `${arr[0]*1+change}-${arr[1]}`
			//获取当前月天数
		  getMonth()
    }
    //日期范围选择
    const selectRange = (item) =>{
      if(!rangeStart.value || (rangeStart.value && rangeEnd.value)) {
        rangeStart.value = `${yearMonth.value}-${item.id}`
        rangeEnd.value = ''
        rangeEndMove.value = ''
      }
      if(rangeStart.value && !rangeEnd.value) {
        rangeEnd.value = rangeEndMove.value
      }
      compareDate(rangeEnd.value)
    }
    //鼠标over时触发
    const onMouseOver = (e) => {
      if(rangeStart.value && !rangeEnd.value) {
        rangeEndMove.value = `${yearMonth.value}-${e.target.textContent}` 
        compareDate(rangeEndMove.value)
      }
      
    }
    //比较两个时间大小
    const compareDate = (end) => {
      const t1 = dayjs(rangeStart.value);
      const t2 = dayjs(end);
      if(t1.isAfter(t2)) {
        rangeDate.value = [end, rangeStart.value]
      } else {
        rangeDate.value = [rangeStart.value, end]
      }
    }
    //判断日期是否在范围内
    const isRange = (item) => {
      const t1 = dayjs(rangeDate.value[0]);
      const t2 = dayjs(rangeDate.value[1]);
      const t3 = dayjs(item.date)
      if(!t1 || !t2) {
        return
      }
      if((t3.isAfter(t1) && t3.isBefore(t2)) || rangeDate.value[0] == item.date || rangeDate.value[1] == item.date) {
        return true
      } else {
        return false
      }
    }

    onMounted(() => {
      //获取 年-月
      yearMonth.value = dayjs(new Date()).format('YYYY-MM')
      yearMonthCopy.value = yearMonth.value
      //获取日
      curDay.value = dayjs(new Date()).format('DD')
      getMonth()
    })
</script>
<style lang="scss">
body{
  margin: 0;
  padding: 0;
}
.app{
  width: 100vw;
  height: 100vh;
  background: #fefefe;
}
.my-calendar {
  width: 300px;
  background-color: #fff;
  border-radius: 3px;
  box-shadow: 0px 20px 30px 0px rgba(0,0,0,0.05);
  padding: 30px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
	.calendar-header {
		margin: 10px 10px 30px;
		.month-change {
			display: flex;
			justify-content: space-between;
			align-items: center;
			.date-text {
				font-size: 16px;
				font-weight: 500;
			}
			img{
				width: 15px;
        cursor: pointer;
			}
      .change-arrow{
        width: 10px;
      }
      .arrow-rotate{
        transform: rotate(180deg);
      }
      .d-arrow-right{
        transform: rotate(180deg) translateY(-0.5px);
      }
      &>*{
        margin-right: 15px;
      }
      :last-child{
        margin-right: 0;
      }
		}
	}
	.calendar-weekdays {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
		.weekday {
			place-self: center;
			font-size: 14px;
			font-weight: 400;
      color: #c1c1c1;
		}
	}
	.calendar-month {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
		.day {
			place-self: center;
      width: 100%;
      box-sizing: border-box;
			.day-num {
				font-size: 12px;
				font-weight: 400;
        height: calc(300px/7);
        width: 100%;
        text-align: center;
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
			}
			.cur-day {
        box-sizing: border-box;
        color: #5c358a;
        border-radius: 50%;
				border: 1px solid #5c358a;
			}
      .range-start-day{
        border-top-left-radius: 50%;
        border-bottom-left-radius: 50%;
      }
      .range-end-day{
        border-top-right-radius: 50%;
        border-bottom-right-radius: 50%;
      }
      .range-select-day{
        background-color: #5c358a;
        color: #fff;
        border-radius: 50%;
      }
      .range-day{
        background-color: #f5f1f4;
      }
      .range-one-day{
        border-radius: 50%;
      }
		}
	}
  .time-picker{
    border-top: 1px solid #e0e0e0;
    margin-top: 10px;
    .start-time{
      display: flex;
      justify-content: space-between;
      margin-top: 20px;
      .time-num{
        background-color: #f3f3f3;
        border-radius: 3px;
        padding: 1px 3px;
      }
    }
  }
}
</style>