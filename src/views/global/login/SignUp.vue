<template>
  <section>
    <h1>Create account</h1>
    <div v-if="signingUp">
      <img src="@/assets/spinner.svg">
    </div>
    <div v-if="errorMessage" class="alert alert-danger" role="alert">{{ errorMessage }}</div>
    <div class="container">
      <div class="row">
        <div class="col-lg=3 col-md-2 col-sm-1 col-xs-12"></div>
        <div class="col-lg=6 col-md-8 col-sm-10 col-xs-12">
          <form v-if="!signingUp" @submit.prevent="signup" class="form-global">
            <div class="form-group">
              <label for="name">username</label>
              <input
                v-model="user.username"
                type="text"
                class="form-control-global"
                id="name"
                aria-describedby="nameHelp"
                required
              >
            </div>
            <div class="form-group">
              <label for="email">Email</label>
              <input
                v-model="user.email"
                type="text"
                class="form-control-global"
                id="email"
                aria-describedby="emailHelp"
                required
              >
            </div>
            <div class="form-row">
              <div class="form-group col-md-6">
                <label for="password">Password</label>
                <input
                  v-model="user.password"
                  type="password"
                  class="form-control-global"
                  id="password"
                  aria-describedby="passwordHelp"
                  required
                >
              </div>
              <div class="form-group col-md-6">
                <label for="confirmPassword">Confirm Password</label>
                <input
                  v-model="user.confirmPassword"
                  type="password"
                  class="form-control-global"
                  id="confirmPassword"
                  aria-describedby="confirmPasswordHelp"
                  required
                >
              </div>
            </div>
            <button type="submit" class="button-submit">Signup</button>
          </form>
          <div class="mt-3">
            <router-link to="/login">I am already a member</router-link>
            </div>
        </div>
      </div>
      <div class="col-lg=3 col-md-2 col-sm-1 col-xs-12"></div>
    </div>
  </section>
</template>

<script>
import Joi from "joi";
const { API_URL } = require("@/constants.js");
const SIGNUP_URL = API_URL + "/signup";
export default {
    name: 'signUp',
  data: () => ({
    signingUp: false,
    errorMessage: "",
    user: {
      email: "",
      password: "",
      confirmPassword: "",
      username: ""
    }
  }),
  watch: {
    user: {
      handler() {
        this.errorMessage = "";
      },
      deep: true
    }
  },
  methods: {
    signup() {
      this.errorMessage = "";
      if (this.validUser()) {
        const body = {
          email: this.user.email,
          password: this.user.password,
          username: this.user.username
        };
        this.signingUp = true;
        fetch(SIGNUP_URL, {
          method: "POST",
          body: JSON.stringify(body),
          headers: {
            "content-type": "application/json"
          }
        })
          .then(response => {
            // console.log("response", response);
            if (response.ok) {
              return response.json();
            }
            return response.json().then(error => {
              throw new Error(error.error);
            });
          })
          .then(result => {
            localStorage.token = result.token;
            this.signingUp = false;
            this.$router.go("/login");
          })
          .catch(error => {
            // console.log("error", error);
            this.signingUp = false;
            this.errorMessage = error.message;
          });
      }
    },
    validUser() {
      const schema = Joi.object().keys({
        email: Joi.string()
          .email()
          .required(),
        password: Joi.string()
          .regex(/^[a-zA-Z0-9]{5,30}$/)
          .required(),
        confirmPassword: Joi.string()
          .regex(/^[a-zA-Z0-9]{5,30}$/)
          .required(),
        username: Joi.string()
          .min(5)
          .max(30)
          .required()
      });
      if (this.user.password !== this.user.confirmPassword) {
        this.errorMessage = "Passwords must match.";
        return false;
      }
      const result = Joi.validate(this.user, schema);
      if (result.error === null) {
        return true;
      }
      if (result.error.message.includes("name")) {
        this.errorMessage = "bad length or format of a name";
        return false;
      }
      if (result.error.message.includes("email")) {
        this.errorMessage = "email is invalid.";
      } else {
        this.errorMessage = "Password is invalid.";
      }
      return false;
    }
  }
};
</script>

<style>

</style>