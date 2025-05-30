<route lang="json">
{
  "meta": {
    "title": "Login - Artalk Sidebar"
  }
}
</route>

<script setup lang="ts">
import type { ArtalkType } from 'artalk'
import { getArtalk } from '../global'
import { useUserStore } from '../stores/user'

const router = useRouter()
const { t } = useI18n()

let userForm = ref({
  email: '',
  password: '',
})
let version = ref('')
let buildHash = ref('')
let loginErr = ref('')
let userSelector = ref<string[] | null>(null)

onMounted(() => {
  getArtalk()!
    .ctx.getApi()
    .version.getVersion()
    .then((res) => {
      version.value = res.data.version
      buildHash.value = res.data.commit_hash
    })
})

function onFocus() {
  loginErr.value = ''
}

function login(username?: string) {
  loginErr.value = ''

  const artalk = getArtalk()
  if (!artalk) throw new Error('Artalk instance not initialized')

  artalk.ctx
    .getApi()
    .user.login({
      name: username || '',
      email: userForm.value.email,
      password: userForm.value.password,
    })
    .then((res) => {
      artalk.ctx.getUser().update({
        ...res.data.user,
        token: res.data.token,
      })
      useUserStore().sync()
      router.replace('/')
    })
    .catch((e: ArtalkType.FetchError) => {
      if (e.data?.need_name_select) {
        userSelector.value = e.data?.need_name_select
      } else {
        loginErr.value = e.message || t('loginFailure')
      }
    })
}

function selectUser(username: string) {
  userSelector.value = null
  login(username)
}

const versionInfo = computed(() => {
  return `v${version.value + (buildHash.value ? ' / ' + buildHash.value : '')}`
})
</script>

<template>
  <div class="login-dialog">
    <a href="https://artalk.js.org/" target="_blank">
      <img class="logo" src="../assets/favicon.png" alt="logo" draggable="false" />
    </a>
    <form class="login-form" @submit.prevent="login()">
      <input v-model="userForm.email" type="text" :placeholder="t('email')" @focus="onFocus" />
      <input
        v-model="userForm.password"
        type="password"
        :placeholder="t('password')"
        @focus="onFocus"
      />
      <div v-if="!!loginErr" class="err-msg atk-fade-in">{{ loginErr }}</div>
      <button type="submit">{{ t('login') }}</button>
    </form>
    <div class="copyright">
      Powered by
      <a href="https://artalk.js.org" target="_blank">Artalk</a>
      ({{ versionInfo }})
    </div>

    <div v-if="userSelector" class="layer">
      <div class="user-selector atk-fade-in">
        <div class="text">{{ t('loginSelectHint') }}</div>
        <div class="user-list">
          <div v-for="(u, i) in userSelector" :key="i" class="item" @click="selectUser(u)">
            {{ u }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.login-dialog {
  z-index: 11;
  position: fixed;
  display: flex;
  flex-direction: column;
  place-items: center;
  justify-content: center;
  top: 0;
  left: 0;
  background: var(--at-color-bg);
  width: 100%;
  height: 100%;

  .logo {
    user-select: none;
    display: inline-block;
    width: 50px;
    height: 50px;
    margin-bottom: 50px;
    margin-top: -50px;
    border-radius: 2px;
  }

  .copyright {
    font-size: 13px;
    color: var(--at-color-meta);
    position: fixed;
    height: 30px;
    bottom: 0;
    width: 100%;
    text-align: center;
  }
}

.login-form {
  display: flex;
  flex-direction: column;
  width: 280px;

  input {
    display: block;
    border: 0;
    background: transparent;
    border-bottom: 1px solid var(--at-color-border);
    padding: 0 20px;
    line-height: 40px;
    margin-bottom: 10px;

    &:focus {
      outline: none;
    }
  }

  button {
    color: #fff;
    background: #6e8392;
    border-radius: 2px;
    cursor: pointer;
    border: 0;
    line-height: 35px;
    padding: 0 20px;
    margin-top: 20px;
    display: block;

    &:hover {
      opacity: 0.95;
    }
  }

  .err-msg {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0 10px;
    margin-top: 5px;
    padding: 0 10px;
    color: var(--at-color-red);

    &::before {
      content: '';
      display: inline-block;
      vertical-align: bottom;
      height: 19px;
      width: 19px;
      margin-right: 6px;
      background-size: contain;
      background-position: center;
      background-repeat: no-repeat;
      background-image: url("data:image/svg+xml,%3Csvg width='19' height='19' viewBox='0 0 19 19' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M9.5 15.8332C12.9978 15.8332 15.8333 12.9976 15.8333 9.49984C15.8333 6.00203 12.9978 3.1665 9.50001 3.1665C6.0022 3.1665 3.16667 6.00203 3.16667 9.49984C3.16667 12.9976 6.0022 15.8332 9.5 15.8332ZM17.4167 9.49984C17.4167 13.8721 13.8723 17.4165 9.5 17.4165C5.12775 17.4165 1.58334 13.8721 1.58334 9.49984C1.58334 5.12758 5.12775 1.58317 9.50001 1.58317C13.8723 1.58317 17.4167 5.12758 17.4167 9.49984ZM8.70834 13.4581L8.70834 11.8748L10.2917 11.8748L10.2917 13.4581L8.70834 13.4581ZM10.2917 5.54148L10.2917 11.0831L8.70834 11.0831L8.70834 5.54148L10.2917 5.54148Z' fill='%23F53F3F'/%3E%3C/svg%3E");
    }
  }
}

.layer {
  position: fixed;
  width: 100%;
  height: 100%;
  background: var(--at-color-bg-transl);
}

.user-selector {
  z-index: 999;
  position: fixed;
  left: 50%;
  top: 43.5%;
  transform: translate(-50%, -50%);
  background: var(--at-color-bg);
  border: 1px solid var(--at-color-border);
  padding-bottom: 10px;
  width: 280px;

  & > .text {
    font-size: 15px;
    padding: 15px 20px;
  }

  .user-list {
    & > .item {
      cursor: pointer;
      padding: 10px 20px;
      transition: border 0.3s ease;
      border-left: 2px solid transparent;

      &:hover {
        border-left: 2px solid #6e8392;
      }
    }
  }
}
</style>
