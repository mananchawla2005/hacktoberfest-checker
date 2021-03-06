<template>
  <div class="z-0 w-full h-without-header">
    <div class="flex flex-col items-center justify-end text-center h-3/4">
      <div class="w-1/2">
        <h1
          class="text-2xl Title md:text-4xl xl:text-6xl"
          v-html="$t('components.check_repository.title')"
        />
      </div>
      <div class="w-4/5 mt-4 mb-8">
        <div v-if="isErrors" class="Errors">
          <h1 class="text-xl Title md:text-2xl xl:text-4xl">
            {{ $t('components.check_repository.error') }}
          </h1>

          <ul class="mb-4">
            <li v-for="(error, index) in errors" :key="index">
              <code>{{ error.message ? error.message : error }}</code>
            </li>
          </ul>

          <a
            href="#"
            class="inline-flex items-center px-4 py-2 font-bold text-gray-800 bg-gray-300 rounded hover:bg-gray-400"
            @click.prevent="resetForm"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              class="w-4 h-4 mr-2 fill-current"
            >
              <path
                fillRule="evenodd"
                d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z"
                clipRule="evenodd"
              />
            </svg>
            {{ $t('components.check_repository.retry') }}
          </a>
        </div>
        <div v-else-if="result" class="Result">
          <h2
            class="text-xl Title md:text-2xl xl:text-4xl"
            :class="{
              'Title--yes': theyHacktoberfest,
              'Title--no': !theyHacktoberfest,
            }"
          >
            {{ theyHacktoberfest ? 'Yes' : 'No' }}
          </h2>
          <h3>
            {{ $t('components.check_repository.results_are_in') }}
            <strong>{{ result.long_name || result.name }}</strong>
          </h3>
          <h3
            v-if="theyHacktoberfest"
            v-html="$t('components.check_repository.success')"
          />
          <h3 v-else v-html="$t('components.check_repository.failure')" />
          <a
            href="#"
            class="inline-flex items-center px-4 py-2 font-bold text-gray-800 bg-gray-300 rounded hover:bg-gray-400"
            @click.prevent="resetForm"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              class="w-4 h-4 mr-2 fill-current"
            >
              <path
                fillRule="evenodd"
                d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z"
                clipRule="evenodd"
              />
            </svg>
            {{ $t('components.check_repository.retry') }}
          </a>
        </div>
        <div v-else-if="processing" class="Processing">
          <scale-loader
            :loading="processing"
            :color="
              $colorMode.value == 'dark'
                ? loaderOpt.darkColor
                : loaderOpt.lightColor
            "
            :height="loaderOpt.height"
            :width="loaderOpt.width"
          ></scale-loader>
        </div>
        <div v-else class="relative">
          <svg
            class="absolute top-0 w-8 h-8 mt-6 ml-6 text-gray-600"
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <form class="flex" @submit.prevent="checkRepository()">
            <input
              v-model="url"
              class="text-sm Input md:text-1xl xl:text-3xl"
              type="text"
              :placeholder="`e.g. https://github.com/${randomRepoName} or ${randomRepoName}`"
              :disabled="processing"
            />
            <button
              :disabled="processing"
              class="text-sm Button md:text-1xl xl:text-3xl"
            >
              {{ $t('components.check_repository.cta') }}
            </button>
          </form>
        </div>
        <div>
          <ul class="Previous">
            <template v-if="previous.length > 0">
              <Repository
                v-for="(repo, index) in previous"
                :key="index"
                :repo="repo"
                @remove="removePrevious"
                @refresh="refreshPrevious"
              />
            </template>
            <li v-else>No previous repo checks found... yet!</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ScaleLoader from 'vue-spinner/src/ScaleLoader.vue'

export default {
  components: {
    ScaleLoader,
  },
  data() {
    return {
      url: null,
      processing: false,
      errors: [],
      result: null,
      previous: [],
      loaderOpt: {
        lightColor: '#9c4668',
        darkColor: '#ff8ae2',
        height: '50px',
        width: '10px',
      },
      repos: [
        'cli/cli',
        'digitalocean/hacktoberfest',
        'lukeocodes/guys-bot',
        'alhassanv/boilerplate',
      ],
    }
  },

  computed: {
    isErrors() {
      return this.errors.length !== 0
    },

    theyHacktoberfest() {
      return (this.result.topic || this.result.tag_prs) && !this.result.banned
    },

    randomRepoName() {
      return this.repos[Math.floor(Math.random() * this.repos.length)]
    },
  },

  watch: {
    previous(previous) {
      localStorage.setItem('previousResults', JSON.stringify(previous))
    },
  },

  mounted() {
    if (localStorage.previousResults) {
      this.previous = JSON.parse(localStorage.getItem('previousResults')) || []
    }
  },

  methods: {
    resetForm() {
      this.url = null
      this.processing = false
      this.errors = []
      this.result = null
    },

    checkRepository() {
      if (!this.url) {
        return this.errors.push({ message: 'Please enter a url to check.' })
      }

      // eslint-disable-next-line no-var
      var isGithub = new RegExp(
        "https?:\\/\\/(.+?\\.)?github\\.com(\\/[A-Za-z0-9\\-\\._~:\\/\\?#\\[\\]@!$&'\\(\\)\\*\\+,;\\=]*)?"
      )
      // eslint-disable-next-line no-var
      var isGithubRepo = new RegExp('[a-zA-Z]+/[a-zA-Z]+')

      if (!isGithub.test(this.url)) {
        this.url = `https://github.com/${this.url}`
      }

      if (!isGithub.test(this.url) && !isGithubRepo.test(this.url)) {
        return this.errors.push({
          message: 'Please use a GitHub url to check.',
        })
      }

      this.processing = true
      this.$axios
        .$get(`/api/check-repository?url=${this.url}`)
        .then((result) => {
          this.url = null
          this.processing = false
          this.result = result
          this.previous.unshift(result)
        })
        .catch((error) => {
          this.errors.push(error)
        })
    },

    removePrevious(key) {
      this.previous = this.previous.filter((p, i) => i !== key)
    },

    refreshPrevious(key) {
      this.resetForm()

      const repo = this.previous[key]
      this.previous = this.previous.filter((p, i) => i !== key)
      this.url = repo.url

      this.checkRepository()
    },
  },
}
</script>

<style scoped>
.Title {
  @apply font-bold leading-none mb-4;
  color: var(--color-tertiary);
}

.Title >>> strong {
  color: var(--color-secondary);
}

.Result h3 {
  @apply text-lg mb-4;
  color: var(--color-tertiary);
}
.Result h3 strong {
  color: var(--color-secondary);
}

.Errors h3 {
  @apply mb-4;
  color: var(--color-tertiary);
}
.Errors h3 strong {
  color: var(--color-secondary);
}

.Input {
  @apply bg-gray-100 text-gray-800 pb-4 pt-5 pl-20 pr-4 rounded-l-lg w-full border-b-4 shadow-inner;
}

.Input:focus {
  @apply outline-none;
  border-bottom-color: var(--color-secondary);
}

.Button {
  @apply px-8 rounded-r-lg w-1/4 pb-4 pt-5 pl-4 pr-4 border-t border-b-4 border-r;
  background-color: var(--color-tertiary);
  color: var(--bg);
  border-right-color: var(--color-tertiary);
  border-top-color: var(--color-tertiary);
  border-bottom-color: var(--color-tertiary);
}

.Button:hover {
  background-color: var(--color-secondary);
  border-right-color: var(--color-secondary);
  border-top-color: var(--color-secondary);
  border-bottom-color: var(--color-secondary);
}

.Previous {
  @apply mt-10 w-9/12 mx-auto;
}
</style>
