<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <v-card>
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-card-title class="headline">
            <!-- Login {{ this.process.env.SERVER }} -->
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="user.username"
              :rules="emailRules"
              label="E-mail"
              required
              @keyup.enter="validate"
            ></v-text-field>
            <v-text-field
              v-model="user.password"
              :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
              :type="show ? 'text' : 'password'"
              :rules="passwordRules"
              label="Password"
              required
              @keyup.enter="validate"
              @click:append="show = !show"
            ></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn :disabled="!valid" color="success" @click="validate">
              Login
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
export default {
  middleware({ store, redirect }) {
    // If the user is not authenticated
    if (store.state.auth.loggedIn) {
      return redirect('/')
    }
  },
  auth: false,
  data() {
    return {
      valid: true,
      user: {
        username: '',
        password: ''
      },
      show: false,
      password: 'Password',
      emailRules: [
        (v) => !!v || 'E-mail is required',
        (v) => /.+@.+\..+/.test(v) || 'E-mail must be valid'
      ],
      passwordRules: [(v) => !!v || 'Password is required']
    }
  },
  methods: {
    validate() {
      if (this.$refs.form.validate()) {
        this.passwordGrantLogin()
      }
    },
    async passwordGrantLogin() {
      try {
        await this.$auth.loginWith('laravelSanctum', {
          data: {
            email: this.user.username,
            password: this.user.password
          }
        })
        // this.$store.dispatch('snackbar/setSnackbar', {
        //   text: `Welcome ${this.$auth.user.name}`
        // })
        this.$store.commit('snackbar/setSnack', 'Loggedin')

        this.$router.push('/')
      } catch (error) {
        this.$store.commit('snackbar/setSnack', `Error: ${error}`)
        // this.$store.dispatch('snackbar/setSnackbar', {
        //   color: 'red',
        //   text: `${error}`
        // })
        console.log(error)
      }
    }
  }
}
</script>
