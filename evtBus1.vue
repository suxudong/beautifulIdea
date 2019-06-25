http://www.js-code.com/vue-js/vue-js_26414.html

vue采用EventBus实现跨组件通信及注意事项
 Vue.js  2019-06-12  21  0  0

EventBus
EventBus是一种发布/订阅事件设计模式的实践。
在vue中适用于跨组件简单通信，不适应用于复杂场景多组件高频率通信，类似购物车等场景状态管理建议采用vuex。

挂载EventBus到vue.prototype
添加bus.js文件
//src/service/bus.js

export default (Vue) => {
  const eventHub = new Vue()
  Vue.prototype.$bus = {
  /**
   * @param {any} event 第一个参数是事件对象，第二个参数是接收到消息信息，可以是任意类型
   * @method $on  事件订阅, 监听当前实例上的自定义事件。https://cn.vuejs.org/v2/api/#vm-on
   * @method $off  取消事件订阅，移除自定义事件监听器。  https://cn.vuejs.org/v2/api/#vm-off  https://github.com/vuejs/vue/issues/3399
   * @method $emit  事件广播, 触发当前实例上的事件。 https://cn.vuejs.org/v2/api/#vm-emit
   * @method $once  事件订阅, 监听一个自定义事件，但是只触发一次，在第一次触发之后移除监听器。 https://cn.vuejs.org/v2/api/#vm-once
   */
    $on (...event) {
      eventHub.$on(...event)
    },
    $off (...event) {
      eventHub.$off(...event)
    },
    $once (...event) {
      eventHub.$once(...event)
    },
    $emit (...event) {
      eventHub.$emit(...event)
    }
  }
}
注册
//main.js

import BUS from './service/bus'
BUS(Vue)
注意事项
事件订阅($on)必须在事件广播($emit)前注册；
取消事件订阅($off)必须跟事件订阅($on)成对出现。
使用场景
跨路由组件使用eventbus通信
假设a路由跳转b路由需要通过eventbus通信，先观察路由跳转前后a,b组件的生命周期钩子函数，可以发现两者是交叉执行的。
由于事件订阅必须在事件广播前注册，所以事件订阅可以放在b组件beforeCreate,created,beforeMout钩子函数中，而事件广播可以放在a组件的beforeDestroy,destroyed中。
取消事件订阅必须跟事件订阅成对出现，否则会重复订阅，对javascript性能造成不必要的浪费。因此B组件销毁前需取消当前事件订阅。
图片描述

A组件

    beforeDestroy () {
    //事件广播
      this.$bus.$emit('testing', color)
    }
B组件

    created () {
    //事件订阅
      this.$bus.$on('testing', (res) => { console.log(res) })
    },
    beforeDestroy () {
      this.$bus.$off('testing')
    }
2.普通跨组件通信：由于两组件几乎同时加载，因此建议事件广播放在created钩子内，事件订阅放在mouted中即可。具体使用场景建议在两组件分别打印生命钩子函数进行准确判断。

  beforeCreate: function () {
    console.group('A组件 beforeCreate 创建前状态===============》')
  },
  created: function () {
    console.group('A组件 created 创建完毕状态===============》')
  },
  beforeMount: function () {
    console.group('x组件 beforeMount 挂载前状态===============》')
  },
  mounted: function () {
    console.group('x组件 mounted 挂载结束状态===============》')
  },
  beforeUpdate: function () {
    console.group('x组件 beforeUpdate 更新前状态===============》')
  },
  updated: function () {
    console.group('x组件 updated 更新完成状态===============》')
  },
  beforeDestroy: function () {
    console.group('x组件 beforeDestroy 销毁前状态===============》')
  },
  destroyed: function () {
    console.group('x组件 destroyed 销毁完成状态===============》')
  }