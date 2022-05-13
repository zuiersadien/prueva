<template>
    <div>
        <div class="exchange">
            <div class="timerlive">
                <small>{{ currentDate }} {{ time }}</small>
            </div>
            <h5 class="ex-head">fiTs<br/>Exchange</h5>
            <div v-if="status.calculator === 'loading'">
                <SkeletonTheme color="#DDD9F2" highlight="#f1ebff">
                    <div class="exchange-box" style="height: 170px">
                        <PuSkeleton width="80px" height="28px"/>
                        <PuSkeleton width="280px" height="58px"/>
                        <PuSkeleton width="170px" height="20px"/>
                    </div>
                    <div class="exchange-box" style="height: 170px">
                        <PuSkeleton width="80px" height="28px"/>
                        <PuSkeleton width="280px" height="58px"/>
                        <div class="d-flex justify-content-end">
                            <PuSkeleton width="70px" height="20px"/>
                        </div>
                    </div>
                    <div class="">
                        <PuSkeleton width="280px" height="50px"/>
                    </div>
                </SkeletonTheme>
            </div>
            <div class="d-flex align-items-center" v-else-if="status.calculator === 'error'" style="height: 340px">
                <div class="text-center w-100">
                    Volver a cargar <a v-on:click="getParams"><i class="fas fa-redo-alt"></i></a>
                </div>
            </div>
            <div v-else-if="status.calculator === 'success'">
                <span v-if="orderInputs">
                    <InputFiat label="Tu envías"
                               :tcPenUsd="params.tc.penusd"
                               :currencies="params.fiatCurrenciesAvailable"
                               v-on:fiatCurrency="setFiatCurrency"
                               :fiatAmount="fiatAmount"
                               v-on:fiatAmount="setFiatAmount"/>
                </span>

                <div v-if="!orderInputs" class="d-flex justify-content-center my-2">
                    <button v-on:click="changeOrderInput" class="rotate btn btn-link" href="#">
                        <img src="/assets_home/img/exchange-img.png" alt="">
                    </button>
                </div>

                <span v-if="orderInputs">
                    <InputCrypto label="Recibes"
                                 :fiatCharge="params.fiatCharge"
                                 :cryptoCharge="params.cryptoCharge"
                                 :fiatCurrency="fiatCurrency"
                                 :cryptoCurrencies="params.cryptoCurrenciesAvailable"
                                 v-on:cryptoCurrency="setCryptoCurrency"
                                 :cryptoAmount="cryptoAmount"
                                 v-on:cryptoAmount="setCryptoAmount"/>
                </span>
            </div>

            <a v-if="status.calculator === 'success'" href="/register" class="button button-1">Comprar</a>
        </div>
    </div>
</template>

<script>
import InputFiat from './InputFiat.vue'
import InputCrypto from './InputCrypto.vue'
import { SkeletonTheme } from 'vue-loading-skeleton';

export default {
    name: "CalculatorFiatExchange",
    props: [],
    components: {
        InputFiat, InputCrypto, SkeletonTheme
    },
    data:function() {
        return {
            status: {
                calculator: 'loading'
            },
            orderInputs: true,
            fiatAmount: 0,
            fiatCurrency: 'PEN',
            cryptoAmount: 0,
            cryptoCurrency: 'USDT',
            lastInputEdited: null,
            params: {},
            currentDate: null,
            interval: null,
            time: null
        }
    },
    computed: {

    },
    watch: {
        fiatAmount: function(value) {
            if (isNaN(value)) {
                this.fiatAmount = 0
                this.cryptoAmount = 0
            }
        },
        cryptoAmount: function(value) {
            if (isNaN(value)) {
                this.fiatAmount = 0
                this.cryptoAmount = 0
            }
        }
    },
    mounted () {
        this.init()
    },
    methods: {
        init() {
            this.getParams()
            this.loadTimer()
        },
        changeOrderInput() {
            this.orderInputs = !this.orderInputs
        },
        setFiatAmount(val) {
            let amount = parseFloat(val)
            this.fiatAmount = amount
            this.updateCryptoAmountFromFiat(amount)
            this.lastInputEdited = 'setFiatAmount'
        },
        setFiatCurrency(val) {
            this.fiatCurrency = val
            if (this.lastInputEdited === 'setFiatAmount') {
                this.updateCryptoAmountFromFiat()
            }
            if (this.lastInputEdited === 'setCryptoAmount') {
                this.updateFiatAmountFromCrypto()
            }
        },
        setCryptoAmount(val) {
            this.cryptoAmount = parseFloat(val)
            this.updateFiatAmountFromCrypto()
            this.lastInputEdited = 'setCryptoAmount'
        },
        setCryptoCurrency(val) {
            this.cryptoCurrency = val
            this.updateCryptoAmountFromFiat()
        },
        updateCryptoAmountFromFiat(fiatAmount = 0) {
            let totalUSDT = 0
            let amount = fiatAmount === 0 ? this.fiatAmount : fiatAmount
            let fiatCharge = this.params.fiatCharge.find(fiat => fiat.code === this.fiatCurrency)
            let charge = 1 + (fiatCharge.chargePercentage / 100)

            switch (this.fiatCurrency) {
                case 'PEN':
                    totalUSDT = amount / this.params.tc.penusd / charge
                    break

                case 'USD':
                    totalUSDT = amount / charge
                    break

                default:
                    break
            }

            let cryptoCharge = this.params.cryptoCharge.find(crypto => crypto.code === this.cryptoCurrency)

            this.cryptoAmount = (totalUSDT - cryptoCharge.charge).toFixed(1) + '0'
        },
        updateFiatAmountFromCrypto(cryptoAmount = 0) {
            let totalUSD = 0
            let amount = cryptoAmount === 0 ? this.cryptoAmount : cryptoAmount
            let fiatCharge = this.params.fiatCharge.find(fiat => fiat.code === this.fiatCurrency)
            let charge = 1 + (fiatCharge.chargePercentage / 100)

            switch (this.fiatCurrency) {
                case 'PEN':
                    totalUSD = amount * this.params.tc.penusd * charge
                    break

                case 'USD':
                    totalUSD = amount * charge
                    break

                default:
                    break
            }

            this.fiatAmount = totalUSD.toFixed(1) + '0'
        },
        loadTimer() {
            const today = new Date();
            const monthNames = ["Enero", "Febrero", "Marzp", "Abril", "Mayo", "Junio",
                "Julio", "Agoto", "Setiembre", "Octubre", "Nov", "Diciembre"
            ];
            let weekdays = ['Domingo', 'Lunes', 'Martes', 'Miercoles', 'Jueves', 'Viernes', 'Sábado']

            this.currentDate = weekdays[today.getDay()] + ', ' + today.getDate() + ' ' + monthNames[(today.getMonth())];
            this.interval = setInterval(() => {
                this.time = Intl.DateTimeFormat(navigator.language, {
                    hour: 'numeric',
                    minute: 'numeric',
                    second: 'numeric'
                }).format()
            }, 1000)
        },
        getParams() {
            this.status.calculator = 'loading'
            this.$axios.get(this.$apiURL + '/params')
                .then(response => {
                    this.status.calculator = 'success'
                    this.params = response.data
                })
                .catch((error) => {
                    // setTimeout(() => this.status.calculator = 'error', 3000);
                    this.status.calculator = 'error'
                    if (!error.response) {
                        this.errorStatus = 'Error: Network Error';
                    } else {
                        this.errorStatus = error.response.data.message;
                    }
                })
        }
    },
};
</script>


