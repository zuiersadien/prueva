<style>
    #sign-cryptocurrency {
        position: absolute;
        right: 15px;
        top: 16px;
        color: #d2c8ec;
    }
</style>

<template>
    <div style="height: 170px">
        <div class="exchange-box">
            <div class="selector">
                <p class="text">{{ label }}</p>
                <div class="coin">
                    <div class="language-select">
                        <v-select v-if="cryptoCurrencies.length > 1" class="select-bar" v-model="cryptoCurrencySelected" :options="cryptoCurrencies" :reduce="option => option.id" label="name" @input="emitCryptoCurrency"/>
                    </div>
                </div>
            </div>
            <form action="#" id="faq-form-2">
                <div class="form-group" style="position: relative">
                    <div id="sign-cryptocurrency">USDT</div>
                    <input onClick="this.setSelectionRange(0, this.value.length)" type="text" name="number" class="form-control" placeholder="24.30" maxlength="10" :value="cryptoAmount" v-on:keyup="emitCryptoAmount">
                </div>
            </form>
            <div class="row">
                <div class="d-flex justify-content-end mt-2 mb-4">
                    <div v-if="cryptoCurrencySelected === 'USDT'">
                        <small >Comisión: {{ fiatCharge.find(fiat => fiat.code === fiatCurrency).chargePercentage }}%</small>
                    </div>
                    <small v-if="cryptoCurrencySelected === 'BTC'">Comisión: 2% + USD $30</small>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'input-crypto',
    props: ['label', 'fiatCurrency', 'cryptoAmount', 'fiatCharge', 'cryptoCurrencies'],
    data: () => ({
        cryptoCurrencySelected: null
    }),
    mounted: function () {
        this.init()
    },
    methods: {
        init() {
            let defaultValue = this.cryptoCurrencies[0].id
            this.cryptoCurrencySelected = defaultValue
            this.emitCryptoCurrency(defaultValue)
        },
        emitCryptoCurrency(value) {
            this.$emit('cryptoCurrency', value)
        },
        emitCryptoAmount(e) {
            this.$emit('cryptoAmount', e.target.value)
        },
    }
}
</script>
