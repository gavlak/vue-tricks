<template>
    <form @submit.prevent="create" class="component__form">
        <h2>Submit new component</h2>

        <div class="form__group">
            <label for="name">Repository</label>
            <input type="text" v-model="suggestion.repository">
            <small class="form__helper">Git repository of the project</small>
        </div>

        <vue-recaptcha @verify="onVerify" ref="recaptcha" sitekey="6LdbDz8UAAAAAC82uobt8M3gH7GsndvKGsKs0LWO"></vue-recaptcha>

        <div class="form__submit">
            <button type="submit" class="btn btn--primary" :disabled="loading">Send →</button>
            <span class="loader loader--inline" v-if="loading"></span>
        </div>
    </form>
</template>

<script>
import { isWebUri } from 'valid-url'
import { api } from '../../api'
import VueRecaptcha from 'vue-recaptcha'

export default {
    components: {
        VueRecaptcha
    },
    data() {
        return {
            suggestion: {
                repository: "",
            },
            recaptcha: "",
            verified: false,
            loading: false
        }
    },

    methods: {
        create() {
            if (this.recaptcha === "" || this.verified === false)
            {
                this.$notify({
                    group: 'main',
                    title: 'Are you a robot? 🤖',
                    type: 'danger'
                })
                return;
            }

            if (this.suggestion.repository === "")
            {
                this.$notify({
                    group: 'main',
                    title: 'Repository cannot be empty 😠',
                    type: 'danger'
                })
                return;
            }

            if (!isWebUri(this.suggestion.repository))
            {
                this.$notify({
                    group: 'main',
                    title: 'Repository must be valid URL 😠',
                    type: 'danger'
                })
                return;
            }

            this.loading = true
            api.post('/suggestions', {
                suggestion: this.suggestion,
                recaptcha: this.recaptcha
            }).then(_ => {
                this.$notify({
                    group: 'main',
                    title: 'Thank you for a suggestion 😊',
                    type: 'success'
                })
                this.reset()
                this.suggestion.repository = ""
                this.loading = false
            }).catch(e => {
                this.$notify({
                    group: 'main',
                    title: 'Oh no! Something went wrong 😵',
                    type: 'danger'
                })
                this.reset()
                this.loading = false
            })
        },
        onVerify(response) {
            this.recaptcha = response
            this.verified = true
        },
        reset() {
            this.$refs.recaptcha.reset()
        }
    }
}
</script>

<style lang="stylus">
primary = #41B883
secondary = #35495E
color__gray = #BABFBD

.btn[disabled="disabled"]
    background color__gray
    cursor not-allowed

.form
    &__helper
        margin-top 0.4em
        color gray

    &__submit
        display flex
        align-items center
        justify-content flex-start
        margin-top 1em

    &__group
        display flex
        flex-flow column
        align-items flex-start
        padding 0.7em 0

        label
            color secondary
            font-weight bold
            font-size 1.25em
            margin-bottom 0.2em

        code
            background #F2FAF7
            padding 0.25em 0.5em
            border-radius 4px

        input
            width 100%
            padding 0.4em 0.2em
            border-radius 4px
            border 1px solid color__gray
</style>
