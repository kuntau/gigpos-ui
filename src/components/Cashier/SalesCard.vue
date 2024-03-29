<script setup lang="ts">
import { ref, computed } from 'vue';
import { storeToRefs } from 'pinia';
import CustomerSelection from '@/components/Cashier/CustomerSelection.vue'
import { useCart } from '../../stores/cart'
import type { Product } from '../../stores/products'
import type { CartItem } from '../../stores/cart'

const cartStore = useCart()
const customerSelectionPopup = ref(false)

interface CurrentSale extends Product {
  quantity: number,
  addon?: Product[]
}
  
/* const currentSale: CartItem[] = storeToRefs(cart.cart) */
const { cart, editCartMode } = storeToRefs(cartStore)
/* const cart = computed(() => cartStore.cart) */
const debug = true

const OneItem: Product = {
  id: 150,
  name: 'Kari Kambing',
  price: 10.0,
  categoryId: 3,
}

// methods
function formatPrice(price: number) {
  return price.toFixed(2).toLocaleString();
}

function formatPriceString(price: number) {
  return 'RM' + formatPrice(price);
}

function calcPerProductSum(items: CurrentSale) {
  let sum = items.price * items.quantity;
  if ('addon' in items && items.addon!.length > 0) {
    items.addon!.forEach((el: Product) => (sum += el.price));
  }
  return formatPrice(sum);
}

const clearCart = () => {
  console.log("Resetting cart...", cartStore.getSubTotal)
  /* cartStore.$patch({ cart: [] }) */
  /* cartStore.cart = [] */
  /* cart = [] */
  /* currentSale = [] */
  /* cart.$reset() */
  cartStore.clearCart()
  console.log("Resetting cart...", cartStore.getSubTotal)
}

// computed
const getSubTotalString = computed(() =>
  formatPriceString(cartStore.getSubTotal)
);

const getTotalString = computed(() =>
  formatPriceString(cartStore.getTotal)
);
</script>

<template>
  <div class="card flex flex-col bg-card-bg bg-white shadow-lg rounded relative" >
    <div class="card-header bg-white rounded-t border-b flex">
      <h3 class="text-lg text-gray-700 font-semibold tracking-wide py-4 mx-4">Cart</h3>
      <div
        class="flex-grow text-right text-lg text-gray-700 border-l p-4 cursor-pointer"
        @click="customerSelectionPopup = !customerSelectionPopup"
      >
        <!-- @mouseover="customerSelectionPopup = true" -->
        <!-- @mouseleave="customerSelectionPopup = false" -->
        <p>Customer345 <span class="ml-2 far fa-chevron-circle-down"></span></p>
        <CustomerSelection v-if=customerSelectionPopup />
      </div>
    </div>

    <div class="list-card-content flex-grow pt-3">

      <div class="flex-grow pl-3">
        <table class="card-list table w-full" >
          <tbody clsss="text-green-900" >
            <tr v-show="cart.length === 0">
              <td>Please add item to cart.</td>
            </tr>
            <tr
              v-for="(sale, index) in cart"
              :key="index"
            >
              <td v-show="editCartMode === true"><button @click="cartStore.removeCartItem(sale.id)"><i class="fa fa-times bg-red-400 text-white rounded p-1"></i></button></td>
              <td>
                {{ sale.name }}
                <div
                  v-for="(addon, addonIndex) in sale.addon"
                  :key="addonIndex"
                  class="text-gray-500 text-sm card-list-item-detail"
                >
                  {{ addon.name }} &mdash; {{ formatPriceString(addon.price) }}
                </div>
              </td>
              <td class="text-sm text-center align-top">
                <button v-show="editCartMode" @click="cartStore.decProductQuantity(sale.id)"><i class="fa fa-minus bg-amber-400 text-white rounded p-1"></i></button>
                {{ sale.quantity }}
                <button v-show="editCartMode" @click="cartStore.incProductQuantity(sale.id)"><i class="fa fa-plus bg-green-400 text-white rounded p-1"></i></button>
              </td>
              <td class="pr-2 text-right align-top">{{ calcPerProductSum(sale)  }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Start subtotal section -->
      <div class="card-footer bg-white mt-3 p-5 border-t">
        <p class="text-right text-gray-700 font-normal tracking-wide">
          Sub total : {{ getSubTotalString }}
        </p>
        <p class="text-right text-gray-700 font-normal tracking-wide">
          Tax : {{ cartStore.taxRate }}%
        </p>
        <p class="text-right text-gray-700 font-normal tracking-wide">
          Discount : {{ cartStore.discount }}%
        </p>
      </div>
      <!-- Start total section -->
      <div class="card-footer bg-gray-200 rounded-b p-3 border-t flex">
        <p
          class="text-xl text-right text-gray-700 font-bold tracking-wide flex-1"
        >
          Total: {{ getTotalString }}
        </p>
        <a class="text-xl px-2 ml-3" @click="clearCart"><i class="text-2xl text-green-500 hover:shadow-lg fad fa-cash-register"></i></a>
      </div>
      <!-- Start debug section -->
      <div v-show="debug" class="card-footer rounded-b p-3 border-t flex h-14 bg-fuchsia-100">
        <button @click="clearCart" class="px-3 ml-2 text-sm rounded text-white bg-red-500"><i class="fad fa-times"></i></button>
        <button @click="cartStore.$reset()" class="px-3 ml-2 text-sm rounded text-white bg-cyan-500"><i class="fad fa-allergies"></i></button>
        <button @click="editCartMode = !editCartMode" class="px-3 pr-2 ml-2 text-sm rounded text-white bg-blue-500"><i class="fad fa-edit"></i></button>
        <button @click="cartStore.addCartItem(OneItem as CartItem)" class="px-3 pr-2 ml-2 text-sm rounded text-white bg-green-500"><i class="fad fa-cart-plus"></i></button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.card {
  background-color: var(--card-bg-color);
}

.card-content {
  background-color: pink;
}

.card-footer:nth-child(odd) {
  background-color: var(--card-bg-color);
}

.card-list {
  margin-top: 0.2rem;
}

.card-list-quantity {
  background-color: var(--card-bg-color);
}

.card-list-delete {
  right: -10px;
}

.bg-card-bg {
  background-color: var(--card-bg-color);
}
</style>
