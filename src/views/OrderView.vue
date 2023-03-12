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
      <ul class="order-progress pt-4">
        <li class="active">填寫資料</li>
        <li :class="{active:num >= 2}">訂單確認</li>
        <li :class="{active:num >= 3}">完成訂單</li>
      </ul>
    </div>
    <div class="container-md container-fuli d-none">
      <div class="row">
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
    </div>
    <div class="container">
      <div class="row">
        <v-form v-slot="{ errors }" @submit="onSubmit">
          <div class="row">
            <div class="mb-3 col-md-6"> <!-- 信箱 -->
              <label for="email" class="form-label">信箱: <span v-if="errors['email']" class="text-danger">{{ errors.email }}</span></label>
              <v-field id="email"
                name="email"
                type="email"
                class="form-control"
                :class="{ 'is-invalid': errors['email'] }"
                placeholder="請輸入 Email" rules="email|required"
                v-model="user.email">
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
                v-model="user.phone">
              </v-field>
              <error-message name="電話" class="invalid-feedback"></error-message>
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
                v-model="user.address"
                :disabled="!address.areaName">
              </v-field>
              <error-message name="地址" class="invalid-feedback"></error-message>
            </div>
          </div>
          <button type="submit" class="btn btn-primary">Submit</button>
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
      cartData: [],
      dalModal: '',
      delData: {},
      delId: '',
      disabled: false,
      addressData: '',
      address: {
        city: '',
        areaName: '',
        index: '',
        areaList: [],
      },
      user: {
        email: '',
        address: '',
        phone: ''
      }
    }
  },
  components: {
  },
  methods: {
    nextBtn(){
      this.num ++
    },
    preBtn(){
      this.num--
    },
    getCartData () {
      this.$http.get(`${VITE_URL}/v2/api/${VITE_PATH}/cart`)
        .then((res) => {
          this.cartData = res.data.data
        })
        .catch((err) => {
          console.log(err)
        })
    },
    openDelModal (item) {
      this.deldata = {}
      this.delModal.show()
      this.delId = item.id
      this.delData = item.product
    },
    delCart () {
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
    changeQty (num, id) {
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
    getAddressData () {
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
    city () {
      console.log(this.address.city)
      this.address.areaName = ''
      const index = this.addressData.findIndex(item => item.CityName === this.address.city);
      this.address.index = index
      this.address.areaList = { ...this.addressData[index].AreaList }
    },
    isPhone(value) {
    const phoneNumber = /^(09)[0-9]{8}$/
    return phoneNumber.test(value) ? true : '需要正確的電話號碼'
    }
  },
  mounted () {
    this.getCartData()
    this.delModal = new Modal(this.$refs.delModal)
    this.getAddressData()
  }
}
</script>
