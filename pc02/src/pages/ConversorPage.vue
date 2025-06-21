<template>
  <q-card class="q-pa-lg" style="max-width: 500px; margin: auto;">
    <q-card-section>
      <div class="text-h5 q-mb-md text-center" style="margin-bottom: 30px;">
        <q-icon name="trending_up" class="q-mr-sm" color="secondary" />
        Conversión de Monedas
      </div>
      <q-form @submit.prevent="convierte" class="q-gutter-md">
        <q-input
          v-model="form.monto"
          type="number"
          :min="1"
          label="Monto"
          filled
          :rules="[val => !!val || 'El monto es requerido']"
          autofocus
          @change="showConversion = false"
        />
        <div class="selectores-moneda q-gutter-sm">
          <q-select
            filled
            v-model="form.desde"
            :options="listaMonedas"
            option-value="id"
            option-label="label"
            label="Desde"
            filled-class="q-mb-md"
            :rules="[val => !!val || 'La moneda origen es requerida']"
            @update:model-value="showConversion = false"
            class="select-field"
          />
          <q-select
            filled
            v-model="form.hacia"
            :options="listaMonedas"
            option-value="id"
            option-label="label"
            label="Hacia"
            filled-class="q-mb-md"
            :rules="[val => !!val || 'La moneda destino es requerida']"
            @update:model-value="showConversion = false"
            class="select-field"
          />
        </div>
        <div class="row justify-center q-mb-md">
          <q-btn
            round
            dense
            color="primary"
            icon="swap_vert"
            @click="intercambiarMonedas"
            aria-label="Intercambiar monedas"
          />
        </div>
        <q-btn
          type="submit"
          color="primary"
          label="Convertir"
          class="full-width q-mt-md"
        />
      </q-form>
      <q-banner v-if="showConversion" class="q-mt-md text-center">
        <div class="text-h6">
          {{ form.monto }} {{ form.desde.id }} equivalen a {{ convertido.rate }} {{ form.hacia.id }}
        </div>
      </q-banner>
    </q-card-section>
  </q-card>
</template>

<style>
.selectores-moneda {
  display: flex;
  justify-content: space-between;
}
.select-field {
  width: calc(50% - 10px);
}
</style>

<script>
export default {
  name: 'ConversorPage',
  data() {
    return {
      form: {
        monto: '',
        desde: '',
        hacia: ''
      },
      listaMonedas: [],
      showConversion: false,
      convertido: {
        amount: 0,
        base: '',
        date: '',
        rate: 0
      }
    };
  },
  mounted() {
    this.loadMonedas();
  },
  methods: {
    loadMonedas() {
      let endpointURL = "/currencies";
      this.$api.get(endpointURL)
        .then(response => {
          console.log('Monedas cargadas:', response.data);
          this.listaMonedas = Object.entries(response.data).map(([id, label]) => ({
            id,
            label: `${id} - ${label}`
          }));
        })
        .catch(error => {
          console.error('Error al cargar las monedas:', error);
        });
    },
    intercambiarMonedas() {
      const temp = this.form.desde;
      this.form.desde = this.form.hacia;
      this.form.hacia = temp;
      this.showConversion = false;
    },
    convierte() {
      console.log('Formulario enviado:', this.form);

      if(this.form.desde.id == this.form.hacia.id) {
        this.$q.notify({
          type: 'negative',
          message: '¡Ha seleccionado la misma moneda!'
        });
        return;
      }

      let endpointURL = "/latest";
      this.$api.get(endpointURL, {
          params: {
            amount: this.form.monto,
            from: this.form.desde.id,
            to: this.form.hacia.id
          }
        })
        .then(response => {
          console.log('Monedas cargadas:', response.data);
          this.convertido = response.data;
          this.convertido.rate = response.data.rates[this.form.hacia.id];
          this.showConversion = true;
        })
        .catch(error => {
          console.error('Error al cargar las monedas:', error);
        });
    }
  }
};
</script>
