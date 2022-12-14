<template>
  <div v-if="$config.oneEventVersion === false">
    <section class="grid-container">
      <div class="grid-row grid-gap">
        <div class="tablet:grid-col">
          <h1
            v-if="benefitTopic"
            class="font-heading-lg tablet:font-heading-xl margin-top-5 text-secondary">
            {{ benefitTopic }}
          </h1>
          <p
            v-if="topic && topic.lede"
            class="tablet:font-heading-lg line-height-serif-6 text-normal measure-6">
            {{ topic.lede }}
          </p>
        </div>
      </div>

      <div class="grid-row grid-gap print:display-block">
        <div class="tablet:grid-col margin-bottom-3">
          {{ $t("lifeEvent.labelShowText") }} {{ lifeEventBenefits.length }} {{ $t("lifeEvent.labelShowText2") }}
        </div>
      </div>

      <div class="display-none tablet:display-flex grid-row grid-gap print:display-block">
        <div class="tablet:grid-col margin-bottom-3">
          <div>
            <button
              class="usa-button usa-button--unstyled open-all"
              aria-controls="acc-id"
              @click="openAll">
              {{ $t("lifeEvent.buttonLabel1") }}
            </button>
            /
            <button
              class="usa-button usa-button--unstyled close-all"
              aria-controls="acc-id"
              @click="closeAll">
              {{ $t("lifeEvent.buttonLabel2") }}
            </button>
            /
            <button
              class="usa-button usa-button--unstyled clear-all"
              aria-controls="acc-id"
              @click="clearCriteria">
              {{ $t("lifeEvent.buttonLabel3") }}
            </button>
          </div>
        </div>
      </div>

      <div class="grid-row grid-gap print:display-block">
        <div class="tablet:grid-col-7 desktop:grid-col-8">
          <div
            v-if="$fetchState.pending"
            class="usa-alert usa-alert--info usa-alert--no-icon usa-alert--slim">
            <div class="usa-alert__body">
              <p class="usa-alert__text">{{ $t("lifeEvent.fetchState.pending") }}</p>
            </div>
          </div>
          <div
            v-if="$fetchState.error"
            class="usa-alert usa-alert--error usa-alert--slim">
            <div class="usa-alert__body">
              <p class="usa-alert__text">{{ $t("lifeEvent.fetchState.error") }}</p>
            </div>
          </div>
          <div
            v-if="lifeEventBenefits && lifeEventBenefits.length == 0"
            class="usa-alert usa-alert--error usa-alert--slim">
            <div class="usa-alert__body">
              <p class="usa-alert__text">{{ $t("lifeEvent.fetchState.none") }}</p>
            </div>
          </div>
          <!-- Mobile meta sort and open -->
          <div class="tablet:display-none">
            <h2 class="font-heading-lg margin-top-1">{{ $t("lifeEvent.benefits") }} {{ $t("lifeEvent.results") }}</h2>
            <button
              class="usa-button clear-all"
              aria-controls="acc-id"
              @click="clearCriteria">
              {{ $t("lifeEvent.buttonLabel3") }}
            </button>
            <OpenCloseButtons
              :is-open-active-prop="true"
              @open-all="openAll"
              @close-all="closeAll" />
          </div>

          <Accordion
            ref="accordion"
            :life-event-benefits="lifeEventBenefits"
            :expanded="true"
            :show-icons="true"
            :show-matching-count="false" />
        </div>
      </div>

      <div class="grid-row grid-gap margin-top-2">
        <div class="grid-col">
          <print @print="openAll()" />
        </div>
      </div>
    </section>

    <cross-sell
      title="Other topics that might be relevant to you."
      :cards="topic.related"
      class="print:display-none" />
  </div>
</template>

<script>
import _ from "lodash"
import { tObj, tCsv } from "~/services/translation"
import OpenCloseButtons from "~/components/OpenCloseButtons.vue"

export default {
  components: {
    OpenCloseButtons,
  },
  data() {
    return {
      benefitTopic: "",
      lifeEventBenefits: [],
      topic: {
        title: "",
        summary: "",
        lede: "",
        relatedKeys: [],
        related: [],
      },
    }
  },
  async fetch() {
    this.benefitTopic = _.capitalize(_.lowerCase(this.$route.params.type))
    const lifeEventBenefits = tObj.call(
      this,
      await this.$content("benefits")
        .where({
          tags: { $contains: _.lowerCase(this.$route.params.type) },
        })
        .sortBy("title")
        .fetch()
    )
    const allEligibilityCriteria = tCsv.call(this, await this.$content("criteria").fetch()).body
    await this.$store.dispatch("criteria/populate", allEligibilityCriteria)
    // eslint-d

    this.topic = tObj.call(this, await this.$content("types", this.$route.params.type).fetch())
    this.topic.related = []
    for (const related of this.topic.relatedKeys || []) {
      this.topic.related.push(tObj.call(this, await this.$content("types", related).fetch()))
    }
    this.lifeEventBenefits = lifeEventBenefits
  },
  head() {
    return {
      title: this.benefitTopic,
    }
  },
  mounted() {
    if (this.$config.oneEventVersion !== false) {
      this.$router.push(this.$route.fullPath.split("types")[0] + this.$config.oneEventVersion)
    }
  },
  /* istanbul ignore next */
  methods: {
    openAll() {
      this.$refs.accordion.openAll()
    },
    closeAll() {
      this.$refs.accordion.closeAll()
    },
    clearCriteria() {
      this.$store.dispatch("criteria/clear")
    },
  },
}
</script>
