<template lang="html">
  <div class="scroll-container">
    <div class="scroll-content">
      <slot name="default">
        <ul class="list-content" v-if="data.length > 0">
          <li class="list-item" v-for="item in data">{{item}}</li>
        </ul>
      </slot>
      <slot name="pullup" :pullUpLoad="options.pullUpLoad" :isPullUpLoad="isPullUpLoad" >
        <div class="pullup-wrapper" v-if="options.pullUpLoad">
          <div class="before-trigger" v-if="!isPullUpLoad">
            <span>{{pullUpTxt}}</span>
          </div>
          <div class="after-trigger" v-else>
            <loading></loading>
          </div>
        </div>
      </slot>
    </div>
    <slot name="pulldown"
          :pullDownRefresh="options.pullDownRefresh"
          :pullDownStyle="pullDownStyle"
          :beforePullDown="beforePullDown"
          :isPullingDown="isPullingDown"
          :bubbleY="bubbleY"
    >
      <div class="pulldown-wrapper" :style="pullDownStyle" v-if="options.pullDownRefresh">
        <div class="before-trigger" v-if="beforePullDown">
          <bubble :y="bubbleY"></bubble>
        </div>
        <div class="after-trigger" v-else>
          <div v-if="isPullingDown" class="loading">
            <loading></loading>
          </div>
          <div v-else><span>{{refreshTxt}}</span></div>
        </div>
      </div>
    </slot>
  </div>

</template>

<script>
  import Bscroll from './better-scroll/dist/bscroll';
  import loading from './component/loading';
  import bubble from './component/bubble';

  const blank = new Function();
  const options = {
    startX: 0,
    // 类型：Number,
    // 默认值：0
    // 作用：横轴方向初始化位置。
    startY: 0,
    // 类型：Number,
    // 默认值：0
    // 作用：纵轴方向初始化位置，见 Demo 。
    scrollX: false,
    // 类型：Boolean
    // 默认值: false
    // 作用：当设置为 true 的时候，可以开启横向滚动。
    // 备注：当设置 eventPassthrough 为 'horizontal' 的时候，该配置无效。
    scrollY: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：当设置为 true 的时候，可以开启纵向滚动。
    // 备注：当设置 eventPassthrough 为 'vertical' 的时候，该配置无效。
    freeScroll: false,
    // 类型：Boolean
    // 默认值：false
    // 作用：有些场景我们需要支持横向和纵向同时滚动，而不仅限制在某个方向，这个时候我们只要设置 freeScroll 为 true 即可。
    // 备注：当设置 eventPassthrough 不为空的时候，该配置无效。
    directionLockThreshold: 5,
    // 类型：Number
    // 默认值：5（不建议修改）
    // 作用：当我们需要锁定只滚动一个方向的时候，我们在初始滚动的时候根据横轴和纵轴滚动的绝对值做差，当差值大于 directionLockThreshold 的时候来决定滚动锁定的方向。
    // 备注：当设置 eventPassthrough 的时候，directionLockThreshold 设置无效，始终为 0。
    eventPassthrough: '',
    // 类型： String
    // 默认值：''
    // 可选值：'vertical'、'horizontal'
    // 作用：有时候我们使用 better-scroll 在某个方向模拟滚动的时候，希望在另一个方向保留原生的滚动（比如轮播图，我们希望横向模拟横向滚动，而纵向的滚动还是保留原生滚动，我们可以设置 eventPassthrough 为 vertical；相应的，如果我们希望保留横向的原生滚动，可以设置eventPassthrough为 horizontal）。
    // 备注：eventPassthrough 的设置会导致其它一些选项配置无效，需要小心使用它。
    click: false,
    // 类型：Boolean
    // 默认值：false
    // 作用：better-scroll 默认会阻止浏览器的原生 click 事件。当设置为 true，better-scroll 会派发一个 click 事件，我们会给派发的 event 参数加一个私有属性 _constructed，值为 true。
    tap: false,
    // 类型：Boolean | String
    // 默认值：false
    // 作用：因为 better-scroll 会阻止原生的 click 事件，我们可以设置 tap 为 true，它会在区域被点击的时候派发一个 tap 事件，你可以像监听原生事件那样去监听它，如 element.addEventListener('tap', doSomething, false);。如果 tap 设置为字符串, 那么这个字符串就作为自定义事件名称。如 tap: 'myCustomTapEvent'。
    bounce: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：当滚动超过边缘的时候会有一小段回弹动画。设置为 true 则开启动画。
    bounceTime: 700,
    // 类型：Number
    // 默认值：700（单位ms，不建议修改）
    // 作用：设置回弹动画的动画时长。
    momentum: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：当快速在屏幕上滑动一段距离的时候，会根据滑动的距离和时间计算出动量，并生成滚动动画。设置为 true 则开启动画。
    momentumLimitTime: 300,
    // 类型：Number
    // 默认值：300（单位ms，不建议修改）
    // 作用：只有在屏幕上快速滑动的时间小于 momentumLimitTime，才能开启 momentum 动画。
    momentumLimitDistance: 15,
    // 类型：Number
    // 默认值：15（单位px，不建议修改）
    // 作用：只有在屏幕上快速滑动的距离大于 momentumLimitDistance，才能开启 momentum 动画。
    swipeTime: 2500,
    // 类型：Number
    // 默认值：2500（单位ms，不建议修改）
    // 作用：设置 momentum 动画的动画时长。
    swipeBounceTime: 500,
    // 类型：Number
    // 默认值：500（单位ms，不建议修改）
    // 作用：设置当运行 momentum 动画时，超过边缘后的回弹整个动画时间。
    deceleration: 0.001,
    // 类型：Number
    // 默认值：0.001（不建议修改）
    // 作用：表示 momentum 动画的减速度。
    flickLimitTime: 200,
    // 类型：Number
    // 默认值：200（单位ms，不建议修改）
    // 作用：有的时候我们要捕获用户的轻拂动作（短时间滑动一个较短的距离）。只有用户在屏幕上滑动的时间小于 flickLimitTime ，才算一次轻拂。
    flickLimitDistance: 100,
    // 类型：Number
    // 默认值：100（单位px，不建议修改）
    // 作用：只有用户在屏幕上滑动的距离小于 flickLimitDistance ，才算一次轻拂。
    resizePolling: 60,
    // 类型：Number
    // 默认值：60（单位ms，不建议修改)
    // 作用：当窗口的尺寸改变的时候，需要对 better-scroll 做重新计算，为了优化性能，我们对重新计算做了延时。60ms 是一个比较合理的值。
    probeType: 0,
    // 类型：Number
    // 默认值：0
    // 可选值：1、2、3
    // 作用：有时候我们需要知道滚动的位置。当 probeType 为 1 的时候，会非实时（屏幕滑动超过一定时间后）派发scroll 事件；当 probeType 为 2 的时候，会在屏幕滑动的过程中实时的派发 scroll 事件；当 probeType 为 3 的时候，不仅在屏幕滑动的过程中，而且在 momentum 滚动动画运行过程中实时派发 scroll 事件。如果没有设置该值，其默认值为 0，即不派发 scroll 事件。
    preventDefault: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：当事件派发后是否阻止浏览器默认行为。这个值应该设为 true，除非你真的知道你在做什么，通常你可能用到的是 preventDefaultException。
    preventDefaultException: { tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/ },
    // 类型：Object
    // 默认值：{ tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/}
    // 作用：better-scroll 的实现会阻止原生的滚动，这样也同时阻止了一些原生组件的默认行为。这个时候我们不能对这些元素做 preventDefault，所以我们可以配置 preventDefaultException。默认值 {tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/}表示标签名为 input、textarea、button、select 这些元素的默认行为都不会被阻止。
    // 备注：这是一个非常有用的配置，它的 key 是 DOM 元素的属性值，value 可以是一个正则表达式。比如我们想配一个 class 名称为 test 的元素，那么配置规则为 {className:/(^|\s)test(\s|$)/}。
    HWCompositing: true,
    // 类型：Boolean
    // 默认值：true（不建议修改）
    // 作用：是否开启硬件加速，开启它会在 scroller 上添加 translateZ(0) 来开启硬件加速从而提升动画性能，有很好的滚动效果。
    // 备注：只有支持硬件加速的浏览器开启才有效果。
    useTransition: true,
    // 类型：Boolean
    // 默认值：true（不建议修改）
    // 作用：是否使用 CSS3 transition 动画。如果设置为 false，则使用 requestAnimationFrame 做动画。
    // 备注：只有支持 CSS3 的浏览器开启才有效果。
    useTransform: true,
    // 类型：Boolean
    // 默认值：true（不建议修改）
    // 作用：是否使用 CSS3 transform 做位移。如果设置为 false, 则设置元素的 top/left (这种情况需要 scroller 是绝对定位的)。
    // 备注：只有支持 CSS3 的浏览器开启才有效果。
    bindToWrapper: false,
    // 类型：Boolean
    // 默认值：false
    // 作用：move 事件通常会绑定到 document 上而不是滚动的容器上，当移动的过程中光标或手指离开滚动的容器滚动仍然会继续，这通常是期望的。当然你也可以把 move 事件绑定到滚动的容器上，bindToWrapper 设置为 true 即可，这样一旦移动的过程中光标或手指离开滚动的容器，滚动会立刻停止。
    disableMouse: undefined,
    // 类型：Boolean
    // 默认值：根据当前浏览器环境计算而来（不建议修改）
    // 作用：当在移动端环境（支持 touch 事件），disableMouse 会计算为 true，这样就不会监听鼠标相关的事件，而在 PC 环境，disableMouse 会计算为 false，就会监听鼠标相关事件，不建议修改该属性，除非你知道你在做什么。
    disableTouch: undefined,
    // 类型：Boolean
    // 默认值：根据当前浏览器环境计算而来（不建议修改）
    // 作用：当在移动端环境（支持 touch 事件），disableTouch 会计算为 false，这样会监听 touch 相关的事件，而在 PC 环境，disableTouch 会计算为 true，就不会监听 touch 相关事件。不建议修改该属性，除非你知道你在做什么。
    observeDOM: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：会检测 scroller 内部 DOM 变化，自动调用 refresh 方法重新计算来保证滚动的正确性。它会额外增加一些性能开销，如果你能明确地知道 scroller 内部 DOM 的变化时机并手动调用 refresh 重新计算，你可以把该选项设置为 false。
    autoBlur: true,
    // 类型：Boolean
    // 默认值：true
    // 作用：在滚动之前会让当前激活的元素（input、textarea）自动失去焦点。
    wheel: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置是为了做 Picker 组件用的，默认为 false，如果开启则需要配置一个 Object。
    // wheel:{
    //   selectedIndex: 0,
    //   rotate: 25,
    //   adjustTime: 400,
    //   wheelWrapperClass: 'wheel-scroll',
    //   wheelItemClass: 'wheel-item'
    // }
    // 备注：这是一个高级的配置，一般场景不需要配置，具体应用场景可见 Picker Demo 。想了解更多的细节可以去看 example 中的 picker 组件的代码。注意：如果配置为 Object 的时候wheelWrapperClass 和 wheelItemClass 必须对应于你的实例 better-scroll 的 wrapper 类名和 wrapper 内的子类名。二者的默认值是 "wheel-scroll"/"wheel-item"，如果你不配置或者配置的名称和你对应DOM节点的类名不一致的话会导致一个问题：滚动起来的时候点击一下终止滚动并不会触发 scrollEnd 事件，进而影响诸如城市选择器联动数据的这种组件的结果。
    snap: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置是为了做 Slide 组件用的，默认为 false，如果开启则需要配置一个 Object，例如：
    //
    // 注意：loop 为 true 是为了支持横向循环轮播，并不支持纵向，threshold 表示可滚动到下一个的阈值，easing 表示滚动的缓动函数。
    // 备注：这是一个高级的配置，一般场景不需要配置，具体应用场景可见 Slide Demo 。想了解更多的细节可以去看 example 中的 slide 组件的代码。
    scrollbar: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置可以开启滚动条，默认为 false。当设置为 true 或者是一个 Object 的时候，都会开启滚动条，例如：
    // scrollbar: {
    //  fade: true,
    //  interactive: false // 1.8.0 新增
    // }
    // fade 为 true 表示当滚动停止的时候滚动条是否需要渐隐，interactive 表示滚动条是否可以交互。 见 Demo 。了解更多的细节可以去看 example 中的 scroll 组件代码。
    pullDownRefresh: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置用于做下拉刷新功能，默认为 false。当设置为 true 或者是一个 Object 的时候，可以开启下拉刷新，例如：
    // pullDownRefresh: {
    //   threshold: 50,
    //   stop: 20
    // }
    // 可以配置顶部下拉的距离（threshold） 来决定刷新时机以及回弹停留的距离（stop），见 Demo 。 了解更多的细节可以去看 example 中的 scroll 组件代码。
    pullUpLoad: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置用于做上拉加载功能，默认为 false。当设置为 true 或者是一个 Object 的时候，可以开启上拉加载，例如：
    // pullUpLoad: {
    //   threshold: 50
    // }
    // 可以配置离（threshold）来决定开始加载的时机，见 Demo 。 了解更多的细节可以去看 example 中的 scroll 组件代码。
    mouseWheel: false,
    // 类型：Boolean | Object
    // 默认值：false
    // 作用：这个配置用于 PC 端的鼠标滚轮，默认为 false，。当设置为 true 或者是一个 Object 的时候，可以开启鼠标滚轮，例如：
    // mouseWheel: {
    //  speed: 20,
    //  invert: false
    // }
    // speed 表示鼠标滚轮滚动的速度，invert 为 true 表示滚轮滚动和时机滚动方向相反，见Demo。

  };
  const eventNames = ['beforeScrollStart', 'scrollStart', 'scroll', 'scrollCancel', 'scrollEnd', 'touchEnd', 'flick', 'refresh', 'destroy', 'pullingDown', 'pullingUp'];
  const fnNames = ['refresh', 'scrollTo', 'scrollBy', 'scrollToElement', 'stop', 'enable', 'disable', 'destroy', 'on', 'once', 'off'];
  export default {
    components: {
      loading,
      bubble,
    },
    props: {
      data: {
        type: Array,
        default() {
          return [];
        },
      },
      options: {
        type: Object,
        default() {
          return {};
        },
      },
    },
    data() {
      return {
        bscroll: null,
        beforePullDown: true,
        isRebounding: false,
        isPullingDown: false,
        isPullUpLoad: false,
        pullUpDirty: true,
        pullDownStyle: '',
        bubbleY: 0,
        pullDownInitTop: -50,
      };
    },
    computed: {
      pullUpTxt() {
        const moreTxt = this.options.pullUpLoad && this.options.pullUpLoad.txt && this.options.pullUpLoad.txt.more || '加载更多';
        const noMoreTxt = this.options.pullUpLoad && this.options.pullUpLoad.txt && this.options.pullUpLoad.txt.noMore || '没有更多数据了';
        return this.pullUpDirty ? moreTxt : noMoreTxt;
      },
      refreshTxt() {
        return this.options.pullDownRefresh && this.options.pullDownRefresh.txt || '下拉刷新';
      },
    },
    methods: {
      init() {
        const opt = {};
        // 读取通过props传入的options参数设置初始化参数
        for (const key in options) {
          if (this.options[key] !== undefined) {
            opt[key] = this.options[key];
          }
        }
        // 初始化better-scroll实例
        this.bscroll = new Bscroll(this.$el, opt);
        if (opt.pullDownRefresh) {
          this._initPullDownRefresh();
        }
        if (opt.pullUpLoad) {
          this._initPullUpLoad();
        }
        // better-scroll的事件触发时，冒泡给当前组件触发同名事件
        for (let i = 0, item; item = eventNames[i++];) {
          this.bscroll.on(item, (...args) => {
            args.unshift(item);
            this.$emit.apply(this, args);
          }, this);
        }
        // 将better-scroll实例方法，暴露到组件上（同名方法，如果实例无此方法，则注册一个空函数 new Function()）
        for (let i = 0, len = fnNames.length; i < len; i++) {
          this[fnNames[i]] = this.bindFn(this.bscroll, fnNames[i]);
        }
      },
      bindFn(bscroll, fnName) {
        return typeof bscroll[fnName] === 'function' ? function () {
          return bscroll[fnName].apply(bscroll, arguments);
        } : blank;
      },
      _initPullDownRefresh() {
        this.bscroll.on('pullingDown', () => {
          this.beforePullDown = false;
          this.isPullingDown = true;
        });

        this.bscroll.on('scroll', (pos) => {
          if (this.beforePullDown) {
            this.bubbleY = Math.max(0, pos.y + this.pullDownInitTop);
            this.pullDownStyle = `top:${Math.min(pos.y + this.pullDownInitTop, 10)}px`;
          } else {
            this.bubbleY = 0;
          }

          if (this.isRebounding) {
            this.pullDownStyle = `top:${10 - (this.options.pullDownRefresh.stop - pos.y)}px`;
          }
        });
      },
      forceUpdate(dirty) {
        if (this.options.pullDownRefresh && this.isPullingDown) {
          this.isPullingDown = false;
          this._reboundPullDown().then(() => {
            this._afterPullDown();
          });
        } else if (this.options.pullUpLoad && this.isPullUpLoad) {
          this.isPullUpLoad = false;
          this.bscroll.finishPullUp();
          this.pullUpDirty = dirty;
          this.refresh();
        } else {
          this.refresh();
        }
      },
      _initPullUpLoad() {
        this.bscroll.on('pullingUp', () => {
          this.isPullUpLoad = true;
        });
      },
      _reboundPullDown() {
        return new Promise((resolve) => {
          setTimeout(() => {
            this.isRebounding = true;
            this.bscroll.finishPullDown();
            resolve();
          }, this.bscroll.options.pullDownRefresh.stopTime || 600);
        });
      },
      _afterPullDown() {
        setTimeout(() => {
          this.pullDownStyle = `top:${this.pullDownInitTop}px`;
          this.beforePullDown = true;
          this.isRebounding = false;
          this.refresh();
        }, this.bscroll.options.bounceTime);
      },
    },
    watch: {
    },
    created() {
    },
    mounted() {
      this.init();
    },
    destroyed() {
      this.destroy();
    },
  };
</script>

<style lang="scss">
  .scroll-container {
    .scroll-content {
      position: relative;
      .pullup-wrapper{
        width: 100%;
        display: flex;
        justify-content :center;
        align-items :center;
        padding: 16px 0;
      }
    }
    .pulldown-wrapper {
      position: absolute;
      width: 100%;
      left: 0;
      display: flex;
      justify-content:center;
      align-items:center;
      transition: all;
      .after-trigger{
        margin-top: 10px;
      }
    }
  }
</style>
