<template>
  <form class="bg-white p-8 rounded-lg w-full max-w-sm mx-auto h-auto">
    <header class="text-center my-2 text-slate-300">
      <slot name="header"></slot>
    </header>
    <div v-for="field in fields" :key="field.id" class="mb-6">
      <input
        :placeholder="field.placeholder"
        :type="field.type"
        :id="field.id"
        :name="field.name"
        v-model="field.value"
        autocomplete="new-password"
        class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-50"
      />
    </div>
    <button :class="buttonClass" type="submit" @click="onClick">
      {{ buttonText }}
    </button>
    <div v-if="googleButton" class="mt-4">
      <div class="relative">
        <div>
          <button
            @click="onClickGoogle"
            class="flex items-center justify-center w-full max-w-xs p-3 bg-white border border-gray-300 rounded-md shadow-sm hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            <svg class="w-6 h-6 mr-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48">
              <g>
                <path
                  fill="#EA4335"
                  d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"
                ></path>
                <path
                  fill="#4285F4"
                  d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"
                ></path>
                <path
                  fill="#FBBC05"
                  d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"
                ></path>
                <path
                  fill="#34A853"
                  d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"
                ></path>
                <path fill="none" d="M0 0h48v48H0z"></path>
              </g>
            </svg>
            <span>Sign in with Google</span>
          </button>
        </div>
      </div>
    </div>
    <div class="pt-2">
      <router-link :to="linkTo" class="text-blue-500 hover:text-blue-700">
        {{ linkText }}
      </router-link>
    </div>
    <div class="text-center mt-2" :class="linkClass">
      <slot name="link"></slot>
    </div>
  </form>
</template>

<script setup>
const props = defineProps({
  fields: Array,
  buttonText: {
    type: String,
    default: 'Submit'
  },
  buttonClass: {
    type: String,
    default:
      'w-full bg-blue-600 text-white font-bold py-2 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50'
  },
  linkText: {
    type: String,
    default: ''
  },
  linkTo: {
    type: String,
    default: ''
  },
  linkClass: {
    type: String,
    default: ''
  },
  googleButton: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['button-click', 'google-sign-in'])

function onClick(event) {
  event.preventDefault()
  emit('button-click')
}

function onClickGoogle(event) {
  event.preventDefault()
  emit('google-sign-in')
}
</script>
