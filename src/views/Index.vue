<template>
  <section class="index">
    <el-carousel class="carousel">
      <el-carousel-item v-for="(item, idx) in bannerPic" :key="idx" :style="{background: `#333 url(${ item }) center center no-repeat`, backgroundSize: 'cover'}"></el-carousel-item>
    </el-carousel>
    <div class="number-info info">
      <div class="number-list list">
        <div class="number-item" v-for="(item, idx) in number" :key="idx">
          <h3>{{ item.fromData | beautifyNumber }}{{ item.title === '用户总数' ? 'k+' : '' }}</h3>
          <p>{{ item.title }}</p>
        </div>
      </div>
    </div>

    <div class="sales-info info">
      <div class="sales-list list">
        <div class="sales-item">
          <h3>剁手排行</h3>
          <canvas id="chartBuyNum" width="100" height="100"></canvas>
        </div>
        <div class="sales-item">
          <h3>销量份额</h3>
          <canvas id="chartSalesNum"></canvas>
        </div>
      </div>
    </div>

    <div class="type-info info">
      <h2>
        <span class="ityped"></span>
      </h2>
    </div>

    <div class="user-info info">
      <h3>买家评价</h3>
      <ul>
        <li>
          <h4>不愿意透露姓名的黄昭为先生：</h4>
          <p>自从用了这个网站，我再也不去澳门皇家赌场赌博了。</p>
        </li>
        <li>
          <h4>可爱的唐柳柳：</h4>
          <p>在这个网站上，我终于买到了寻找多年的，可以更换电池的限量iPhone了！</p>
        </li>
        <li>
          <h4>厂妹小达达：</h4>
          <p>为什么没有OPPO卖？又不是不能开机！</p>
        </li>
      </ul>
    </div>

    <js-footer></js-footer>
  </section>
</template>

<script>
import http from 'axios'
import Chart from 'chart.js'
import TWEEN from 'tween.js'
import ityped from 'ityped'

import chartBuyNumOption from '../components/Index/chartBuyNum.js'
import chartSalesNumOption from '../components/Index/chartSalesNum.js'
import JsFooter from '../components/Common/Footer.vue'

export default {
  data () {
    return {
      bannerPic: [
        require('../public/images/index_banner_0.jpg'),
        require('../public/images/index_banner_1.jpg')
      ],
      chartBuyNum: null,
      chartSalesNum: null,
      number: {
        one: { title: '订单总数', fromData: 0, toData: 0 },
        two: { title: '累计金额', fromData: 0, toData: 0 },
        three: { title: '用户总数', fromData: 0, toData: 0 }
      }
    }
  },
  async mounted () {
    ityped.init(document.querySelector('.ityped'), {
      strings: ['注册就送 ￥5000', '踏踏实实做人，认认真真搞机', '今天大家的机儿都放假了吗？', '湘大首家线上手机旗舰店上线啦', ':)'],
      loop: true
    })
    let indexData = (await http.get('/index/info')).data
    let userRankList = { labels: [], data: [] }
    let goodsRankList = { labels: [], data: [] }
    indexData.userRank.forEach(function (ele) {
      userRankList.labels.push(ele.name)
      userRankList.data.push(ele.buyNum)
    }, this)
    indexData.goodList.forEach(function (ele) {
      goodsRankList.labels.push(ele.name + ' ' + ele.memory + 'G ' + ele.color)
      goodsRankList.data.push(ele.salesNum)
    }, this)
    this.number.one.toData = indexData.orderCount
    this.number.two.toData = indexData.orderSum
    this.number.three.toData = indexData.userCount
    this.initCanvas(userRankList, goodsRankList)
    this.animateNumber()
  },
  filters: {
    beautifyNumber (value) {
      let temp = value.toString().split('')
      let ret = []
      for (let i = temp.length - 1, cnt = 1; i >= 0; i--, cnt++) {
        ret.push(temp[i])
        cnt % 3 === 0 && temp[i - 1] && ret.push(',')
      }
      return ret.reverse().join('')
    }
  },
  methods: {
    async initCanvas (userRankList, goodsRankList) {
      // 剁手排行柱状图
      let chartBuyNumId = document.getElementById('chartBuyNum')
      chartBuyNumOption.data.labels = userRankList.labels
      chartBuyNumOption.data.datasets[0].data = userRankList.data
      this.chartBuyNum = new Chart(chartBuyNumId, chartBuyNumOption)

      chartSalesNumOption.data.labels = goodsRankList.labels
      chartSalesNumOption.data.datasets[0].data = goodsRankList.data

      let chartSalesNumId = document.getElementById('chartSalesNum')
      this.chartSalesNum = new Chart(chartSalesNumId, chartSalesNumOption)
    },
    animateNumber () {
      let vm = this
      function animate (time) {
        window.requestAnimationFrame(animate)
        TWEEN.update(time)
      }
      new TWEEN.Tween({ one: 0, two: 0, three: 0 })
        .easing(TWEEN.Easing.Quadratic.Out)
        .to({ one: vm.number.one.toData, two: vm.number.two.toData, three: vm.number.three.toData }, 1500)
        .onUpdate(function () {
          vm.number.one.fromData = this.one.toFixed(0)
          vm.number.two.fromData = this.two.toFixed(0)
          vm.number.three.fromData = this.three.toFixed(0)
        })
        .delay(800)
        .start()
      animate()
    }
  },
  components: {
    JsFooter
  }
}
</script>

<style lang="stylus">
$mobile-width = 767px

.index
  padding 0 !important
  .carousel
    width: 100%
    // margin-bottom: 20px

  .info
    overflow: hidden
    h2
      text-align: center
      border-bottom: 3px dashed #eee
      padding-bottom: 10px
      font-size 50px
    .list
      display: flex
      flex-wrap: wrap
      justify-content: space-around
  .number-info
    background-color #fff
    text-align center
    .number-item
      width calc(100% / 3)
    h3
      color #4688f1
      font-size 26px

  .sales-info
    padding 20px 0
    width: 90%
    margin: 0 auto
    .sales-item
      background-color #fff
      padding 20px
      box-shadow 1px 2px 5px rgba(0,0,0,0.1)
      box-sizing border-box
      width 35vw
    @media (max-width: $mobile-width)
      width: 96%
      .sales-item
        // width 80vw
        // width 90vw
        width 100%
        margin-bottom: 50px
        &:last-child
          margin-bottom: 0

  .user-info
    width 70%
    margin 0 auto 50px
    background-color #fff
    padding 20px
    box-shadow 1px 2px 5px rgba(0,0,0,0.1)
    border-radius 4px
    box-sizing border-box
    ul
      border-top 1px solid #eee
    h4
      color #4688f1
    @media (max-width: $mobile-width)
      width 96%

  .ityped-cursor
    color #4688f1
</style>
