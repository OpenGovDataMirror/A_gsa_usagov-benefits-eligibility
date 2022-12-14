<template>
  <div>
    <a
      class="usa-skipnav"
      href="#main-content"
      >{{ $t("skipnav") }}</a
    >
    <TheBanner />
    <div class="usa-overlay" />
    <header class="usa-header usa-header--extended">
      <div class="usa-navbar display-flex flex-justify flex-align-center flex-justify-center">
        <div
          id="extended-logo"
          class="usa-logo"
          style="max-width: 60%">
          <em class="usa-logo__text display-flex flex-align-center">
            <nuxt-link
              to="/"
              title="USAGov Logo"
              aria-label="USAGov Logo">
              <img
                class="circle-5 desktop:circle-10 margin-right-1"
                src="~/assets/img/logo-img-usagov.png"
                alt="USAGov Logo" />
            </nuxt-link>
            <nuxt-link
              to="/"
              title="Home"
              aria-label="Home"
              class="margin-left-1">
              {{ $t("projectName") }}
            </nuxt-link>
          </em>
        </div>
        <button class="usa-menu-btn print:display-none">{{ $t("header.menu") }}</button>
      </div>
      <nav
        aria-label="Primary navigation"
        class="usa-nav print:display-none">
        <div class="usa-nav__inner">
          <button class="usa-nav__close">
            <img
              src="~/assets/img/usa-icons/close.svg"
              alt="close mobile navigation" />
          </button>
          <div class="usa-nav__secondary">
            <ul class="usa-nav__secondary-links">
              <li class="usa-nav__secondary-item">
                <a href="https://www.usa.gov/phone">1-844-USA-GOV1</a>
              </li>
            </ul>
            <form
              class="usa-search"
              role="search"
              action="https://search.usa.gov/search"
              method="get"
              name="search_form"
              accept-charset="UTF-8">
              <label
                class="usa-sr-only"
                for="extended-search-field-small"
                >Search small</label
              >
              <input
                v-if="$i18n.locale === 'en'"
                id="affiliate"
                name="affiliate"
                type="hidden"
                value="usagov" />
              <input
                v-if="$i18n.locale === 'es'"
                id="affiliate"
                name="affiliate"
                type="hidden"
                value="gobiernousa_only" />
              <input
                id="search-field-small"
                type="search"
                name="query"
                :placeholder="$t('header.meta.placeholder')"
                onfocus="this.placeholder = ''"
                class="usa-input text usagov-search-autocomplete ui-autocomplete-input"
                autocomplete="off"
                aria-autocomplete="list"
                aria-haspopup="true" />
              <button
                class="usa-button"
                type="submit">
                <span class="usa-sr-only">{{ $t("header.meta.search") }}</span>
                <svg
                  class="usa-icon text-middle usa-icon--size-3"
                  aria-hidden="true"
                  focusable="false"
                  role="img">
                  <use xlink:href="~/assets/img/sprite.svg#search" />
                </svg>
              </button>
            </form>
          </div>
          <ul class="usa-nav__primary usa-accordion">
            <li
              v-if="isLifeEventPage"
              class="usa-nav__primary-item">
              <nuxt-link
                to="/"
                class="usa-nav__link">
                <span>{{ $t("utilityNav.linkOne") }}</span>
              </nuxt-link>
            </li>
            <li
              v-else
              class="usa-nav__primary-item">
              <nuxt-link
                :to="localePath('/')"
                exact
                class="usa-nav__link">
                <span>{{ $t("utilityNav.linkOne") }}</span>
              </nuxt-link>
            </li>
            <li class="usa-nav__primary-item">
              <nuxt-link
                :to="localePath('/types')"
                class="usa-nav__link">
                <span>{{ $t("utilityNav.linkTwo") }}</span>
              </nuxt-link>
            </li>
            <li class="usa-nav__primary-item">
              <nuxt-link
                :to="localePath('/agencies')"
                class="usa-nav__link">
                <span>{{ $t("utilityNav.linkThree") }}</span>
              </nuxt-link>
            </li>
            <li class="usa-nav__primary-item tablet:margin-left-auto">
              <button
                v-if="$config.languageToggleActive"
                id="language-toggle-button"
                class="language-toggle-mobile usa-button"
                @click="switchLanguage">
                {{ $t("header.meta.language") }}
              </button>
            </li>
          </ul>
        </div>
      </nav>
    </header>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isLifeEventPage: this.doesRouteMatchLifeEventPages(),
      query: this.$route?.query?.q || "",
    }
  },
  watch: {
    $route() {
      this.isLifeEventPage = this.doesRouteMatchLifeEventPages()
    },
  },
  methods: {
    switchLanguage() {
      let route = ""
      const locale = this.$i18n.locale
      const oneEventString = !this.$config.oneEventVersion ? "" : this.$config.oneEventVersion
      if (locale === "en") {
        route = `/es/${oneEventString}`
        this.$i18n.setLocale("es")
      } else {
        route = `/${oneEventString}`
        this.$i18n.setLocale("en")
      }
      this.$router.push(route)
    },
    doesRouteMatchLifeEventPages() {
      /* istanbul ignore next */
      return (
        this.$route &&
        this.$route.matched &&
        this.$route.matched[0] &&
        this.$route.matched[0].path &&
        this.$route.matched[0].path === "/:slug"
      )
    },
  },
}
</script>
<style
  lang="scss"
  scoped>
// Primary Colors
$dark-blue: #11385b; //NOTE not currently in our code
$aqua-blue: #02bfe7; //highlights action items and key areas (buttons, borders, etc)
$white: #ffffff; //site background, text on dark
// Supporting Colors
$dark-gray: #4b4b4d; //footer primary
$light-gray: #d9d9d9; //footer secondary
$beige: #ebe6de; //leftnav
$red: #c61f0c; //h1, leftnav current item, featurebox h2
$blue: #154285; //standard links, h2, leftnav header
$black: #000000; //standard text, h3
// Other Colors
$h3: $dark-gray;
$button-text: $black;
$aqua-darker: #00a6d2; //hover
$lang-toggle: $dark-blue;
$AZ-button: $blue;
$AZ-button-disabled: #859cba;
.va-middle {
  vertical-align: middle;
}
.language-toggle-mobile {
  color: $white !important;
  text-decoration: none;
  padding: 5px 20px 5px 20px;
  background: $dark-blue;
  font-size: 90%;
}
</style>
