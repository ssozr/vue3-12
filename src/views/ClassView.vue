<style>
.card-img {
  height: 268px;
  width: 268px;
  object-fit: cover;
  object-position: center;
}
.imgI {
  width: 100%;
  height: 400px;
  object-fit: cover;
  object-position: center;
}
</style>

<template>
  <div class="container mb-15">
    <div class="row">
      <div class="text-center p-0 mb-10">
        <img :src="classData.imagesUrl" class="imgI" alt="">
      </div>
      <div class="mb-10">
        <h2 class="border-bottom border-primary border-3">{{ classData.title }} <span class="fs-6 ps-5">{{ classData.description }}</span></h2>
      </div>
      <div class="row">
        <div class="col-8 d-flex flex-column">
          <p class="fs-4 mb-15">{{ classData.content }}</p>
          <div class="d-flex justify-content-between  align-items-end">
            <ul class="p-0 m-0">
              <li><p class="fs-4">課程堂數:<span class="ms-4">{{ classData.origin_price }}堂</span></p></li>
              <li><p class="fs-4 m-0">課程總額:<span class="ms-4">NT${{ classData.price }}</span></p></li>
            </ul>
            <div class="d-flex">
              <button type="button" class="btn btn-primary" @click="addCart(classData)">收藏課程</button>
              <button type="button" class="btn btn-primary ms-6">立即上課</button>
            </div>
          </div>
        </div>
        <div class="col-4 text-center">
          <img :src="classData.imageUrl" alt="" class="card-img rounded-circle mb-4">
          <h3>授課老師:{{ classData.unit }}</h3>
        </div>
      </div>
<!--       <div class="row">
        <div class="col-6">
          <ul class="p-0">
            <li><p class="fs-4">課程堂數:<span class="ms-4">{{ classData.origin_price }}堂</span></p></li>
            <li><p class="fs-4">課程總額:<span class="ms-4">NT${{ classData.price }}</span></p></li>
          </ul>
        </div>
        <div class="col-6">
          <button type="button" class="btn btn-primary">收藏課程</button>
        </div>
      </div> -->
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
const { VITE_PATH, VITE_URL} = import.meta.env
export default{
  data () {
    return {
      classData: {},
      id: ''
      
    }
  },
  methods: {
    getClassData () {
      this.$http.get(`${VITE_URL}/v2/api/${VITE_PATH}/product/${this.id}`)
        .then((res) => {
          console.log(res)
          this.classData = res.data.product
        })
        .catch((err) => {
          console.log(err)
        })
    },
    addCart (item) {
      const data = {
        product_id: item.id,
        qty: 1
      }
      this.$http.post(`${VITE_URL}/v2/api/${VITE_PATH}/cart`, { data })
        .then((res) => {
          console.log(res)
          Swal.fire(`${res.data.message}`)
        })
        .catch((err) => {
          console.log(err)
        })
    }
  },
  mounted () {
    const { id } = this.$route.params
    this.id = id
    this.getClassData()
  }
}
</script>
