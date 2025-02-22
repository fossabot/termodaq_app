<template>
  <div>
    <div class="app flex-row align-items-center">
      <div class="container">
        <div class="row justify-content-center">
          <div class="col-md-8">
            <div class="card-group">
              <div class="card p-4">
                <div class="card-body">
                  <form @submit.prevent="onSubmit">
                    <h1>Login</h1>
                    <p class="text-muted">Inicia sesión en tu cuenta</p>
                    <div role="group" class="input-group mb-3">
                      <div class="input-group-prepend">
                        <div class="input-group-text">
                          <i class="fas fa-user"></i>
                        </div>
                      </div>
                      <input
                        v-model="userData.username"
                        type="text"
                        placeholder="Usuario"
                        autocomplete="usuario email"
                        class="form-control form-control"
                        id="_usuario"
                        required
                      />
                    </div>
                    <div role="group" class="input-group mb-4">
                      <div class="input-group-prepend">
                        <div class="input-group-text">
                          <i class="fas fa-key"></i>
                        </div>
                      </div>
                      <input
                        v-model="userData.password"
                        type="password"
                        placeholder="Contraseña"
                        autocomplete="current-password"
                        class="form-control form-control"
                        id="_password"
                        required
                      />
                    </div>
                    <div class="row">
                      <div class="col-12">
                        <button
                          type="submit"
                          class="btn px-4 mb-2 btn-primary w-100"
                          :disabled="loginBtn"
                        >Entrar</button>
                      </div>
                    </div>
                    <div class="row">
                      <div class="text-center col-6 col-lg-12 col-xl-12">
                        <button
                          type="button"
                          @click="restablecer"
                          class="btn px-0 btn-link"
                        >¿Has olvidado tu contraseña?</button>
                      </div>
                      <div class="text-center d-lg-none d-xl-none">
                        <button
                          type="button"
                          @click="registro"
                          class="btn px-0 btn-link"
                        >¡Regístrate ahora!</button>
                      </div>
                    </div>
                  </form>
                </div>
              </div>
              <div class="card text-white bg-primary py-5 d-md-down-none" style="width: 44%;">
                <div class="card-body text-center">
                  <div>
                    <h2>¿Usuario Nuevo?</h2>
                    <p
                      class="mt-4"
                    >Para utilizar la plataforma usted necesita una cuenta de usuario, esto le permite respaldar y visualizar los muestreos que realice con su tarjeta termoDaQ.</p>
                    <button
                      type="button"
                      @click="registro"
                      class="btn active mt-3 btn-primary"
                    >¡Regístrate ahora!</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loginBtn: false,
      userData: {
        username: "",
        email: "",
        password: ""
      }
    };
  },

  asyncData(context) {
    return {
      project: "SPC"
    };
  },

  async mounted() {
    let env = require("~/const/env.json");
    let token = localStorage.getItem("token");
    let userId = localStorage.getItem("userId");
    let url = `${env.api_host}/usuario/${userId}/accessTokens?access_token=${token}`;
    let apiToken = "";
    let self = this;

    if (!token) {
      return;
    }

    await this.$axios
      .$get(urlToken)

      .then(function(response) {
        response.forEach(element => {
          if (element.id === token) {
            apiToken = token;
            console.log("token-presente");
          }
        });

        if (token != apiToken) {
          localStorage.setItem("token", "");
          localStorage.setItem("userId", "");
        }
      })

      .catch(function(e) {
        if (e.response) {
          let error = e.response.data.error;
          let detalles = error.details;

          if ((error.statusCode = 401)) {
            console.log("Acceso no autorizado, inicie sesión nuevamente");
          }

          self.$toast.error("Acceso no autorizado, inicie sesión nuevamente", {
            duration: 5000,
            iconPack: "fontawesome",
            icon: "check"
          });

          localStorage.setItem("token", "");
          localStorage.setItem("userId", "");
        } else {
          console.log(e);
        }
      });
  },

  methods: {
    onSubmit(params) {
      this.loginBtn = true;

      let userData = this.userData;
      let env = require("~/const/env.json");
      let url = env.api_host + "/usuario/login";
      let self = this;

      this.$axios({
        method: "post",
        url: url,
        mode: "no-cors",
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Content-Type": "application/json"
        },
        withCredentials: false,
        data: {
          username: userData.username,
          password: userData.password
        }
      })

        .then(function(response) {
          self.loginBtn = false;

          localStorage.setItem("token", response.data.id);
          localStorage.setItem("userId", response.data.userId);

          self.$toast.success("Bienvenido", {
            duration: 3500,
            iconPack: "fontawesome",
            icon: "check"
          });

          self.$router.push("/tablero/usuario/perfil");
        })

        .catch(function(e) {
          self.loginBtn = false;

          if (e.response) {
            let error = e.response.data.error;
            let detalles = error.details;

            if (error.code == "LOGIN_FAILED_EMAIL_NOT_VERIFIED") {
              self.$toast.error(
                "Cuenta no verificada, un link de confirmacion fue enviado a su email",
                {
                  duration: 10000,
                  iconPack: "fontawesome",
                  icon: "times"
                }
              );
            }

            if (error.code == "LOGIN_FAILED") {
              self.$toast.error("Login invalido, verifique sus datos", {
                duration: 3500,
                iconPack: "fontawesome",
                icon: "times"
              });
            }
          } else {
            console.log(e);
          }
        });
    },

    registro(params) {
      this.$router.push("/auth/registro");
    },

    restablecer(params) {
      this.$router.push("/auth/restablecer-contraseña");
    }
  }
};
</script>
