<style>
.order-progress {
  height: 90px;
}
.order-progress li {
  list-style-type: none;
  float:left;
  width: 33.3%;
  height: 50px;
  position:relative;
  text-align:center;
}

.order-progress li:before {
  content:"";
  counter-increment: step;
  width:30px;
  height:30px;
  line-height:30px;
  border:1px solid #ddd;
  display:block;
  text-align:center;
  margin:0 auto 10px auto;
  border-radius:50%;
  background-color:#fff;
}

.order-progress li:after {
  content:"";
  border:1px solid #ddd;
  position: absolute;
  width:100%;
  height:1px;
  background-color:#ddd;
  top:15px;
  left:-50%;
  z-index:-1;
}

.order-progress li:first-child:after{
  content:none;
}

.order-progress li.active {
  color:#F79E1B;
}

.order-progress li.active:before {
  border-color:#F79E1B;
}

.order-progress li.active:after {
  background-color:#F79E1B;
}

</style>

<template>
  <div class="container-fuli">
    <div class="row">
      <ul class="order-progress pt-4"> <!-- 流程 -->
        <li class="active">訂單確認</li>
        <li :class="{active:num >= 2}"> 填寫資料</li>
        <li :class="{active:num >= 3}">付款成功</li>
      </ul>
    </div>
    <div class="container"> <!-- 資料填寫 -->
      <div class="row"> 
        <v-form v-slot="{ errors }" @submit="onSubmit">
          <div class="row" v-if="num === 1"> 
        <h2 class="text-center">訂單確認</h2>
        <div class="row">
          <table class="table align-middle">
            <thead>
              <tr class="text-center">
                <th scope="col">課程名稱</th>
                <th scope="col">授課老師</th>
                <th scope="col">總課程數</th>
                <th scope="col">購買數量</th>
                <th scope="col">價格</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, i) in cartData.carts" :key="i" class="text-center">
                <td>
                  <h2>{{ item.product.title }}</h2>
                </td>
                <td>{{ item.product.unit }}</td>
                <td >{{ item.product.origin_price * item.qty}}堂</td>
                <td>
                  <div class="dropdown" >
                    <button :disabled="disabled" class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
                      {{ item.qty }} 
                    </button>
                    <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1" >
                      <li v-for="(num, i) in 20" :key="i"><a  @click="changeQty(num,item.id)" class="dropdown-item">{{ num }}</a></li>
                    </ul>
                  </div>
                </td>
                <td>{{ item.total }}</td>
                <td><button type="button" class="btn btn-primary rounded-pill" @click=" openDelModal(item)">刪除</button></td>
              </tr>
              <tr class="text-center">
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td class="align-bottom"><h2 class="border-bottom border-primary border-3 mb-0">總金額:NT <span class="ms-1">{{ cartData.total }}</span></h2></td>
              </tr>
            </tbody>
          </table>
          <div class="modal" tabindex="-1" ref="delModal">
            <div class="modal-dialog">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 v-if="delData.title" class="modal-title">{{ delData.title }}</h5>
                  <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                  <p>確定移出購物車?</p>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                  <button type="button" class="btn btn-primary" @click="delCart(delData.id)">確認移出</button>
                </div>
              </div>
            </div>
          </div>
        </div>
          </div>  
          <div class="row" v-if="num ===2">
            <div class="mb-3 col-md-6"> <!-- 姓名 -->
              <label for="name" class="form-label">姓名: <span v-if="errors['姓名']" class="text-danger">{{ errors['姓名'] }}</span></label>
              <v-field id="name"
                name="姓名"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errors['姓名'] }"
                placeholder="請輸入 姓名" rules="required"
                v-model="userData.user.name">
              </v-field>
              <error-message name="姓名" class="invalid-feedback"></error-message>
            </div>
            <div class="mb-3 col-md-6"> <!-- 信箱 -->
              <label for="email" class="form-label">信箱: <span v-if="errors['email']" class="text-danger">{{ errors.email }}</span></label>
              <v-field id="email"
                name="email"
                type="email"
                class="form-control"
                :class="{ 'is-invalid': errors['email'] }"
                placeholder="請輸入 Email" rules="email|required"
                v-model="userData.user.email">
              </v-field>
              <error-message name="email" class="invalid-feedback"></error-message>
            </div>
            <div class="mb-3 col-md-6"> <!-- 電話 -->
              <label for="phone" class="form-label">手機號碼: <span v-if="errors['電話']" class="text-danger">{{ errors['電話'] }}</span></label>
              <v-field id="phone"
                name="電話"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errors['電話'] }"
                placeholder="0912345678"
                :rules="isPhone"
                v-model="userData.user.phone">
              </v-field>
              <error-message name="電話" class="invalid-feedback"></error-message>
            </div>
            <div class="mb-3 col-md-6"> <!-- 付款方式 -->
              <label for="payment" class="form-label">付款方式: <span v-if="errors['付款方式']" class="text-danger">{{ errors['付款方式'] }}</span></label>
                <v-select id="payment"
                  name="付款方式"
                  :items="paymentOptions"
                  class="form-control"
                  :class="{ 'is-invalid': errors['付款方式'] }"
                  placeholder="請選擇付款方式"
                  v-model="userData.paymentMethod">
                </v-select>
                <error-message name="付款方式" class="invalid-feedback"></error-message>
            </div>
            <div class="input-group mb-3"> <!-- 地址 -->
              <label style="width: 100%;" for="地址" class="form-label">地址: <span v-if="errors['地址']" class="text-danger">{{ errors.地址 }}</span></label>
              <select class="btn btn-outline-secondary" @change="city" v-model="address.city"> <!-- 縣市 -->
                <option value="" v-if="!address.city">請選擇</option>
                <option :value="address.CityName" v-for="(address, index) in addressData" :key="index">{{ address.CityName }}</option>
              </select>
              <select class="btn btn-outline-secondary" :disabled="!address.city" v-model="address.areaName"> <!-- 縣市 -->
                <option value=""  v-if="!address.areaName">請選擇</option>
                <option :value="areaName.AreaName" v-for="(areaName, index) in address.areaList" :key="index">{{ areaName.AreaName }}</option>
              </select>
              <v-field id="地址"
                name="地址"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errors['地址'] }"
                placeholder="請輸入 路段 門牌號碼" rules="required"
                v-model="userData.user.address"
                :disabled="!address.areaName">
              </v-field>
              <error-message name="地址" class="invalid-feedback"></error-message>
            </div>
            <div class="input-group"><!-- 留言 -->
              <span class="input-group-text">留言區</span>
              <textarea class="form-control" aria-label="With textarea" placeholder="可以說說自己，或是想告訴我們的" v-model="userData.message"  ></textarea>
            </div>
          </div>
          <button type="button" v-if="num === 2" @click="preBtn">上一步</button>
          <button type="button" v-if="num === 1" @click="nextBtn">下一步</button>
          <button type="submit" v-if="num === 2"  :disabled="errors['email','電話','地址']">送出訂單</button>
        </v-form>
      </div>
    </div>
  </div>
</template>

<script>
import { Modal } from 'bootstrap'
import Swal from 'sweetalert2'
const { VITE_URL, VITE_PATH } = import.meta.env

export default {
    data () {
    return {
      paymentOptions:['ATM','信用卡','電信支付'],
      num:1,
      cartData: [], /* 購物車資料 */
      dalModal: '', /* 刪除訂單Modal */
      delData: {}, /* 刪除訂單資烙 */
      delId: '', /* 刪除訂單ID */
      disabled: false,
      addressData: '', /* 縣市區域資料 */
      address: { /* 地址填寫 */
        city: '',
        areaName: '',
        index: '',
        areaList: [],
      },
      userData: { /* 訂單客戶資料 */
        user: {
        name: '',
        email: '',
        address: '',
        phone: ''
      },
      message:'',
      paymentMethod: ''
      }
    }
  },
  components: {
  },
  methods: {
    nextBtn(){ /* 流程下一步 */
      this.num ++
    },
    preBtn(){ /* 流程上一步 */
      this.num--
    },
    getCartData () { /* 獲取購物車資料 */
      this.$http.get(`${VITE_URL}/v2/api/${VITE_PATH}/cart`)
        .then((res) => {
          this.cartData = res.data.data
        })
        .catch((err) => {
          console.log(err)
        })
    },
    openDelModal (item) { /* 開啟刪除Modal */
      this.deldata = {}
      this.delModal.show()
      this.delId = item.id
      this.delData = item.product
    },
    delCart () { /* 刪除購物車資料 */
      this.$http.delete(`${VITE_URL}/v2/api/${VITE_PATH}/cart/${this.delId}`)
      .then((res) => {
        console.log(res)
        this.delData = {}
        this.delId = ""
        this.delModal.hide()
        this.getCartData()
        Swal.fire(`${res.data.message}`)
      })
      .catch((err) => {
        console.log(err)
      })
    },
    changeQty (num, id) { /* 更改購物車產品數量 */
      this.disabled = true
      const data = {
        product_id:id,
        qty:num
      }
      this.$http.put(`${VITE_URL}/v2/api/${VITE_PATH}/cart/${id}`,{ data })
        .then((res) => {
          console.log(res)
          Swal.fire(`${res.data.message}`)
          this.disabled = false
          this.getCartData()
        })
        .catch((err) => {
          console.log(err)
        })
    },
    getAddressData () { /* 獲取縣市區域資料 */
      this.$http.get(`https://raw.githubusercontent.com/donma/TaiwanAddressCityAreaRoadChineseEnglishJSON/master/AllData.json`)
        .then((res) => {
          this.addressData = res.data
          console.log(res)
          console.log(this.addressData)
        })
        .catch((err) => {
          console.log(err)
        })
    },
    city () { /* 地址填寫市區selec */
      console.log(this.address.city)
      this.address.areaName = ''
      const index = this.addressData.findIndex(item => item.CityName === this.address.city);
      this.address.index = index
      this.address.areaList = { ...this.addressData[index].AreaList }
    },
    isPhone(value) { /* 手機號碼判斷 */
      const phoneNumber = /^(09)[0-9]{8}$/
      return phoneNumber.test(value) ? true : '需要正確的電話號碼'
    }
  },
  mounted () {
    this.getCartData() /* 獲取購物車資料 */
    this.delModal = new Modal(this.$refs.delModal) /* Modal */
    this.getAddressData() /* 獲取縣市區域資料 */
  }
}
</script>
