<template>
  <div>
    <h3>Payment</h3>
    <form novalidate @submit.prevent="onSave">
      <div class="row">
        <div class="col-md-6">
          <div><strong>Shipping Information</strong></div>
          <Address-View :address="model.shipping">
            <div class="form-group">
              <label for="name">Name</label>
              <input type="text" id="name" class="form-control" placeholder="Your Name" v-model="model.shipping.fullName.$model" />
            </div>
            <div class="form-group">
              <label for="company">Company Name</label>
              <input type="text" id="company" class="form-control" placeholder="Your Company"  v-model="model.shipping.company.$model" />
            </div>
          </Address-View>
          <div class="form-row">
            <div class="form-group">
              <input type="submit" value="Next" class="btn btn-success"/>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div><strong>Billing Information</strong></div>
          <Address-View :address="model.billing" :isDisabled="model.billing.sameAsShipping.$model">
            <div class="form-check">
              <input type="checkbox" id="sameAsShipping" class="form-check-input" v-model="model.billing.sameAsShipping.$model" />
              <label for="sameAsShipping" class="form-check-label">Same As Shipping?</label>
            </div>
          </Address-View>
          <div><strong>Credit Card</strong></div>
          <div class="form-group">
            <label for="ccNumber">Credit Card Number</label>
            <input class="form-control" type="text" v-model="model.creditcard.number.$model" id="ccNumber" />
            <ValidationMessage :model="model.creditcard.number" />
          </div>
          <div class="form-group">
            <label for="nameOnCard">Name on Card</label>
            <input class="form-control" type="text" v-model="model.creditcard.nameOnCard.$model" id="nameOnCard" />
            <ValidationMessage :model="model.creditcard.nameOnCard" />
          </div>
          <div class="form-row">
            <div class="form-group col-md-4">
              <label for="expirationMonth">Expiration Month</label>
              <select v-model="model.creditcard.expirationMonth.$model" class="form-control">
                <option v-for="m in months" :key="m.number" :value="m.number">
                  {{ m.name }}
                </option>
              </select>
              <ValidationMessage :model="model.creditcard.expirationMonth" />
            </div>
            <div class="form-group col-md-4">
              <label for="expirationYear">Expiration Year</label>
              <select v-model="model.creditcard.expirationYear.$model" class="form-control">
                <option v-for="year in years" :key="year" :value="year">{{ year }}</option>
              </select>
              <ValidationMessage :model="model.creditcard.expirationYear" />
            </div>
            <div class="form-group col-md-4">
              <label for="cvv">CVV Code</label>
              <input  v-model="model.creditcard.cvv.$model" class="form-control" id="cvv" />
              <ValidationMessage :model="model.creditcard.cvv" />
            </div>
          </div>
        </div>
      </div>
    </form>
    <div>
      <pre>{{ payment }}</pre>
      <pre>{{ model }}</pre>
    </div>
  </div>
</template>
<script>
import { ref, computed, watch, reactive } from "vue";
import states from "@/lookup/states";
import formatters from "@/formatters";
import months from "@/lookup/months";
import AddressView from "./AddressView";
import state from "@/state";
import { required } from "@vuelidate/validators";
import useVuelidate from '@vuelidate/core';
import ValidationMessage from "../components/ValidationMessage.vue";
import { creditcard } from "@/validators";

export default {
  components: { AddressView, ValidationMessage },
  emits: [ "onError" ],
  setup(props, { emit }) {  
    const payment = reactive(state);

    async function onSave() {
      if (await model.value.$validate()) {
        state.errorMessage.value = "We can't save yet, we don't have an API";
      }  
    }

    const zipCode = computed({
      get: () => payment.postalCode,
      set: (val) => {
        if (val && typeof val === "string") {
          if (val.length <= 5 || val.indexOf("-") > -1) {
            payment.postalCode = val;
          } else {
            payment.postalCode = `${val.substring(0, 5)}-${val.substring(5)}`;
          }
        }
      },
    });

    watch(
      // What to watch
      () => payment.billing.sameAsShipping,
      // What to do
      () => {
        if (payment.billing.sameAsShipping.value) {
          payment.clear();
        }
      }
    );

    const years = Array.from({ length: 10 }, (_, i) => i + 2020);
    const rules = {
       number : { required, creditcard },
       nameOnCard : { required },
       expirationMonth : { required },
       expirationYear : { required },
       cvv : { required }
    };
    
    const model = state.toModel();

    return {
      payment,
      states,
      onSave,
      ...formatters,
      zipCode,
      months,
      years,
      model
    };
  }
}
</script>