<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <v-card>
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-card-title class="headline">
            Login
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="user.name"
              :rules="nameRules"
              label="Name"
              required
              @keyup.enter="validate"
            ></v-text-field>
            <v-text-field
              v-model="user.email"
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
  // middleware({ store, redirect }) {
  //   // If the user is not authenticated
  //   if (store.state.auth.loggedIn) {
  //     return redirect('/')
  //   }
  // },
  auth: 'guest',
  mounted() {
    // Before loading login page, obtain csrf cookie from the server.
    this.$axios.$get('http://localhost:8000/sanctum/csrf-cookie', {
      headers: {
        'X-Requested-With': 'XMLHttpRequest'
      },
      withCredentials: true
    })
  },
  data() {
    return {
      valid: true,
      user: {
        name: '',
        email: '',
        password: ''
      },
      show: false,
      password: 'Password',
      nameRules: [(v) => !!v || 'Name is required'],
      emailRules: [
        (v) => !!v || 'E-mail is required',
        (v) => /.+@.+\..+/.test(v) || 'E-mail must be valid'
      ],
      passwordRules: [(v) => !!v || 'Password is required']
    }
  },
  methods: {
    validate() {
      // if (this.$refs.form.validate()) {
      // }
      this.register()
    },
    async register() {
      try {
        await this.$axios.post(
          'http://localhost:8000/register',
          {
            name: this.user.name,
            email: this.user.email,
            password: this.user.password
          },
          {
            withCredentials: true
          }
        )

        await this.$auth.loginWith('laravelSanctum', {
          data: {
            email: this.user.email,
            password: this.user.password
          }
        })

        this.$store.commit('snackbar/setSnack', 'Registered and loggedin')

        this.$router.push('/')
      } catch (e) {
        this.$store.commit(
          'snackbar/setSnack',
          `Error: ${e.response.data.message}`
        )
        this.error = e.response.data.message
      }
    },
    register1() {
      this.$axios
        .post(
          'http://localhost:8000/register',
          {
            name: this.user.name,
            email: this.user.email,
            password: this.user.password
          },
          {
            withCredentials: true
          }
        )
        .then(function(response) {
          console.log(response)
        })
    },
    async registerUser() {
      // this.$axios.get('http://localhost:8000/sanctum/csrf-cookie', {
      //   withCredentials: true
      // })
      // .then(function (response) {
      //   console.log(response);
      // })

      await this.$axios
        .$get('http://localhost:8000/sanctum/csrf-cookie', {
          withCredentials: true
        })
        .then(function(response) {
          console.log('resp' + response)
        })

      this.$axios
        .post('http://localhost:8000/register', {
          data: {
            name: this.user.name,
            email: this.user.email,
            password: this.user.password
          },
          withCredentials: true
        })
        .then(function(response) {
          console.log(response)
        })

      // await this.$auth.request('http://localhost:8000/sanctum/csrf-cookie', {
      //   maxRedirects: 0
      // })
      // this.$store.dispatch('snackbar/setSnackbar', {
      //   text: `Welcome ${this.$auth.user.name}`
      // })

      // this.$store.commit('snackbar/setSnack', 'csrf')
    }
  }
}
</script>
