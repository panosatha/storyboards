<template>
  <div class="flex flex-col w-screen h-screen bg-gray-200 gap-2">
    <!-- Header -->
    <div class="flex gap-10 m-2 items-center">
      <NuxtLink :to="`/`">
        <img src="~/static/eucp_logo.png" alt="EUCP Logo">
      </NuxtLink>
      <h1 class="text-2xl">
        Storyboard: {{ story.title }}
      </h1>
    </div>

    <!-- Chapter overview bar -->
    <div class="flex no-wrap text-left gap-2">
      <div v-for="(headline, idx) of headlines" :key="idx">
        <div role="button" class="flex-grow bg-white rounded p-3 prose" @click="toggleChapter(idx)">
          {{ headline }}
        </div>
      </div>
    </div>

    <!-- Chapter image and description -->
    <div v-for="(chapter, idx) in chapters" v-show="idx===currentChapter" :key="idx" class="flex gap-2 overflow-auto h-full">
      <div class="w-2/3 bg-white rounded">
        <img v-if="!chapter.props.image.endsWith('html')" :src="getContent(chapter.props.image)" class="object-contain w-auto h-full max-w-full max-h-full mx-auto">
        <iframe v-else :src="getContent(chapter.props.image)" frameborder="0" class="w-full h-full" />
      </div>
      <div class="p-4 w-1/3 bg-white rounded overflow-auto">
        <nuxt-content :document="chapter" class="prose mb-6" />
        <EditOnGitHub :target="gitHubURL()" />
        <p class="prose italic">
          For more information on editing stories, see <a href="https://blog.esciencecenter.nl/storyboards-for-science-communication-85e399e5c1b5" target="_blank">this blog post</a>.
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  async asyncData ({ $content, params }) {
    const story = await $content(params.story).fetch()
    const chapters = story.body.children
      .filter(child => child.tag === 'chapter')
      .map((child) => {
        return { body: { children: child.children }, props: child.props }
      })
    const headlines = chapters.map(chapter => chapter.props.headline)
    return { params, story, headlines, chapters }
  },
  data () {
    return {
      error: false,
      currentChapter: 0
    }
  },
  methods: {
    getContent (path) {
      return `stories/_${this.params.story}/${path}`
    },
    toggleChapter (i) {
      this.currentChapter = i
    },
    gitHubURL () {
      return `https://github.com/eucp-project/storyboards/blob/main/static/stories/${this.story.slug}.md`
    }
  }
}
</script>
