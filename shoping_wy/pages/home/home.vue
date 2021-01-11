<template>
	<view>
    <!-- 使用自定义的搜索组件 -->
    <view class="search-box">
      <my-search @click="gotoSearch"></my-search>
    </view>
  <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true" >
    <swiper-item v-for="(item, i) in swiperList" :key="i">
      <navigator  class="swiper-item" :url="'../../subpkg/goods-detail/goods-detail?goods_id=' + item.goods_id">
        <image :src="item.image_src"></image>
      </navigator >
    </swiper-item>
  </swiper>
  <!-- 分类导航 -->
  <view class="nav-list">
    <view class="nav-item" v-for="(item,i) in navList" @click="navClickHandler(item)">
      <image :src="item.image_src" mode="" class="nav-img"></image>
    </view>
  </view>
  <!-- 楼层区域 -->
  <view class="floor-list">
    <!-- 楼层 item项 -->
    <view class="floor-item" v-for="(item,i) in floorList" :key="i">
      <!-- 楼层标题 -->
      <image :src="item.floor_title.image_src" class="floor-title"></image>
      <!-- 楼层图片区域 -->
      <view class="floor-img-box">
        <!-- 左侧大图片的盒子 -->
        <navigator class="left-img-box" :url="item.product_list[0].url">
          <image :src="item.product_list[0].image_src" mode="widthFix" :style="{width:item.product_list[0].image_width+'rpx'}"></image>
        </navigator>
        <!-- 右侧的四个小图片的盒子 -->  
        <view class="right-img-box">
            <navigator class="right-item-img" v-for="(item2,i2) in item.product_list" :key="i2" v-if="i2 !== 0" :url="item2.url">
              <image :src="item2.image_src" mode="widthFix" :style="{width:item2.image_width + 'rpx'}"></image>
            </navigator>
        </view>
      </view>
    </view>
  </view>
	</view>
</template>

<script>
  import badgeMix from '@/minxins/tabbar-badge.js'  
	export default {
    minxins:[badgeMix],
		data() {
			return {
				swiperList: [],
        navList: [],
        floorList: []
			};
		},
     onLoad() {
        // 2. 在小程序页面刚加载的时候，调用获取轮播图数据的方法
        this.getSwiperList()
        this.getNavList()
        this.getFloorList()
      },
      methods: {
        // 3. 获取轮播图数据的方法
        async getSwiperList() {
          // 3.1 发起请求
          const { data: res } = await uni.$http.get('/api/public/v1/home/swiperdata')
          // 3.2 请求失败
          console.log(res)
          if (res.meta.status !== 200) return uni.$showMsg()
          // 3.3 请求成功，为 data 中的数据赋值
          this.swiperList = res.message
          uni.$showMsg('数据请求成功')
        },
       async getNavList() {
       const {data: res} = await uni.$http.get('/api/public/v1/home/catitems')
       console.log(res)
       if(res.meta.status !==200) return uni.$showMsg()
       this.navList = res.message
        },
        navClickHandler(item) {
          if(item.name === '分类') {
            uni.switchTab({
              url:'/pages/cate/cate'
            })
          }
        },
        // 定义获取楼层的列表数据方法
       async getFloorList() {
          const {data:res} = await uni.$http.get('/api/public/v1/home/floordata')
          if(res.meta.status !== 200) return uni.$showMsg()
          //  对数据进行处理
          res.message.forEach(floor => {
            floor.product_list.forEach(prod => {
              prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]})
          })
          this.floorList = res.message
        },
        gotoSearch() {
          uni.navigateTo({
            url: '/subpkg/search/search'
          })
        }
      },
      
      
	}
</script>

<style lang="scss">
  swiper {
    // width: 100%;
   height: 330rpx;
   .swiper-item,
   image {
     width: 100%;
     height: 100%;
   }
  }
  .nav-list {
    display: flex;
    justify-content: space-around;
    margin: 15px 0;
    .nav-img {
      width: 128rpx;
      height: 148rpx;
    }
  }
  .floor-title {
    height: 60rpx;
    width: 100%;
    display: flex;
  }
  .right-img-box {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
  }
  .floor-img-box {
    display: flex;
    padding-left: 10rpx;
  }
  .search-box {
    // 设置定位效果为“吸顶”
    position: sticky;
    // 吸顶的“位置”
    top: 0;
    // 提高层级，防止被轮播图覆盖
    z-index: 999;
  }
</style>
