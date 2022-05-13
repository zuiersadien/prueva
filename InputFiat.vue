<template>
    <div style="height: 170px">
        <div class="exchange-box">
            <div class="selector">
                <p class="text">{{ label }}</p>
                <div class="coin">
                    <div class="language-select">
                        <button v-on:click="emitFiatCurrency('PEN')" class="btn btn-sm" :class="classButtonPEN">Soles</button>
                        <button v-on:click="emitFiatCurrency('USD')" class="btn btn-sm" :class="classButtonUSD">Dólares</button>
                    </div>
                </div>
            </div>
            <div class="form-group">
                <input onClick="this.setSelectionRange(0, this.value.length)" type="text" id="phone-mask" name="number" placeholder="1000" class="form-control" :value="fiatAmount" v-on:keyup="emitFiatAmount">
            </div>
            <small v-if="currencySelected === 'PEN'">Tipo de cambio: S/ {{ tcPenUsd.toFixed(3) }}</small>
        </div>
    </div>
</template>

<script>
export default {
    name: 'input-feat',
    props: ['label', 'currencies', 'fiatAmount', 'tcPenUsd'],
    data: () => ({
        currencySelected: null,
    }),
    computed: {
        classButtonPEN: function() {
            return {
                'btn-primary': this.currencySelected === 'PEN',
            }
        },
        classButtonUSD: function() {
            return {
                'btn-primary': this.currencySelected === 'USD',
            }
        }
    },
    mounted: function () {
        this.init()
        IMask(document.getElementById('phone-mask'), {
            mask: Number,
            decimal : ".",
            min: 0,
            max: 9999999,
        });
    },
    methods: {
        init() {
            let defaultValue = this.currencies[0].id
            this.currencySelected = defaultValue
            this.emitFiatCurrency(defaultValue)
        },
        emitFiatCurrency(value) {
            this.currencySelected = value
            this.$emit('fiatCurrency', value)
        },
        emitFiatAmount(e) {
            this.$emit('fiatAmount', e.target.value)
        },
    }
}
</script>
