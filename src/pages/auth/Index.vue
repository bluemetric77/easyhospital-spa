<template>
  <q-page class="flex flex-center bg">
    <q-card style="width: 350px; max-width: 90vw">
      <q-bar class="entry-caption">
        <q-icon
          name="vpn_key"
          size="xs"
        />
      </q-bar>
      <div class="flex flex-center q-my-sm">
        <img
          :alt="profile.company_name"
          :src="profile.company_logo"
          spinner-color="white"
          style="height: 100px; max-width: 200px"
        />
      </div>
      <div class="q-pa-xs q-gutter-sm">
        <q-input
          v-model="txtuser"
          label="Username"
          bg-color="white"
          square
          outlined
          dense
          type="text"
        />
        <q-input
          v-model="txtpwd"
          square
          label="Password"
          outlined
          dense
          :type="isPwd ? 'password' : 'text'"
          hint="Password dengan toggle"
        >
          <template v-slot:append>
            <q-icon
              class="fas fa-eye-slash"
              @click="isPwd = !isPwd"
              color="green-10"
              size="xs"
            />
          </template>
        </q-input>
      </div>

      <q-separator />
      <q-card-section class="dialog-action-login q-pa-sm">
        <q-btn
          class="btn-login full-width"
          rounded
          label="Login"
          no-caps
          flat
          @click="check_auth()"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar, QSpinnerBars } from 'quasar'

export default defineComponent({
  name: 'UserAuthorization',
  setup() {
    const $q = useQuasar()
    const $store = useStore()
    const $router = useRouter()

    const txtuser = ref('')
    const txtpwd = ref('')
    const isPwd = ref(true)

    const profile = computed(() => {
      return $store.state.home.profile
    })

    async function check_auth() {
      let payload = {
        url: 'auth',
        user_id: txtuser.value,
        password: txtpwd.value
      }
      $q.loading.show({ spinner: QSpinnerBars, delay: 1000 })
      $store
        .dispatch('home/POST_DATA', payload)
        .then((respon) => {
          if (!(typeof respon === 'undefined')) {
            let msg = ''
            if (respon.success) {
              $q.sessionStorage.set('auth-jwt', respon.data.token)
              $router.push('/')
            } else {
              msg = respon.data
              $q.notify({
                color: 'red-10',
                textcolor: 'white',
                message: msg,
                position: 'top',
                timeout: 2000,
                icon: 'tag_faces'
              })
            }
          }
        })
        .finally((final) => {
          $q.loading.hide()
        })
    }

    async function refreshpage() {
      try {
        let props = {}
        props.url = 'profile'
        let respon = await $store.dispatch('home/GET_DATA', props)
        console.info('Login :' + JSON.stringify(respon))
        respon.company_logo = 'http://localhost:8000/' + respon.company_logo
        $store.commit('home/UpdateProfiles', respon)
      } finally {
      }
    }

    onMounted(async () => {
      await refreshpage()
      if ($q.localStorage.getItem('fullscreen')) {
        $q.fullscreen.request()
      } else {
        $q.fullscreen.exit()
      }
    })

    return {
      txtuser,
      txtpwd,
      isPwd,
      profile,
      check_auth,
      refreshpage
    }
  }
})
</script>
