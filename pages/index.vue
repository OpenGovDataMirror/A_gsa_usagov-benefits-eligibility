<template>
  <div>
    <section
      class="grid-container usa-section"
      :style="$config.oneEventVersion !== false ? 'display: none' : ''">
      <div class="grid-row grid-gap">
        <div class="tablet:grid-col-10">
          <h1 class="font-heading-lg tablet:font-heading-xl margin-top-0 text-secondary">
            {{ landingPage.title }}
          </h1>
          <p class="tablet:font-heading-lg line-height-serif-6 text-normal measure-6">
            {{ landingPage.summary }}
          </p>
          <ol class="usa-process-list">
            <li
              v-for="step in landingPage.processListSteps"
              :key="step"
              class="usa-process-list__item padding-bottom-4">
              <p class="usa-process-list__heading font-sans-md tablet:font-sans-lg line-height-sans-1">
                {{ step }}
              </p>
            </li>
          </ol>
        </div>
      </div>

      <div class="grid-row grid-gap margin-top-4">
        <div class="tablet:grid-col-10">
          <ul
            v-if="lifeEvents && lifeEvents.length > 0"
            class="usa-card-group">
            <li
              v-for="event in lifeEvents"
              :key="event.slug"
              class="usa-card desktop:grid-col-6"
              :aria-label="$t(event.title)">
              <nuxt-link
                class="display-block height-full margin-x-1"
                style="text-decoration: none; outline-offset: 0.25rem"
                :to="localePath(`/${event.slug}`)">
                <Card
                  :card-body="$t(event.summary)"
                  :card-title="$t(event.title)"
                  :card-container-classes="['hover:border-base-light', 'margin-x-0']"
                  card-title-heading-level="h2" />
              </nuxt-link>
            </li>
          </ul>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { tObj } from "~/services/translation"
export default {
  layout: "default",
  async asyncData({ $content }) {
    const lifeEvents = await $content("life-events").sortBy("title").fetch()
    const landingPage = await $content("landing-page").fetch()
    return { lifeEvents, landingPage }
  },
  data() {
    return {
      lifeEvents: [],
      landingPage: {},
    }
  },
  mounted() {
    if (this.$config.oneEventVersion !== false) {
      this.$router.replace(this.$route.fullPath + this.$config.oneEventVersion)
    }
    this.landingPage = tObj.call(this, this.landingPage)
  },
}
</script>
