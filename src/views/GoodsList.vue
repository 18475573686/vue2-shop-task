<template>
    <div>
      <nav-header></nav-header>
      <nav-bread>
        <span>商品列表</span>
      </nav-bread>
      <div class="accessory-result-page">
        <div class="container">
          <div class="filter-nav">
            <span class="sortby">排序:</span>
            <a href="javascript:void(0)" class="default cur" @click.stop="defaultSort">默认</a>
            <a href="javascript:void(0)" class="price" :class="{'sort-up': sortFlag}" @click.stop="sortGoods">价格 <svg class="icon icon-arrow-short"><use xlink:href="#icon-arrow-short"></use></svg></a>
            <a href="javascript:void(0)" class="filterby" @click.stop="showFilterPop">筛选</a>
          </div>
          <div class="accessory-result">
            <!-- filter -->
            <div class="filter" id="filter" :class="{'filterby-show': filterBy}">
              <dl class="filter-price">
                <dt>价格区间:</dt>
                <dd><a href="javascript:void(0)" @click="setPriceFilter('all')" :class="{'cur': priceChecked=='all'}">选择价格</a></dd>
                <dd v-for="(item, index) of priceFilter">
                  <a href="javascript:void(0)" @click="setPriceFilter(index)" :class="{'cur': priceChecked==index}">￥ {{item.startPrice}} - {{item.endPrice}} 元</a>
                </dd>
              </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
              <div class="accessory-list col-4">
                <ul>
                  <li v-for="(item, index) of goodsList" :key="item.productId">
                    <div class="pic">
                      <a href="#"><img v-lazy="`/static/` + item.productImage" alt=""></a>
                    </div>
                    <div class="main">
                      <div class="name">{{item.productName}}</div>
                      <div class="price">{{item.salePrice | currency('￥')}}</div>
                      <div class="btn-area">
                        <a href="javascript:;" class="btn btn--m" @click.stop="addCart(item.productId)">加入购物车</a>
                      </div>
                    </div>
                  </li>
                </ul>
              </div>
              <div class="view-more-normal"
                   v-infinite-scroll="loadMore"
                  infinite-scroll-disabled="busy"
                  infinite-scroll-distance="20">
              <img src="./../../static/loading-svg/loading-spinning-bubbles.svg" v-show="loading">
             </div>
            </div>
          </div>
        </div>
      </div>
      <!-- 商品加入购物车时，请先登录的模态框 start -->
      <modal v-bind:mdShow="mdShow" v-on:close="closeModal">
        <p slot="message">
          请先登录，否则无法加入到购物车中！
        </p>
        <div slot="btnGroup">
          <a href="javascript:void(0)" class="btn btn--m" @click="mdShow = false">关闭</a>
        </div>
      </modal>
      <!-- 商品加入购物车时，请先登录的模态框 end -->
      <!-- 商品成功加入购物模态框 start -->
      <modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
        <p slot="message">
          <svg class="icon-status-ok">
            <use xmlns:xlink="http:///www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
          </svg>
          <span>加入购物车成功！</span>
        </p>
        <div slot="btnGroup">
          <a href="javascript:void(0)" class="btn btn--m" @click="mdShowCart = false">继续购物</a>
          <router-link class="btn btn--m btn--red" href="javascript:void(0)" to="/cart">查看购物车</router-link>
        </div>
      </modal>
      <!-- 商品成功加入购物模态框 end -->
      <div class="md-overlay" v-show="filterBy" @click.stop="closePop"></div>
      <nav-footer></nav-footer>
    </div>
</template>

<script>
import './../assets/css/base.css'
import './../assets/css/goods-list.css'
import NavHeader from './../components/NavHeader.vue'
import NavBread from './../components/NavBread'
import NavFooter from './../components/NavFooter.vue'
import Modal from './../components/Modal.vue'
import {currency} from './../utils/currency'
import axios from 'axios'
export default {
  name: 'GoodsList',
  data () {
    return {
      goodsList: [],
      priceFilter: [
        {
          startPrice:'0.00',
          endPrice:'100.00'
        },
        {
          startPrice:'100.00',
          endPrice:'500.00'
        },
        {
          startPrice:'500.00',
          endPrice:'1000.00'
        },
        {
          startPrice:'1000.00',
          endPrice:'2000.00'
        },
        {
          startPrice:'2000.00',
          endPrice:'3000.00'
        },
        {
          startPrice:'3000.00',
          endPrice:'6000.00'
        }
      ],
      priceChecked: 'all',
      filterBy: false,
      page: 1,
      pageSize: 8,
      sortFlag: true,
      busy:true,
      loading:false,
      mdShow: false,
      mdShowCart: false
    }
  },
  mounted () {
    this.getGoodsList();
  },
  filters: {
    currency: currency
  },
  components: {
    NavHeader,
    NavBread,
    NavFooter,
    Modal
  },
  methods: {
    getGoodsList (flag) {
      var param = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag ? 1 : -1,
        priceLevel: this.priceChecked
      }
      this.loading = true;
      axios.get('/goods/list', {
        params: param
      }).then(res => {
        this.loading = false;
        if (res.data.status == '0') {
          if (flag) {
            this.goodsList = this.goodsList.concat(res.data.result.list);
            if (res.data.result.count === 0) {
              this.busy = true;
            } else {
              this.busy = false;
            }
          }else {
            this.goodsList = res.data.result.list;
            this.busy = false;
          }
        }else {
          this.goodsList = [];
        }
      });
    },
    setPriceFilter (index) {
      this.priceChecked = index;
      this.page = 1;
      this.getGoodsList();
    },
    showFilterPop () {
      this.filterBy = true;
    },
    closePop () {
      this.filterBy = false;
      this.mdShowCart = false;
    },
    sortGoods () {
      this.sortFlag = !this.sortFlag;
      this.page = 1;
      this.getGoodsList();
    },
    defaultSort () {
      this.sortFlag = true;
      this.page = 1;
      this.getGoodsList();
    },
    loadMore () {
      this.busy = true;
      setTimeout(() => {
        this.page++;
        this.getGoodsList(true);
      }, 500);
    },
    addCart (productId) {
      axios.post('/goods/addCart', {
        productId: productId
      }).then(res => {
        var res = res.data;
        if (res.status==0) {
          // alert('加入成功');
          this.mdShowCart = true;
          this.$store.commit('changeCartCount', 1);
        } else {
          // alert('Error msg:' + res.msg);
          this.mdShow = true;
        }
      });
    },
    closeModal () {
      this.mdShow = false;
      this.mdShowCart = false;
    }
  }
}
</script>

<style scoped>
</style>
