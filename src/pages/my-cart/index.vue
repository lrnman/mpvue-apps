<template>
  <div class="cart">
    <ul class="cart_details">
      <li v-for="(v,i) in cartList" :key="i" class="cart_list-item" @touchstart="touchS" @touchmove="touchM" @touchend="touchE" :data-index="i" :style="{marginLeft:v.leftVal}">
        <img :src="v.img" alt="">
        <div class="li_info">
          <p>{{v.text}}</p>
          <p>月售100单</p>
          <div class="li_b">
            <p class="price">¥{{v.price}}</p>
            <div class="count">
              <i class="lower" @click="lowerCart" :data-info="v" v-if="v.num>0">-</i>
              <span v-if="v.num>0">{{v.num}}</span>
              <i class="add" @click="addCart" :data-info="v">+</i>
            </div>
          </div>
        </div>
        <div class="list_item_del" :style="{right:-delBtnWidth+'px',width:delBtnWidth+'px'}" @click="delShop" :data-item="v">删除</div>
      </li>
    </ul>
    <div class="cart_bottom">
      <p>总价：{{count}}元</p>
      <div class="lis_bottom_btn">结算</div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        cartList: [],
        startX: 0,
        delBtnWidth: 60,
        leftMove: 0
      }
    },
    onShow() { //页面渲染就会触发
      let history = wx.getStorageSync('history') || [];
      this.cartList = history;
      this.cartList.forEach(e => {
        e.left = 0;
      })
      console.log(this.cartList)
    },
    mounted() {},
    methods: {
      touchS(e) {
        if (e.touches.length == 1) {
          this.startX = e.touches[0].clientX;
        }
      },
      touchM(e) {
        let index = e.currentTarget.dataset.index;
        if (e.touches.length == 1) {
          //手指滑动开始的位置记录
          let moveX = e.touches[0].clientX;
          let disX = this.startX - moveX;
          let left = 0;
          if (disX == 0 || disX < 0) { //如果移动距离小于等于0，位置不变
            left = "0px";
          } else if (disX > 0) { //移动距离大于0，left值等于手指移动距离
            left = "-" + disX + "px";
            if (disX >= this.delBtnWidth) {
              left = "-" + this.delBtnWidth + "px";
            }
          }
          // console.log(left)
          if (index != "" && index != null) {
            this.cartList[index].leftVal = left;
          }
        }
      },
      touchE(e) {
        let index = e.currentTarget.dataset.index;
        if (e.mp.changedTouches.length == 1) {
          //手指抬起的位置
          let endX = e.mp.changedTouches[0].clientX;
          let disX = this.startX - endX;
          let left = 0;
          //如果距离小于删除按钮的1/2，不显示删除按钮
          left = disX > this.delBtnWidth / 2 ? "-" + this.delBtnWidth + "px" : "0px";
          if (index !== "" && index != null) {
            this.cartList[index].leftVal = left;
          }
        }
      },
      delShop(e) {
        let {
          item
        } = e.currentTarget.dataset;
        console.log(item)
        this.cartList = this.cartList.filter(ele => ele.shopId != item.shopId)
        wx.setStorageSync('history', this.cartList)
      },
      lowerCart(e) {
        let {
          info
        } = e.target.dataset;
        this.cartList.forEach(ele => {
          if (ele.shopId == info.shopId) {
            ele.num--;
          }
        })
        //针对num等于0的数据仍保留做清空处理
        this.cartList && (this.cartList = this.cartList.filter(e => e.num != 0));
        wx.setStorageSync('history', this.cartList)
      },
      addCart(e) {
        let {
          info
        } = e.target.dataset;
        this.cartList.forEach(ele => {
          if (ele.shopId == info.shopId) {
            ele.num++;
          }
        })
        wx.setStorageSync('history', this.cartList)
      },
    },
    computed: {
      count: function() {
        let n = 0;
        this.cartList.forEach(e => {
          if (e.num > 0) {
            n += e.price * e.num;
          }
        })
        return this.cartList.length ? n : 0;
      }
    },
    components: {}
  }
</script>

<style lang="less">
  .cart {
    height: 100%;
    background: #f5f5f5;
    overflow: hidden;
  }
  .cart_details {
    height: 100%;
    padding-bottom: 80rpx;
    box-sizing: border-box;
    overflow-x: hidden;
    overflow-y: scroll;
  }
  .cart_list-item {
    padding: 16rpx;
    border-bottom: 1rpx solid #eee;
    display: flex;
    align-items: center;
    transition: margin-left 0.6s ease;
    width: 100%;
    box-sizing: border-box;
    position: relative;
    img {
      width: 120rpx;
      height: 120rpx;
      margin-right: 20rpx;
    }
    .li_info {
      flex-grow: 1;
      p {
        color: #666;
        font-size: 26rpx;
      }
      .li_b {
        display: flex;
        align-items: center;
        justify-content: space-between;
        .price {
          color: #f00;
        }
        .count {
          display: flex;
          align-items: center;
          .lower,
          .add {
            width: 30rpx;
            height: 30rpx;
            border-radius: 50%;
            background: #2395ff;
            color: #fff;
            line-height: 30rpx;
            text-align: center;
            font-size: 24rpx;
          }
          span {
            margin: 0 16rpx;
            font-size: 24rpx;
          }
        }
      }
    }
    .list_item_del {
      position: absolute;
      top: 0;
      bottom: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #fff;
      background: #ff8b03;
      font-size: 24rpx;
    }
  }
  .cart_bottom {
    height: 80rpx;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    box-sizing: border-box;
    background: #ccc;
    height: 80rpx;
    p {
      font-size: 24rpx;
      color: #666;
      margin-left: 20rpx;
      height: 100%;
      flex-grow: 1;
      line-height: 80rpx;
    }
    .lis_bottom_btn {
      display: flex;
      align-items: center;
      justify-content: center;
      background: #ff8b03;
      font-size: 24rpx;
      color: #fff;
      height: 100%;
      width: 160rpx;
    }
  }
</style>
