<template>
  <div class="container gx-6">
    <div class="row gx-3 bg-light py-3">
      <div class="col-md-4">
        <div class="list-group">
          <a
            @click.prevent="addSelect(product)"
            v-for="(product, key) in products"
            :key="product.name"
            :class="{ active: product.name === tempSelected.name }"
            href="#"
            class="list-group-item list-group-item-action"
          >
            <h6 class="card-title mb-1 text-start">
              {{ key + 1 }}.{{ product.name }}
            </h6>
            <div class="d-flex align-items-center justify-content-between">
              <p class="mb-0">
                <small>{{ product.engName }}</small>
              </p>
              <p class="mb-0">
                <small>NT$ {{ product.price }}</small>
              </p>
            </div>
          </a>
        </div>
      </div>
      <div class="col-md-8">
        <div class="card mb-2">
          <!-- 判斷  !tempSelected.hasOwnProperty('name') 有無 沒有東西就屏蔽-->
          <div
            v-if="!tempSelected.hasOwnProperty('name')"
            class="
              position-absolute
              text-white
              d-flex
              align-items-center
              justify-content-center
            "
            style="
              top: 0;
              bottom: 0;
              left: 0;
              right: 0;
              background-color: rgba(0, 0, 0, 0.65);
              z-index: 100;
            "
          >
            請先選擇飲品
          </div>
          <div class="card-body px-4">
            <div class="mb-3">
              <label for="productNum" class="form-label">數量</label>
              <input
                v-model.number="tempSelected.cup"
                type="number"
                class="form-control"
                id="productNum"
                placeholder="數量"
                min="1"
              />
            </div>
            <div class="mb-3">
              <label for="productNum" class="form-label d-block">冰塊*</label>
              <div
                v-for="ice in iceType"
                :key="ice"
                class="form-check form-check-inline"
              >
                <input
                  v-model="tempSelected.ice"
                  :value="ice"
                  :id="ice"
                  :disabled="
                    !tempSelected.hasOwnProperty('defaults') ||
                    (tempSelected.defaults.ice !== '' &&
                      tempSelected.defaults.ice !== ice)
                  "
                  class="form-check-input"
                  name="iceType"
                  type="radio"
                />
                <label class="form-check-label" :for="ice">{{ ice }}</label>
              </div>
            </div>
            <div class="mb-3">
              <label for="productNum" class="form-label d-block">甜度*</label>
              <div
                v-for="sugar in sugarType"
                :key="sugar"
                class="form-check form-check-inline"
              >
                <input
                  v-model="tempSelected.sugar"
                  :value="sugar"
                  :id="sugar"
                  :disabled="
                    !tempSelected.hasOwnProperty('defaults') ||
                    tempSelected.defaults.toppings.includes(topping)
                  "
                  class="form-check-input"
                  name="sugarType"
                  type="radio"
                />
                <label class="form-check-label" :for="sugar">{{ sugar }}</label>
              </div>
            </div>
            <div class="mb-3">
              <label for="productNum" class="form-label d-block">加料</label>
              <div
                v-for="toppings in toppingsType"
                :key="toppings"
                class="form-check form-check-inline"
              >
                <input
                  v-model="tempSelected.toppings"
                  :value="toppings"
                  :id="toppings"
                  :disabled="
                    !tempSelected.hasOwnProperty('defaults') ||
                    tempSelected.defaults.toppings.includes(toppings)
                  "
                  class="form-check-input"
                  type="checkbox"
                />
                <label class="form-check-label" :for="toppings">{{
                  toppings
                }}</label>
              </div>
            </div>
            <div class="mb-3">
              <label for="productNotice" class="form-label">備註</label>
              <textarea
                v-model="tempSelected.note"
                class="form-control"
                id="productNotice"
                rows="2"
              ></textarea>
            </div>
            <div class="d-flex gap-2">
              <button
                @click="reset"
                class="btn btn-outline-primary w-100"
                type="button"
              >
                取消
              </button>
              <button
                @click="addToOrder(tempSelected)"
                :disabled="!tempSelected.hasOwnProperty('name')"
                class="btn btn-primary w-100"
                type="button"
              >
                加入
              </button>
            </div>
          </div>
        </div>
        <div class="card">
          <div class="card-body">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">品項</th>
                  <th scope="col">冰塊</th>
                  <th scope="col">甜度</th>
                  <th scope="col">加料</th>
                  <th scope="col">單價</th>
                  <th scope="col">數量</th>
                  <th scope="col">小計</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="order in orderList" :key="order">
                  <th scope="row">
                    {{ order.name }}<br />
                    <small class="text-muted fw-normal"
                      >備註：{{ order.note }}</small
                    >
                  </th>
                  <td>{{ order.ice }}</td>
                  <td>{{ order.sugar }}</td>
                  <td>{{ order.toppings.join(",") }}</td>
                  <td>{{ order.price }}</td>
                  <td>{{ order.cup }}</td>
                  <td class="">{{ order.total }}</td>
                </tr>
              </tbody>
            </table>
            <p class="text-end">共 NT$ {{ orderTotal }} 元</p>
            <button
              @click="generateOrder(orderList, orderTotal)"
              :disabled="orderList.length === 0"
              class="btn btn-sm btn-primary w-100"
            >
              產生訂單
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="checkedOrder.orders.length > 0" class="bg-light p-3 mt-3">
    <div class="bg-white p-3 d-flex flex-column" style="min-height: 450px">
      <table class="table">
        <thead>
          <tr>
            <th scope="col">品項</th>
            <th scope="col">冰塊</th>
            <th scope="col">甜度</th>
            <th scope="col">加料</th>
            <th scope="col">單價</th>
            <th scope="col">數量</th>
            <th scope="col">小計</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="order in checkedOrder.orders" :key="order">
            <th scope="row">
              {{ order.name }}<br />
              <small class="text-muted fw-normal">備註：{{ order.note }}</small>
            </th>
            <td>{{ order.ice }}</td>
            <td>{{ order.sugar }}</td>
            <td>{{ order.toppings.join(",") }}</td>
            <td>{{ order.price }}</td>
            <td>{{ order.cup }}</td>
            <td class="">$ {{ order.total }} 元</td>
          </tr>
        </tbody>
      </table>
      <p class="mt-3 mb-1">訂單成立時間：{{ checkedOrder.time }}</p>
      <p class="mb-1">餐點數： {{ checkedOrder.orders.length }}</p>
      <p class="mb-3">付款狀態：未付款</p>
      <p>
        共 NT$ <span class="text-danger"> {{ checkedOrder.total }} </span> 元
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: "DrinkMenu",
  data() {
    return {
      tempSelected: {}, //暫存選擇飲料項目
      orderList: [], //訂單明細
      orderTotal: 0, //明細總價
      checkedOrder: {
        //產生訂單
        time: "",
        total: 0,
        orders: [],
      },
      iceType: ["正常冰", "少冰", "微冰", "去冰", "熱"],
      sugarType: ["全糖", "七分", "半糖", "三分", "無糖"],
      toppingsType: ["珍珠", "粉條", "小粉圓", "椰果", "芋頭"],
      products: [
        {
          name: "珍珠鮮奶茶",
          engName: "Pearl Milk Tea",
          price: 60,
          defaults: {
            toppings: ["珍珠"],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "椰果鮮奶茶",
          engName: "Coconut Milk Tea",
          price: 60,
          defaults: {
            toppings: ["椰果"],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "鮮奶茶",
          engName: "Milk Tea",
          price: 50,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "古意冬瓜茶 (糖固定)",
          engName: "Winter Melon Drink",
          price: 30,
          defaults: {
            toppings: [""],
            sugar: "全糖",
            ice: "",
          },
        },
        {
          name: "蜜香紅茶",
          engName: "Black Tea",
          price: 30,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "包種青茶",
          engName: "Black Tea",
          price: 35,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "檸檬烏龍",
          engName: "Lemon Oolong Tea",
          price: 55,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "薑母茶 (熱飲)",
          engName: "Ginger Tea",
          price: 55,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "熱",
          },
        },
        {
          name: "青草茶",
          engName: "Herbal Tea",
          price: 35,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "金桔檸檬",
          engName: "Kumquat Lemonade",
          price: 40,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
        {
          name: "柳澄青茶",
          engName: "Orange Mountain Tea",
          price: 45,
          defaults: {
            toppings: [""],
            sugar: "",
            ice: "",
          },
        },
      ],
    };
  },
  methods: {
    //選擇品項
    addSelect(product) {
      //設定預設
      this.tempSelected = {
        cup: 1,
        ice: product.defaults.ice !== "" ? product.defaults.ice : "正常冰",
        sugar: product.defaults.sugar !== "" ? product.defaults.sugar : "全糖",
        toppings: [],
        note: "",
        ...product,
      };
      console.log(this.tempSelected);
    },
    //初始化
    reset() {
      this.tempSelected = {};
    },
    //加入訂單
    addToOrder(product) {
      this.tempSelected = {
        ...product,
        total: (product.price + product.toppings.length * 10) * product.cup,
      };
      this.orderList.push(this.tempSelected);
      this.countTotal();
      this.reset();
    },
    //計算總價
    countTotal() {
      this.orderTotal = 0;
      this.orderList.forEach((item) => {
        this.orderTotal += item.total;
      });
    },
    // 產生訂單
    generateOrder(orderList, orderTotal) {
      this.checkedOrder.orders = orderList;
      this.checkedOrder.total = orderTotal;
      this.checkedOrder.time = new Date().toLocaleString();
      this.orderList = [];
      this.orderTotal = 0;
      this.reset();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss"></style>
