---
layout: post
title: '如何正确地 reset Vuex module state'
categories: 前端
tags: Vue Vuex
cover: '/assets/img/posts/2020-03/cover.jpg'
---


这是项目之前遇到的一个bug，最终发现是由于 `reset Vuex state` 不正确，污染了 `initState` 导致的，隐藏得还挺深的，在这里记录一下。

（PS：想直接看代码实现的同学可以从第三节 **正确地 `reset module state` 的姿势** 开始看）


# 背景

项目是用 `Vue + Nuxt` 写的一个H5网页。

下图是分类页的界面，左边的导航给出的分类项可以叠加选择。 

在选择了任意分类项后点击 `重置` 可以把所有选中的项或输入的值恢复到未设置状态。

![](/assets/img/posts/2020-03/1.jpg)


其中 `价格范围` 是输入最小值和最大值。

![](/assets/img/posts/2020-03/2.jpg)


## 一个bug

某天产品经理跟我反馈了一个bug……  
简单来说，就是如果用户设置过价格范围，然后点了重置，下次再次设置价格然后点击重置的时候，会无法重置价格……

为了更好地说明问题，我写了个简单的demo页面，给大家演示一下。

![](/assets/img/posts/2020-03/demo.gif)


# 错误的示范

下面我们来看看这个错误实现的代码是怎样的。

基于上面的界面，而且 `vuex` 也分了模块，所以这个分类页的 `store` 是这样的：

`category.js`
```js
const initState = {
    selectedIds: {
        gender: null,
        category: [],
        discount: [],
        priceRange: {
            start: null,
            end: null
        },
        source: [],
    }
}

export const state = () => {
    return Object.assign({}, initState)
}

export const mutations = {
    SET_FILTER_IDS_STATE(state, data) {
        Object.keys(data).forEach(key => {
            console.log('key', key)
            if (key === 'priceRange') {
                state.selectedIds.priceRange.start = data[key].start
                state.selectedIds.priceRange.end = data[key].end
            } else {
                state.selectedIds[key] = data[key]
            }
        })
    },

    RESET_ALL_FILTERS(state) {
        Object.keys(initState).forEach(key => {
            Object.assign(state[key], initState[key])
        })
    },
}

export const actions = {
    async setFilters({ commit }, { selectedIds }) {
        commit('SET_FILTER_IDS_STATE', selectedIds)
    },

    async resetAllFilters({ commit }) {
        commit('RESET_ALL_FILTERS')
    },
}

export const getters = {
    selectedIds(state) {
        return state.selectedIds
    },
}
```

问题就出在上面的 `RESET_ALL_FILTERS` 方法。这是网上找到的比较多人建议的 `reset state` 的方法。  

其实这种实现方式在大部分情况下还是work的，但是！！因为我们这个分类页的 `state` 是个层级比较深的对象，而里面 `Object.assign(state[key], initState[key])` 这一句，就是关键！  
因为 `Object.assign` 方法，其实是浅拷贝，所以当重置 `priceRange` 的时候，由于 `priceRange` 是个对象，那生成的 `target`【`Object.assign(target, ...sources)`】其实只是把引用指向了 `initState.priceRange` 的引用，也就是说，经过第一次重置之后，`initState` 的 `priceRange` 和当前的 `category state` 的 `priceRange` 是指向了同一块内存的。  
所以，当后面再次设置性别和价格然后点重置的时候，性别可以正常重置，但是价格已经无法重置了，因为 `initState` 已经被污染了！！


# 正确地 `reset module state` 的姿势

## 方法一

既然经过上面的解释，我们明白了是浅拷贝的锅，那很自然地就会想到用深拷贝的方式来解决这个问题。

下面直接上代码。

`category.js`
```js
import cloneDeep from 'lodash.clonedeep'

export const state = () => {
    return cloneDeep(initState)
}

export const mutations = {
    RESET_ALL_FILTERS(state) {
        Object.assign(state, cloneDeep(initState))
    },
}
```
PS：这里就只放跟上文 *错误示范* 里对比有修改的部分啦


## 方法二

在整理这篇文章的时候我又google了一下 `vuex reset store`，找到了个更优雅的实现方式。

如果我们把 `initState` 写成一个函数，比如 `getDefaultState`，这个函数就只是返回 `initState` 的，然后每次重置的时候先调用这个 `getDefaultState` 再赋值，那就能保证 `initState` 一定是初始值啦，也就同样可以避免 `initState` 被污染的问题了。

还是上代码。

`category.js`
```js
const getDefaultState = () => {
    return {
        selectedIds: {
            gender: null,
            category: [],
            discount: [],
            priceRange: {
                start: null,
                end: null
            },
            source: [],
        }
    }
}

export const state = getDefaultState

export const mutations = {
    RESET_ALL_FILTERS(state) {
        const initState = getDefaultState()
        Object.keys(initState).forEach(key => {
            state[key] = initState[key]
        })
    },
}
```
PS：这里只放跟上文 *错误示范* 里对比有修改的部分


# 总结

上面写了两种 `reset state` 的实现方式，我个人觉得第二种更优雅。

当然，其实还有一个问题，就是这个 `category state` 设计得过于复杂了，我们一般做项目的时候其实不建议嵌套太深，容易出问题。所以在一开始设计数据 `model` 的时候，还是要多加考虑呀。


# 参考

- [Reset Vuex Module State Like a Pro](https://medium.com/@Taha_Shashtari/reset-vuex-module-state-like-a-pro-1acb7f8d9e21)
- [Clear all stores or set them all to its initial state](https://github.com/vuejs/vuex/issues/1118#issuecomment-356286218)


# 附录

最后附上 `demo` 页面的代码，方便有需要的同学自取演示。

`demo.vue`
```html
<template>
  <div class="page">
    <section>
      <form>
        <div class="input-group">
          <label class="input-label">性别：</label>
          <div class="radio-group">
            <input id="man" type="radio" value="man" name="gender" v-model="gender" />
            <label for="man">男士</label>
          </div>
          <div class="radio-group">
            <input id="woman" type="radio" value="woman" name="gender" v-model="gender" />
            <label for="woman">女士</label>
          </div>
        </div>

        <div class="input-group">
          <label class="input-label">价格范围：</label>
          ￥<input class="input" type="number" v-model="minPrice" />至
          ￥<input class="input" type="number" v-model="maxPrice" />
        </div>

        <div class="btn-group">
          <button class="btn btn-reset" @click.prevent="onReset">重置</button>
          <button class="btn btn-submit" @click.prevent="onSubmit">提交</button>
        </div>
      </form>
    </section>

    <hr />

    <section class="vuex-display">
      <h3 class="vuex-display-title">Vuex state</h3>
      <div class="state-item" v-for="key in Object.keys(selectedIds)" :key="key">
        <span class="state-key">{{key}}：</span>
        <p
          v-if="key === 'priceRange'"
        >{% raw %}{{selectedIds[key].start && selectedIds[key].end ? selectedIds[key].start + '-' + selectedIds[key].end : '未选择'}}{% endraw %}</p>
        <p v-else-if="key === 'gender'">{% raw %}{{selectedIds[key] || "未选择"}}{% endraw %}</p>
        <p v-else>{% raw %}{{selectedIds[key].join(',') || '未选择'}}{% endraw %}</p>
      </div>
    </section>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  name: "test",
  layout: "single-page",

  data() {
    return {
      minPrice: NaN,
      maxPrice: NaN,
      gender: null,
      category: [],
      discount: [],
      source: []
    };
  },

  computed: {
    ...mapGetters({
      selectedIds: "categoryFilter/selectedIds"
    })
  },

  async mounted() {
    this.initPriceRange();
  },

  methods: {
    ...mapActions({
      setFilters: "categoryFilter/setFilters",
      resetFilters: "categoryFilter/resetAllFilters"
    }),

    async initFilters() {
      try {
        const res = await this.$axios.$get(
          "api" + this.$api.filter.categoryList
        );
        if (res.status === 0) {
          const { data } = res;
          this.$store.commit("category/FETCH_FILTERS", {
            data
          });
        }
      } catch (e) {
        console.error(e);
      }
    },

    initPriceRange() {
      this.minPrice = this.selectedIds.priceRange.start || NaN;
      this.maxPrice = this.selectedIds.priceRange.end || NaN;
    },

    onSubmit() {
      let selectedIds = {
        gender: this.gender,
        category: this.category,
        discount: this.discount,
        source: this.source,
        priceRange: {
          start: Number(this.minPrice),
          end: Number(this.maxPrice)
        }
      };
      this.setFilters({
        selectedIds
      });
    },

    onReset() {
      this.minPrice = NaN;
      this.maxPrice = NaN;
      this.gender = null;
      this.category = [];
      this.discount = [];
      this.source = [];

      this.resetFilters();
    }
  }
};
</script>

<style lang="scss" scoped>
.page {
  padding: 20px;
}

.input-group {
  color: #333;
  display: flex;
  justify-content: flex-start;
  padding: 10px 0;
  align-items: center;
}

.input {
  width: 80px;
  border: solid 0.5px #aaa;
  border-radius: 5px;
  padding: 0 10px;
  line-height: 30px;
  margin-right: 10px;
}

.input-label {
  margin-right: 5px;
  font-weight: bold;
}

input[type="radio"] {
  margin-right: 5px;
}

.radio-group {
  display: flex;
  align-items: center;
  margin-right: 20px;
}

.btn-group {
  margin-top: 20px;
  text-align: right;
  display: flex;
  justify-content: space-between;
}

.btn {
  width: 60px;
  height: 35px;
  border-radius: 5px;
  width: 46%;
}

.btn-submit {
  background-color: #333;
  border: none;
  color: #fff;
}

.btn-reset {
  border: #333 solid 0.5px;
  background: #fff;
  color: #333;
}

hr {
  border: solid 0.5px #aaa;
  margin: 10px 0;
}

section {
  padding-bottom: 20px;
}

.vuex-display-title {
  font-size: 20px;
  padding: 10px 0;
}

.vuex-display {
  color: #333;
}

.state-item {
  line-height: 1.5;
  padding-bottom: 10px;
}

.state-key {
  font-weight: bold;
}
</style>
```