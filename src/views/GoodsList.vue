<template>
    <div>
        <nav-header></nav-header>
        <nav-bread>
            <span>Goods</span>
        </nav-bread>
        <div class="accessory-result-page accessory-page">
            <div class="container">
                <div class="filter-nav">
                    <span class="sortby">Sort by:</span>
                    <a href="javascript:void(0)" class="default cur">Default</a>
                    <a href="javascript:void(0)" class="price" @click="sortGoods">Price
                        <svg class="icon icon-arrow-short">
                            <use xlink:href="#icon-arrow-short"></use>
                        </svg>
                    </a>
                    <a href="javascript:void(0)" class="filterby stopPop" @click="showFilter">Filter by</a>
                </div>
                <div class="accessory-result">
                    <!-- filter -->
                    <div class="filter stopPop" id="filter" :class="{'filterby-show':isShowFilter}">
                        <dl class="filter-price">
                            <dt>Price:</dt>
                            <dd>
                                <a href="javascript:void(0)" :class="{cur:curFliterIndex=='all'}" @click="curSelect('all')">All</a>
                            </dd>
                            <dd v-for="(item,index) in priceFilter" :key="item.startPrice" @click="curSelect(index)">
                                <a :class="{cur:index==curFliterIndex}" href="javascript:void(0)">{{item.startPrice}} - {{item.endPrice}}</a>
                            </dd>
                        </dl>
                    </div>
                    <!-- search result accessories list -->
                    <div class="accessory-list-wrap">
                        <div class="accessory-list col-4">
                            <ul>
                                <li v-for="(item,index) in goodsList" :key="item.id">
                                    <div class="pic">
                                        <a href="#"><img v-lazy="'static/'+item.productImage" alt=""></a>
                                    </div>
                                    <div class="main">
                                        <div class="name">{{ item.productName }}</div>
                                        <div class="price">{{ item.salePrice }}</div>
                                        <div class="btn-area">
                                            <a href="javascript:;" class="btn btn--m" @click="saveToCart(item.productId)">加入购物车</a>
                                        </div>
                                    </div>
                                </li>
                            </ul>
                            <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="30">
                                <img src="../../static/loading-svg/loading-cubes.svg" alt="" v-show="loading">
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="md-overlay" v-show="isShowFilter" @click="closeFilter"></div>
        <modal :mdShow="mdShow" @close="closeModal">
            <p slot="message">
                请先登录，否则无法加入购物车!
            </p>
            <div slot="btnGroup">
                <a class="btn btn--m" @click="mdShow = false">关闭</a>
            </div>
        </modal>
        <modal :mdShow="mdShowCart" @close="closeModal">
            <p slot="message">
                <svg class="icon-status-ok">
                    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
                </svg>
                <span>加入购物车成功!</span>
            </p>
            <div slot="btnGroup">
                <a class="btn btn--m" href="javascript:;" @click="mdShowCart = false">继续购物</a>
                <router-link class="btn btn--m" href="javascript:;" to="/cart">查看购物车</router-link>
            </div>
        </modal>
        <nav-footer></nav-footer>
    </div>
</template>
<script>
import '@/assets/styles/base.css'
import '@/assets/styles/product.css'
import { _goodsList, _addCart } from '../service/service'
export default {
    data() {
        return {
            goodsList: [],
            sortFlag: true,
            page: 1,
            pageSize: 8,
            busy: true,
            loading: false,
            priceLevel: 'all',
            mdShow: false,
            mdShowCart: false,
            priceFilter: [
                {
                    startPrice: '0.00',
                    endPrice: '100.00'
                }, {
                    startPrice: '100.00',
                    endPrice: '500.00'
                }, {
                    startPrice: '500.00',
                    endPrice: '1000.00'
                }, {
                    startPrice: '1000.00',
                    endPrice: '5    000.00'
                }
            ],
            curFliterIndex: 'all',
            isShowFilter: false
        }
    },
    mounted() {
        this.getGoodsList()
    },
    methods: {
        async getGoodsList(flag) {
            let param = {params: {
                page: this.page,
                pageSize: this.pageSize,
                sort: this.sortFlag ? 1 : -1,
                priceLevel: this.priceLevel
            }}
            this.loading = true
            let result = await _goodsList(param)
            this.loading = false
            let res = result.data
            if (res.status === '0') {
                if (flag) {
                    this.goodsList = this.goodsList.concat(res.result.list)
                    if (res.result.count === 0 || res.result.count < this.pageSize) {
                        this.busy = true
                    } else {
                        this.busy = false
                    }
                } else {
                    this.goodsList = res.result.list
                    this.busy = false
                }
            } else {
                this.goodsList = []
            }
        },
        loadMore() {
            setTimeout(() => {
                this.page++
                this.getGoodsList(true)
            }, 500)
        },
        sortGoods() {
            this.sortFlag = !this.sortFlag
            this.page = 1
            this.getGoodsList()
        },
        curSelect(index) {
            this.curFliterIndex = index
            this.priceLevel = index
            this.page = 1
            this.isShowFilter = false
            this.getGoodsList()
        },
        async saveToCart(productId) {
            let res = await _addCart({'productId': productId})
            if (res.data.status === '0') {
                this.mdShowCart = true
            } else {
                this.mdShow = true
            }
        },
        showFilter() {
            this.isShowFilter = true
        },
        closeFilter() {
            this.isShowFilter = false
        },
        closeModal() {
            this.mdShow = false
            this.mdShowCart = false
        }
    }
}
</script>