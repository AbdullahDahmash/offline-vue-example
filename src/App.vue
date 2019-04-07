<template>
    <div id="app">
        <div v-if="formSubmitted">
            <h1 class="submitted">Form submitted successfully! <small>{{formSubmittedText}} </small></h1>
        </div>
        <form @submit.prevent="validateBeforeSubmit">
            <div class="form-group" :class="{'has-error': errors.has('name') }">
                <label class="control-label" for="name">Name</label>
                <input v-model="name" v-validate.initial="name" data-rules="required|alpha|min:3" class="form-control" type="text" placeholder="Full Name">
                <p class="text-danger" v-if="errors.has('name')">{{ errors.first('name') }}</p>
            </div>
            <button class="btn btn-primary btn-block" type="submit">Submit</button>
        </form>
    </div>
</template>

<script>
import Vue from 'vue'
import VeeValidate from 'vee-validate';
import localforage from 'localforage';

localforage.config();

Vue.use(VeeValidate);

export default {
    data () {
        return {
            name: '',
            formSubmitted: false
        }
    },
    mounted() {

        localforage.getItem('form').then((data) => {
            if (data && data.form) {
                const { form } = data
                this.fillForm(form).then(() => {
                    this.validateBeforeSubmit();
                });
            } else {
                // load from server
                console.log('loading from server')
            }
        })

    },
    methods: {
        fillForm(form) {
            if (form) {
                console.log(form.name)
                this.name = form.name;
            }
        },
        validateBeforeSubmit(e) {
            this.$validator.validateAll();

            if (!this.errors.any()) {
                if (navigator.onLine) {
                    this.submitForm('online submission')
                } else {
                    const { name } = this;
                    const handlerId = setInterval(()=>{
                        console.log(`Online status: ${navigator.onLine}`)
                        if (navigator.onLine) {
                            localforage.getItem('form', (err, form) => {
                                this.submitForm('online submission after delay')
                                clearInterval(form.handler)
                                localforage.removeItem('form')
                            })
                        }
                    }, 1000);
                    localforage.setItem('form', {form: {name}, handler: handlerId})
                }
            }
        },
        submitForm(text){
            this.formSubmitted = true
            this.formSubmittedText = text
        }
    }
}

</script>

<style>
body {
    font-family: Helvetica, sans-serif;
}
.container {
    width: 500px;
}
h1 {
    text-align: center
}
img {
    text-align: center
}
.submitted {
    color: #4fc08d;
}
</style>
