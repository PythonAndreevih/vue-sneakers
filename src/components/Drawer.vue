<script setup>
import axios from 'axios'
import { ref, watch, computed, inject } from 'vue'
import DrawerHead from './DrawerHead.vue'
import DrawerCardList from './DrawerCardList.vue'
import InfoBlock from './InfoBlock.vue'

const props = defineProps({
  totalPrice: Number,
  vatPrice: Number
})

const { cart, closeDrawer } = inject('cart')

const isCreatingOrder = ref([false])
const orderId = ref(null)

const createOrder = async () => {
  try {
    isCreatingOrder.value = true
    const { data } = await axios.post(`https://24b3aca65c11a90c.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })
    cart.value = []

    orderId.value = data.id
  } catch (err) {
    console.log(err)
  } finally {
    isCreatingOrder.value = false
  }
}

const cartIsEmpty = computed(() => cart.value.length === 0)
const ButtonDisabled = computed(() => isCreatingOrder.value || cartIsEmpty)
</script>

<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <InfoBlock
        v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавтьте пару кросовок хотя бы что бы сделать заказ"
        imageUrl="/package-icon.png"
      />
      <InfoBlock
        v-if="orderId"
        title="Заказ оформлен!"
        :description="`Ваш заказ # ${orderId} скоро будет передан курьерской доставке`"
        imageUrl="/order-success-icon.png"
      />
    </div>

    <div v-else>
      <DrawerCardList />

      <div class="flex flex-col gap-4 mt-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} Р</b>
        </div>

        <div class="flex gap-2">
          <span>Налог 5%: </span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ vatPrice }} Р</b>
        </div>
        <button
          :disabled="totalPrice ? false : true"
          @click="createOrder"
          class="mt-4 transition bg-lime-500 w-full rounded-xl py-3 text-white disabled:bg-slate-400 hover:bg-lime-600 active:bg-lime-700 cursor-pointer"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>
