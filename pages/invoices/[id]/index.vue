<template>
  <div v-if="invoice" class="invoice">
    <div>
      <div>Numer Faktury:{{ invoice.id }}</div>
      <div>Data: {{ getConvertDate() }}</div>
      <div>Data załaty:{{ getConvertDate() }}</div>
    </div>
    <div>
      <div>Dane sprzedawcy:</div>
      <div>Nazwa firmy:{{ invoice.company.name }}</div>
      <div>Adres firmy:{{ invoice.company.address }}</div>
      <div>NIP firmy:{{ invoice.company.nip }}</div>
    </div>
    <div>
      <div>Dane kupującego:</div>
      <div>Nazwa firmy:{{ invoice.customer.name }}</div>
      <div>Adres firmy:{{ invoice.customer.address }}</div>
      <div>NIP firmy:{{ invoice.customer.nip }}</div>
    </div>
    <div>
      <div>Towary:</div>
      <div v-for="item in invoice.items">
        <div>Nazwa: {{ item.name }}</div>
        <div>Ilość przedmiotów: {{ item.quantity }}</div>
        <div>Cena Netto: {{ item.price }} zł</div>
        <div>Wartość netto:{{ item.price * item.quantity }} zł</div>
        <div>Stawka VAT: {{ item.vat }}%</div>
        <div>
          Kwota VAT:
          {{ Math.round(item.price - (item.price * item.vat) / 100) }} zł
        </div>
        <div>
          Cena Brutto: {{ item.price + (item.price * item.vat) / 100 }} zł
        </div>
        <br />
      </div>
      <div>Cena wszystkich przedmiotów: {{ totalPrice }}</div>
    </div>
  </div>
  <div v-else>No invoice found</div>
</template>

<script lang="ts" setup>
import type { InvoicesBody } from "@/server/api/invoices";
import { getDateFormat } from "@/functions/date";

const { id } = useRoute().params;

const { data: invoice } = await useFetch<InvoicesBody>(`/api/invoices/${id}`);
const getConvertDate = () => {
  if (!invoice || !invoice.value) return "";

  return getDateFormat(invoice.value.date);
};
const getConvertPaymentData = () => {
  if (!invoice || !invoice.value) return "";

  return getDateFormat(invoice.value.payment_date);
};

const totalPrice = computed(() => {
  if (!invoice || !invoice.value) return 0;

  return invoice.value.items.reduce((acc, item) => {
    return acc + Math.round(item.price + (item.price * item.vat) / 100);
  }, 0);
});

computed(() => [getConvertDate]);
computed(() => [getConvertPaymentData]);
computed(() => [totalPrice]);
</script>

<style scoped>
.invoice {
  display: flex;
  gap: 50px;
}
</style>
