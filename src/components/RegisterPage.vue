<template>
    <h1 v-t="'titles.register'" class="font-bold text-center my-4" />
    <hr />
    <div class="text-center">
        <form class="children:pb-3">
            <div>
                <input
                    v-model="username"
                    class="input"
                    type="text"
                    autocomplete="username"
                    :placeholder="$t('login.username')"
                    :aria-label="$t('login.username')"
                    v-on:keyup.enter="register"
                />
            </div>
            <div>
                <input
                    v-model="password"
                    class="input"
                    type="password"
                    autocomplete="password"
                    :placeholder="$t('login.password')"
                    :aria-label="$t('login.password')"
                    v-on:keyup.enter="register"
                />
            </div>
            <div>
                <a class="btn w-auto" @click="register" v-t="'titles.register'" />
            </div>
        </form>
    </div>
    <ConfirmModal
        v-if="showUnsecureRegisterDialog"
        @close="showUnsecureRegisterDialog = false"
        @confirm="
            forceUnsecureRegister = true;
            showUnsecureRegisterDialog = false;
            register();
        "
        :message="$t('info.register_no_email_note')"
    />
</template>

<script>
import { isEmail } from "../utils/Misc.js";
import ConfirmModal from "./ConfirmModal.vue";

export default {
    data() {
        return {
            username: null,
            password: null,
            showUnsecureRegisterDialog: false,
            forceUnsecureRegister: false,
        };
    },
    mounted() {
        //TODO: Add Server Side check
        if (this.getAuthToken()) {
            this.$router.push("/");
        }
    },
    activated() {
        document.title = "Register - Piped";
    },
    methods: {
        register() {
            if (!this.username || !this.password) return;
            if (isEmail(this.username) && !this.forceUnsecureRegister) {
                this.showUnsecureRegisterDialog = true;
                return;
            }
            this.fetchJson(this.authApiUrl() + "/register", null, {
                method: "POST",
                body: JSON.stringify({
                    username: this.username,
                    password: this.password,
                }),
            }).then(resp => {
                if (resp.token) {
                    this.setPreference("authToken" + this.hashCode(this.authApiUrl()), resp.token);
                    window.location = "/"; // done to bypass cache
                } else alert(resp.error);
            });
        },
    },
    components: { ConfirmModal },
};
</script>
