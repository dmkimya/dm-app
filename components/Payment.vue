<template>
  <section class="mt-3">
    <b-tabs v-model="activeTab">
      <b-tab-item label="TAHSİLAT">
        <b-field class="mt-3" label="Ödeme Miktarı">
          <div style="display: flex">
            <b-input
              placeholder="Ödeme miktarı"
              type="number"
              min="0"
              :max="maxpayamount"
              icon="calculator"
              :validation-message="errormessage"
              v-model="payment_amount"
              style="max-width: 15rem; min-width: 12rem"
            >
            </b-input>

            <p class="control">
              <b-button
                @click="getpaid"
                :disabled="!paybtn"
                type="is-success"
                label="Ödeme Al"
              />
            </p>
          </div>
        </b-field>
        <b-field>
          <b-checkbox v-model="isCardUsed" type="is-success">
            Kredi Kartı
          </b-checkbox>
        </b-field>
      </b-tab-item>

      <b-tab-item label="SATIŞ">
        <section class="mt-5 mb-5" style="">
          <b-field label="Ürün ">
            <v-select
              style="width:15rem"
              v-model="productToAdd.product"
              label="name"
              :options="products"
            ></v-select>
          </b-field>
          <b-field label="Adet">
            <b-numberinput
              type="number"
              min="0"
              v-model="productToAdd.count"
              :controls="false"
              validation-message="Lütfen doğru formatta numara girin"
              style="max-width: 30rem; width: 14rem"
            >
            </b-numberinput>
          </b-field>
          <b-field label="Fiyat">
            <b-numberinput
              :controls="false"
              min="0"
              step="0.01"
              v-model="productToAdd.price"
              style="max-width: 30rem; width: 14rem"
            ></b-numberinput>
            <b-button @click="add" type="is-success" label="Ekle" />
          </b-field>
        </section>

        <section v-show="data.length > 0" style="max-width: 30rem">
          <b-field grouped group-multiline>
            <b-button
              label="Temizle"
              type="is-danger"
              icon-left="close"
              class="field"
              @click="data = []"
            />
          </b-field>

          <b-table :data="data" field="" :mobile-cards="false">
            <b-table-column field="product.name" label="Ürün" v-slot="props">
              {{ props.row.product.name }}
            </b-table-column>
            <b-table-column field="count" label="Adet" v-slot="props">
              {{ props.row.count }}
            </b-table-column>
            <b-table-column field="price" label="Fiyat" v-slot="props">
              {{ props.row.price }}
            </b-table-column>
            <b-table-column field="total" label="Ara toplam" v-slot="props">
              {{ props.row.total }}
            </b-table-column>
            <b-table-column v-slot="props">
              <b-button
                style="height:1.5rem"
                class="m-0 p-2"
                @click="data.splice(props.row.index, 1)"
              >
                <b-icon style="color:red;cursor:pointer" icon="close"></b-icon>
              </b-button>
            </b-table-column>
          </b-table>
          <b-field class="mt-5"
            ><span> <b>Toplam</b>: {{ totalTablePrice }}</span></b-field
          >
        </section>

        <b-field class="mt-5" label="Satış Miktarı">
          <div style="display: flex">
            <b-input
              placeholder="Satış miktarı"
              type="number"
              min="0"
              icon="calculator"
              disabled
              validation-message="Lütfen doğru formatta numara girin"
              v-model="sales_amount"
              style="max-width: 15rem"
            >
            </b-input>

            <p class="control">
              <b-button
                @click="sell"
                :disabled="!salebtn"
                type="is-success"
                label="Satış Yap"
              />
            </p>
          </div>
        </b-field>
      </b-tab-item>
    </b-tabs>
  </section>
</template>
<script>
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";
export default {
  components: {
    vSelect
  },
  data() {
    return {
      isCardUsed: false,
      activeTab: 0,
      payment_amount: null,
      sales_amount: null,
      paybtn: false,
      salebtn: false,

      errormessage: "Lütfen doğru formatta numara girin",
      totalTablePrice: 0,
      productToAdd: {
        product: { id: "", name: "" },
        count: null,
        price: null,
        total: null
      },
      data: []
    };
  },
  props: {
    customer: {
      type: Object,
      required: true
    }
  },
  computed: {
    totPrice() {
      return this.totalTablePrice;
    },
    maxpayamount() {
      return this.customer.current_balance;
    },
    products() {
      return this.$store.getters["product/GET_PRODUCTS"];
    }
  },
  watch: {
    data() {
      let total = 0;
      this.data.map(el => (total += parseFloat(el.total)));
      this.totalTablePrice = total;

      this.totalTablePrice === 0
        ? (this.sales_amount = null)
        : (this.sales_amount = this.totalTablePrice);
    },
    payment_amount() {
      if (
        this.payment_amount === (null || "") ||
        parseInt(this.payment_amount) > parseInt(this.maxpayamount) ||
        this.payment_amount <= 0
      ) {
        this.paybtn = false;
      } else {
        this.paybtn = true;
      }

      if (parseInt(this.payment_amount) > parseInt(this.maxpayamount)) {
        this.errormessage = "Ödeme miktarı bakiyeden büyük olmaz";
      } else {
        this.errormessage = "Lütfen doğru formatta numara girin ";
      }
    },
    sales_amount() {
      if (this.sales_amount !== (null || "")) {
        this.salebtn = true;
      } else {
        this.salebtn = false;
      }
    }
  },
  methods: {
    add() {
      this.productToAdd.total =
        parseInt(this.productToAdd.count) * parseFloat(this.productToAdd.price);

      if (
        this.productToAdd.count != (null || "" || 0) &&
        this.productToAdd.price != (null || "" || 0) &&
        this.productToAdd.total != (null || "" || 0) &&
        this.productToAdd.product.id != "" &&
        this.productToAdd.product.name != ""
      ) {
        this.data.push(this.productToAdd);

        this.productToAdd = {
          product: { id: "", name: "" },
          count: null,
          price: null,
          total: null
        };
      }
    },
    getpaid() {
      this.$buefy.dialog.confirm({
        title: "Uyarı Mesajı",
        message: `
                    ${this.payment_amount} TL Tutarındaki <b>${
          this.isCardUsed ? "KREDİ KARTI" : "NAKİT"
        }</b> tahsilatı onaylıyor musun?
                    `,
        cancelText: "İptal",
        confirmText: "Onaylıyorum",
        type: "is-success",
        onConfirm: async () => {
          const loadingComponent = this.$buefy.loading.open();

          try {
            await this.$store.dispatch("customer/getpaid", {
              id: this.customer.id,
              payment_amount: this.payment_amount,
              balance: this.customer.current_balance,
              company_name: this.customer.company_name,
              card: this.isCardUsed
            });
            loadingComponent.close();
            this.$buefy.toast.open({
              message: "İşlem başarıyla tamamlandı",
              type: "is-success"
            });
          } catch (error) {
            loadingComponent.close();
            this.$buefy.dialog.alert({
              title: "HATA!!!",
              message: error.msg,
              type: "is-danger",
              hasIcon: false,
              icon: "exclamation",
              iconPack: "fa",
              ariaRole: "alertdialog",
              ariaModal: true
            });
          }

          this.payment_amount = "";
        }
      });
    },
    sell() {
      this.$buefy.dialog.confirm({
        title: "Uyarı Mesajı",
        message: `
                    ${this.sales_amount} TL Tutarındaki satışı onaylıyor musun?
                    `,
        cancelText: "İptal",
        confirmText: "Onaylıyorum",
        type: "is-success",
        onConfirm: async () => {
          const loadingComponent = this.$buefy.loading.open();

          try {
            await this.$store.dispatch("product/decreaseProduct", {
              products: this.data,
              company: this.customer.company_name
            });

            await this.$store.dispatch("customer/sell", {
              id: this.customer.id,
              sales_amount: this.sales_amount,
              balance: this.customer.current_balance,
              final_shopping: this.data,
              company_name: this.customer.company_name,
              card: this.isCardUsed
            });

            loadingComponent.close();
            this.$buefy.toast.open({
              message: "İşlem başarıyla tamamlandı",
              type: "is-success"
            });
            this.data = [];
            this.sales_amount = "";
          } catch (error) {
            loadingComponent.close();
            this.$buefy.dialog.alert({
              title: error.title || "HATA!!!",
              message: error.msg || "BİRŞEYLER TERS GİTTİ",
              type: "is-danger",
              hasIcon: false,
              icon: "exclamation",
              iconPack: "fa",
              ariaRole: "alertdialog",
              ariaModal: true
            });
          }
        }
      });
    }
  }
};
</script>
