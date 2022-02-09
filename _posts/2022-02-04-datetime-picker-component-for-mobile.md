---
layout: post
title: '自己动手写一个移动端日期选择器组件'
categories: 前端
tags: uni-app vue 小程序 日期选择器 组件
excerpt: '跟着文章一起来造个简单的轮子吧~'
cover: '/assets/img/posts/2022-02/cover.jpg'
---

## 背景

本文写的组件是基于 `uni-app` 框架下的，但是其实框架不重要，思路都是一样的。

有同学可能会问了，`uni-app` 本身不是就有 `picker`，`mode=time` 的时候就是时间选择器了吗，为什么还要自己写一个？那是因为我们产品大佬说，不要固定在底部弹出选择的，想嵌套在页面筛选条件里，因为考虑到交互blabla的……我想了想，好吧，给时间啥都好说，咱就自己造个轮子呗~


## 效果演示

先来看看效果~

- 完整功能

  ![demo](/assets/img/posts/2022-02/demo.gif)


- 年月日模式

  ![demo](/assets/img/posts/2022-02/date.png)


- 年月日时分秒模式

  ![demo](/assets/img/posts/2022-02/datetime.png)


- 年月模式

  ![demo](/assets/img/posts/2022-02/monthRange.png)


## 思路

开始动手之前先捋一下思路。

移动端的日期筛选器交互方式比较常见的都是多列滚动的，所以我们可以用 `picker-view` 来实现。除了基础交互，组件需要注意的点就是年月日之间的相互关联，比如1月有31天，4月是30天，闰年2月是29天等这些，也就是年月日需要相互关联动态变化。此外还可以添加支持配置最大最小时间范围，支持切换不同的时间模式（比如年月日/年月/年月日时分秒）等。

一个常用的日期选择器组件主要的功能就是以上这些了。

完整代码见：[https://github.com/Dandelion-drq/uniapp-datetime-picker](https://github.com/Dandelion-drq/uniapp-datetime-picker)

欢迎喜欢的朋友给个star哈~


## 实现

### 1. `picker-view` 实现基础交互

先封装一个接受多个数组的多列滚动选择组件，方便后面支持不同日期模式切换。

```html
<template>
  <picker-view class="picker-view" :value="indexArr" @change="onChange">
    <picker-view-column class="picker-view-column" v-for="(col, colIdx) in columns" :key="colIdx">
      <view v-for="(item, idx) in col" :key="idx">{{ item }}</view>
    </picker-view-column>
  </picker-view>
</template>

<script src="./index.js"></script>

<style lang="css" scoped src="./index.css"></style>
```

```css
.picker-view {
  height: 356rpx;
}

.picker-view-column {
  font-size: 14px;
  line-height: 34px;
  text-align: center;
  color: #333;
}
```

```js
export default {
  data() {
    return {};
  },
  props: {
    // 所有列选项数据
    columns: {
      type: Array,
      default: () => []
    },
    // 每一列默认选中值数组，不传默认选中第一项
    selectVals: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    // 每一列选中项的索引，当默认选中值变化的时候这个值也要变化
    indexArr: {
      // 多维数组，深度监听
      cache: false,
      get() {
        // console.log('indexArr', this.selectVals, this.columns);
        if (this.selectVals.length > 0) {
          return this.columns.map((col, cIdx) => {
            return col.findIndex((i) => i == this.selectVals[cIdx]);
          });
        } else {
          return [].fill(0, 0, this.columns.length);
        }
      }
    }
  },
  methods: {
    onChange(e) {
      const { value } = e.detail;
      // console.log('pickerview改变', value, this.columns);

      let ret = this.columns.map((item, index) => {
        let idx = value[index];
        if (idx < 0) {
          idx = 0;
        }
        if (idx > item.length - 1) {
          idx = item.length - 1;
        }
        return item[idx];
      });
      // console.log('选中值', ret);

      this.$emit('onChange', {
        value: ret
      });
    }
  }
};
```

### 2. 年月日动态配置以及支持最大最小日期

年份比较简单，从配置的最小日期年份到最大日期年份生成数组就好。月份要注意当如果选中的年份刚好是最小/最大可选日期的年份时，月份要从最小/最大可选日期开始/结束，其他时候月份都是1~12。日就先列出正常一年每个人的天数配置，然后注意闰年2月是29天，还有同样跟月一样要注意的是当如果选中的年份和月份刚好是最小/最大可选日期的年月时，日要从最小/最大可选日期开始/结束。时分秒同理。

```html
<template>
  <view class="datetime-picker">
    <CustomPickerView :columns="dateConfig" :selectVals="selectVals" @onChange="onChangePickerValue" />
  </view>
</template>

<script src="./index.js"></script>
```

```js
import CustomPickerView from '../customPickerView/index.vue';
import DateUtil from '../dateTimePicker/dateUtil';

export default {
  components: {
    CustomPickerView
  },
  data() {
    return {
      selectYear: new Date().getFullYear(),
      selectMonth: new Date().getMonth() + 1, // 选中的月份，1~12
      selectDay: new Date().getDate(),
      selectHour: new Date().getHours(),
      selectMinute: new Date().getMinutes(),
      selectSecond: new Date().getSeconds()
    };
  },
  props: {
    // 可选的最小日期，默认十年前
    minDate: {
      type: String,
      default: ''
    },
    // 可选的最大日期，默认十年后
    maxDate: {
      type: String,
      default: ''
    }
  },
  computed: {
    minDateObj() {
      let minDate = this.minDate;
      if (minDate) {
        if (this.mode == 2 && minDate.replace(/\-/g, '/').split('/').length == 2) {
          // 日期模式为年月时有可能传进来的minDate是2022-02这样的格式，在ios下new Date会报错，加上日期部分做兼容
          minDate += '-01';
        }
        return new Date(DateUtil.handleDateStr(minDate));
      } else {
        // 没有传最小日期，默认十年前
        minDate = new Date();
        minDate.setFullYear(minDate.getFullYear() - 10);
        return minDate;
      }
    },
    maxDateObj() {
      let maxDate = this.maxDate;
      if (maxDate) {
        if (this.mode == 2 && maxDate.replace(/\-/g, '/').split('/').length == 2) {
          // 日期模式为年月时有可能传进来的maxDate是2022-02这样的格式，在ios下new Date会报错，加上日期部分做兼容
          maxDate += '-01';
        }
        return new Date(DateUtil.handleDateStr(maxDate));
      } else {
        // 没有传最小日期，默认十年后
        maxDate = new Date();
        maxDate.setFullYear(maxDate.getFullYear() + 10);
        return maxDate;
      }
    },    
    years() {
      let years = [];
      let minYear = this.minDateObj.getFullYear();
      let maxYear = this.maxDateObj.getFullYear();
      for (let i = minYear; i <= maxYear; i++) {
        years.push(i);
      }

      return years;
    },
    months() {
      let months = [];
      let minMonth = 1;
      let maxMonth = 12;

      // 如果选中的年份刚好是最小可选日期的年份，那月份就要从最小日期的月份开始
      if (this.selectYear == this.minDateObj.getFullYear()) {
        minMonth = this.minDateObj.getMonth() + 1;
      }
      // 如果选中的年份刚好是最大可选日期的年份，那月份就要在最大日期的月份结束
      if (this.selectYear == this.maxDateObj.getFullYear()) {
        maxMonth = this.maxDateObj.getMonth() + 1;
      }

      for (let i = minMonth; i <= maxMonth; i++) {
        months.push(i);
      }

      return months;
    },
    days() {
      // 一年中12个月每个月的天数
      let monthDaysConfig = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
      // 闰年2月有29天
      if (this.selectMonth == 2 && this.selectYear % 4 == 0) {
        monthDaysConfig[1] = 29;
      }

      let minDay = 1;
      let maxDay = monthDaysConfig[this.selectMonth - 1];

      if (this.selectYear == this.minDateObj.getFullYear() && this.selectMonth == this.minDateObj.getMonth() + 1) {
        minDay = this.minDateObj.getDate();
      }
      if (this.selectYear == this.maxDateObj.getFullYear() && this.selectMonth == this.maxDateObj.getMonth() + 1) {
        maxDay = this.maxDateObj.getDate();
      }

      let days = [];
      for (let i = minDay; i <= maxDay; i++) {
        days.push(i);
      }

      return days;
    },
    hours() {
      let hours = [];
      let minHour = 0;
      let maxHour = 23;

      if (
        this.selectYear == this.minDateObj.getFullYear() &&
        this.selectMonth == this.minDateObj.getMonth() + 1 &&
        this.selectDay == this.minDateObj.getDate()
      ) {
        minHour = this.minDateObj.getHours();
      }
      if (
        this.selectYear == this.maxDateObj.getFullYear() &&
        this.selectMonth == this.maxDateObj.getMonth() + 1 &&
        this.selectDay == this.maxDateObj.getDate()
      ) {
        maxHour = this.maxDateObj.getHours();
      }

      for (let i = minHour; i <= maxHour; i++) {
        hours.push(i);
      }

      return hours;
    },
    minutes() {
      let mins = [];
      let minMin = 0;
      let maxMin = 59;

      if (
        this.selectYear == this.minDateObj.getFullYear() &&
        this.selectMonth == this.minDateObj.getMonth() + 1 &&
        this.selectDay == this.minDateObj.getDate() &&
        this.selectHour == this.minDateObj.getHours()
      ) {
        minMin = this.minDateObj.getMinutes();
      }
      if (
        this.selectYear == this.maxDateObj.getFullYear() &&
        this.selectMonth == this.maxDateObj.getMonth() + 1 &&
        this.selectDay == this.maxDateObj.getDate() &&
        this.selectHour == this.maxDateObj.getHours()
      ) {
        maxMin = this.maxDateObj.getMinutes();
      }

      for (let i = minMin; i <= maxMin; i++) {
        mins.push(i);
      }

      return mins;
    },
    seconds() {
      let seconds = [];
      let minSecond = 0;
      let maxSecond = 59;

      if (
        this.selectYear == this.minDateObj.getFullYear() &&
        this.selectMonth == this.minDateObj.getMonth() + 1 &&
        this.selectDay == this.minDateObj.getDate() &&
        this.selectHour == this.minDateObj.getHours() &&
        this.selectMinute == this.minDateObj.getMinutes()
      ) {
        minSecond = this.minDateObj.getSeconds();
      }
      if (
        this.selectYear == this.maxDateObj.getFullYear() &&
        this.selectMonth == this.maxDateObj.getMonth() + 1 &&
        this.selectDay == this.maxDateObj.getDate() &&
        this.selectHour == this.maxDateObj.getHours() &&
        this.selectMinute == this.maxDateObj.getMinutes()
      ) {
        maxSecond = this.maxDateObj.getSeconds();
      }

      for (let i = minSecond; i <= maxSecond; i++) {
        seconds.push(i);
      }

      return seconds;
    }
  }
}
```

```js
// DateUtil.js

/**
 * 日期时间格式化
 * @param {Date} date 要格式化的日期对象
 * @param {String} fmt 格式化字符串，eg：YYYY-MM-DD HH:mm:ss
 * @returns 格式化后的日期字符串
 */
function formatDate(date, fmt) {
  if (typeof date == 'string') {
    date = new Date(handleDateStr(date));
  }

  var o = {
    'M+': date.getMonth() + 1, // 月份
    'd+': date.getDate(), // 日
    'D+': date.getDate(), // 日
    'H+': date.getHours(), // 小时
    'h+': date.getHours(), // 小时
    'm+': date.getMinutes(), // 分
    's+': date.getSeconds(), // 秒
    'q+': Math.floor((date.getMonth() + 3) / 3), // 季度
    S: date.getMilliseconds() // 毫秒
  };

  if (/([y|Y]+)/.test(fmt)) {
    fmt = fmt.replace(RegExp.$1, (date.getFullYear() + '').slice(4 - RegExp.$1.length));
  }
  for (var k in o) {
    if (new RegExp('(' + k + ')').test(fmt)) {
      fmt = fmt.replace(RegExp.$1, RegExp.$1.length == 1 ? o[k] : ('00' + o[k]).slice(('' + o[k]).length));
    }
  }

  return fmt;
}

/**
 * 处理时间字符串，兼容ios下new Date()返回NaN问题
 * @param {*} dateStr 日期字符串
 * @returns
 */
function handleDateStr(dateStr) {
  return dateStr.replace(/\-/g, '/');
}

/**
 * 判断日期1是否在日期2之前，即日期1小于日期2
 * @param {Date} date1
 * @param {Date} date2
 * @returns
 */
function isBefore(date1, date2) {
  if (typeof date1 == 'string') {
    date1 = new Date(handleDateStr(date1));
  }
  if (typeof date2 == 'string') {
    date2 = new Date(handleDateStr(date2));
  }
  return date1.getTime() < date2.getTime();
}

/**
 * 判断日期1是否在日期2之后，即日期1大于日期2
 * @param {Date} date1
 * @param {Date} date2
 * @returns
 */
function isAfter(date1, date2) {
  if (typeof date1 == 'string') {
    date1 = new Date(handleDateStr(date1));
  }
  if (typeof date2 == 'string') {
    date2 = new Date(handleDateStr(date2));
  }
  return date1.getTime() > date2.getTime();
}

export default {
  formatDate,
  handleDateStr,
  isBefore,
  isAfter
};
```

### 3. 支持不同日期模式

支持多种不同的日期模式，包括年月日（默认）、年月、年份、年月日时分秒。主要的处理逻辑是要根据 `mode` 的变化，来动态生成传给 `pickerView` 组件的数组，以及其默认选中值，还有注意 `pickerView` 组件 `onChange` 事件的处理也需要考虑不同日期模式的情况。

```html
<template>
  <view class="datetime-picker">
    <PickerView :columns="dateConfig" :selectVals="selectVals" @onChange="onChangePickerValue" />
  </view>
</template>

<script src="./index.js"></script>

<style scoped></style>
```

```js
{
  props: {
    // 日期模式，1：年月日，2：年月，3：年份，4：年月日时分秒
    mode: {
      type: Number,
      default: 1
    },
    // 默认选中日期（注意要跟日期模式对应）
    defaultDate: {
      type: String,
      default: ''
    }
  }
  computed: {
    // 传给pickerView组件的数组，根据mode来生成不同的数据
    dateConfig() {
      if (this.mode == 2) {
        // 年月模式
        let years = this.years.map((y) => y + '年');
        let months = this.months.map((m) => m + '月');
        return [years, months];
      } else if (this.mode == 3) {
        // 只有年份模式
        let years = this.years.map((y) => y + '年');
        return [years];
      } else if (this.mode == 4) {
        // 年月日时分秒模式
        let years = this.years.map((y) => y + '年');
        let months = this.months.map((m) => m + '月');
        let days = this.days.map((d) => d + '日');
        let hours = this.hours.map((h) => h + '时');
        let minutes = this.minutes.map((m) => m + '分');
        let seconds = this.seconds.map((s) => s + '秒');
        return [years, months, days, hours, minutes, seconds];
      } else {
        // 默认，年月日模式
        let years = this.years.map((y) => y + '年');
        let months = this.months.map((m) => m + '月');
        let days = this.days.map((d) => d + '日');
        return [years, months, days];
      }
    },
    // pickerView默认值，根据mode的切换来变换值
    selectVals() {
      if (this.mode == 2) {
        return [this.selectYear + '年', this.selectMonth + '月'];
      } else if (this.mode == 3) {
        return [this.selectYear + '年'];
      } else if (this.mode == 4) {
        return [
          this.selectYear + '年',
          this.selectMonth + '月',
          this.selectDay + '日',
          this.selectHour + '时',
          this.selectMinute + '分',
          this.selectSecond + '秒'
        ];
      } else {
        return [this.selectYear + '年', this.selectMonth + '月', this.selectDay + '日'];
      }
    }
  },
  methods: {
        onChangePickerValue(e) {
      const { value } = e;
      // console.log('onChangePickerValue', value);

      if (this.mode == 2 && value[0] && value[1]) {
        // 年月模式
        this.selectYear = Number(value[0].replace('年', ''));
        this.selectMonth = Number(value[1].replace('月', ''));
      } else if (this.mode == 3 && value[0]) {
        // 只有年份模式
        this.selectYear = Number(value[0].replace('年', ''));
      } else if (this.mode == 4 && value[0] && value[1] && value[2] != '' && value[3] && value[4] && value[5]) {
        // 年月日时分秒模式
        this.selectYear = Number(value[0].replace('年', ''));
        this.selectMonth = Number(value[1].replace('月', ''));
        this.selectDay = Number(value[2].replace('日', ''));
        this.selectHour = Number(value[3].replace('时', ''));
        this.selectMinute = Number(value[4].replace('分', ''));
        this.selectSecond = Number(value[5].replace('秒', ''));
      } else if (value[0] && value[1] && value[2]) {
        // 默认，年月日模式
        this.selectYear = Number(value[0].replace('年', ''));
        this.selectMonth = Number(value[1].replace('月', ''));
        this.selectDay = Number(value[2].replace('日', ''));
      } else {
        // 其他情况可能是pickerView返回的数据有问题，不处理
        console.log('onChangePickerValue其他情况');
        return;
      }

      let formatTmpl = 'YYYY-MM-DD';
      if (this.mode == 2) {
        formatTmpl = 'YYYY-MM';
      } else if (this.mode == 3) {
        formatTmpl = 'YYYY';
      } else if (this.mode == 4) {
        formatTmpl = 'YYYY-MM-DD HH:mm:ss';
      }

      this.$emit(
        'onChange',
        DateUtil.formatDate(
          new Date(`${this.selectYear}/${this.selectMonth}/${this.selectDay} ${this.selectHour}:${this.selectMinute}:${this.selectSecond}`),
          formatTmpl
        )
      );
    }
  }
}
```

完成了以上3点，日期选择器组件就写好了，完整代码以及使用demo见：[https://github.com/Dandelion-drq/uniapp-datetime-picker](https://github.com/Dandelion-drq/uniapp-datetime-picker)

欢迎喜欢的朋友给个star~
