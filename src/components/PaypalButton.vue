<template>
  <div id="paypal-checkout-button"></div>
</template>

<script lang="ts">
import 'reflect-metadata';
import decamelize from 'decamelize';
import { Component, Prop, Vue, Emit } from 'vue-property-decorator';
import { PropType } from 'vue';

enum ShippingPreference {
    NoShipping = 'NO_SHIPPING',
    GetFromFile = 'GET_FROM_FILE',
    UseProvidedAddress = 'USE_PROVIDED_ADDRESS'
}

interface PaypalButtonProps {
    // readonly amount: string | number;
    // readonly shippingPreference: ShippingPreference.NoShipping | ShippingPreference.GetFromFile | ShippingPreference.UseProvidedAddress; 
    // readonly sdkParams: PaypalSDKParams;
    createOrder: Function;
    onSuccess: Function;
    onApprove: Function;
    onClick: Function;
    onInit: Function;
}

enum Intent {
    Capture = 'capture',
    Authorize = 'authorize'
}

enum Funding {
    Card = 'card',
    Credit = 'credit',
    Bancontact = 'bancontact'
}

export type PaypalSDKParams = {
    readonly clientId: string;
    readonly test: boolean;
    readonly merchantId?: string;
    readonly currency?: string;
    readonly intent?: Intent;
    readonly commit?: string;
    readonly vault?: string;
    readonly components?: string;
    readonly disableFunding?: Funding;
    readonly disableCard?: string;
    readonly integrationDate?: string;
    readonly debug?: 'true' | 'false';
    readonly buyerCountry?: string;
    readonly locale?: string;
}

const PAYPAL_SDK_ASSET = 'https://www.paypal.com/sdk/js';

@Component
export default class PaypalButton extends Vue implements PaypalButtonProps {
    @Prop({ 
        required: true, 
        type: [String, Number] 
    }) readonly amount!: string | number;
    @Prop({ 
        default: ShippingPreference.NoShipping, 
        type: String,
        validator: (prop) => [
            ShippingPreference.NoShipping, 
            ShippingPreference.GetFromFile, 
            ShippingPreference.UseProvidedAddress,
        ].indexOf(prop) > -1, 
    }) readonly shippingPreference!: ShippingPreference.NoShipping | ShippingPreference.GetFromFile;
    @Prop({ 
        required: true, 
        type: Object as PropType<PaypalSDKParams>, 
    }) readonly sdkParams!: PaypalSDKParams;

    /*@Prop({ required: true }) readonly clientId!: string;
    @Prop(String) readonly merchantId?: string;
    @Prop({ default: 'USD' }) readonly currency?: string;
    @Prop(String) readonly intent?: 'capture' | 'authorize';
    @Prop(String) readonly commit?: string;
    @Prop(String) readonly vault?: string;
    @Prop(String) readonly components?: string;*/

    private ready = false;

    created () {
        if (!this.$props.sdkParams.currency) {
            this.$props.sdkParams.currency = 'USD';
        }
    }

    async mounted () {
        // Initialize SDK if not loaded
        if (typeof window !== 'undefined' && !Object.hasOwnProperty.call(window, 'paypal')) {
            let sdkConfigurationParams = `${PAYPAL_SDK_ASSET}?`;
            
            for (const [key, value] of Object.entries(this.$props.sdkParams)) {
              sdkConfigurationParams += `${decamelize(key, '-')}=${value}&`;
            }
            // Slice off trailing ampersand
            sdkConfigurationParams = sdkConfigurationParams.slice(0, sdkConfigurationParams.length - 1);
            console.log(sdkConfigurationParams);
            console.log('Loading PayPal SDK');

            try {
                await Vue.prototype.$loadScript(sdkConfigurationParams);
            } catch (err) {
                alert('Failed to load PayPal SDK');
                this.sdkReady(false);
                console.error(err);
                return;
            }
        } 
        // Render buttons if paypal exists on window
        if (Object.hasOwnProperty.call(window, 'paypal')) {
            console.log('Rendering buttons');
            window.paypal.Buttons({
                createOrder: this.createOrder,
                onApprove: this.onApprove,
                onSuccess: this.onSuccess,
                onError: this.onError,
                onInit: this.onInit,
                onClick: this.onClick,
            }).render(this.$el);
            this.sdkReady(true);
        }
    }

    @Emit()
    private sdkReady (ready: boolean) {
        this.ready = ready;
    }

    @Emit()
    public async onError(err: Error) {
        console.error(err);
    }

    @Emit()
    public async onSuccess (data: any, actions: any) {
        console.log('onSuccess');
    }

    @Emit()
    async onApprove (data: any, actions: any) {
        console.log('onApprove')
        let details = null;
        try {
            details = await actions.order.capture()
        } catch (err) {
            console.error(err);
        }
        return details;
    }

    @Emit()
    async onClick () {
        console.log('onClick');
    }

    @Emit()
    public async onInit (data: any, actions: any) {
        console.log('onInit');
    }

    public createOrder (data: any, actions: any) {
        console.log('createOrder')
        return actions.order.create({
            'purchase_units': [{
                'amount': {
                    'currency_code': this.$props.sdkParams.currency,
                    'value': this.$props.amount.toString()
                }
            }],
            'application_context': {
                'shipping_preference': this.$props.shippingPreference
            }
        });
    }
}
</script>
