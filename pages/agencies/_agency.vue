<template>
  <div v-if="$config.oneEventVersion === false">
    <section class="grid-container">
      <div class="grid-row grid-gap">
        <div class="tablet:grid-col">
          <h1
            v-if="benefitAgency"
            class="font-heading-lg tablet:font-heading-xl margin-top-5 text-secondary">
            {{ benefitAgency }}
          </h1>
          <p
            v-if="agency && agency.lede"
            class="tablet:font-heading-lg line-height-serif-6 text-normal measure-6">
            {{ agency.lede }}
          </p>
        </div>
      </div>

      <div class="grid-row grid-gap">
        <div class="tablet:grid-col margin-bottom-3">Showing {{ lifeEventBenefits.length }} benefits</div>
      </div>
      <!-- Desktop meta sort and open -->
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
          <print @print="openAll" />
        </div>
      </div>
    </section>
    <cross-sell
      title="Other agencies that might be relevant to you."
      :cards="agency.related"
      class="print:display-none" />
  </div>
</template>

<script>
import _ from "lodash"
import { tObj, tCsv } from "~/services/translation"
import mapTags from "~/mixins/MapTags"
import OpenCloseButtons from "~/components/OpenCloseButtons.vue"

export default {
  components: {
    OpenCloseButtons,
  },
  mixins: [mapTags],
  data() {
    return {
      benefitAgency: "",
      lifeEventBenefits: [],
      agency: {
        title: "",
        summary: "",
        lede: "",
        relatedKeys: [],
        related: [],
      },
    }
  },
  async fetch() {
    const slug = this.$route.params.agency.startsWith("u-s-")
      ? _.lowerCase(this.$route.params.agency).replace(/^u s /, "u.s. ")
      : _.lowerCase(this.$route.params.agency)
    const agencyRegex = new RegExp(_.escapeRegExp(slug), "i")
    const lifeEventBenefits = await this.$content("benefits").sortBy("title").fetch()
    const translatedBenefits = lifeEventBenefits.map((benefit) => tObj.call(this, benefit))
    const allEligibilityCriteria = tCsv.call(this, await this.$content("criteria").fetch()).body
    await this.$store.dispatch("criteria/populate", allEligibilityCriteria)
    this.lifeEventBenefits = translatedBenefits.filter(
      (benefit) => benefit?.source?.name && agencyRegex.test(benefit.source.name)
    )
    this.benefitAgency = this.lifeEventBenefits[0]?.source?.name
    this.agency = tObj.call(this, await this.$content("agencies", this.$route.params.agency).fetch())
    this.agency.related = []
    for (const related of this.agency.relatedKeys || []) {
      this.agency.related.push(await this.$content("agencies", related).fetch())
    }
  },
  /* istanbul ignore next */
  head() {
    return {
      title: this.benefitAgency,
    }
  },
  mounted() {
    if (this.$config.oneEventVersion !== false) {
      this.$router.push(this.$route.fullPath.split("agencies")[0] + this.$config.oneEventVersion)
    }
  },
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

<style scoped>
h1 {
  text-transform: capitalize;
}
</style>
