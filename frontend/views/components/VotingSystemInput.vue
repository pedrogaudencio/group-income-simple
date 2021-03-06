<template lang='pug'>
  div
    slider-continuous.c-slider(
      :class='ephemeral.sliderClass'
      :uid='type'
      :label='config[type].slideLabel'
      :min='config[type].slideMin'
      :max='config[type].slideMax'
      :unit='config[type].slideUnit'
      :value='value'
      @input='handleInput'
    )
    transition-expand
      // inner el has paddings. A wrap is needed for a smooth transition.
      div(v-if='warnMajority')
        banner-simple.c-banner(severity='warning')
          i18n(
            :args='{ a_:`<a class="link" href="https://groupincome.org/2016/09/deprecating-mays-theorem/#when-majority-rule-can-harm" target="_blank">`, _a: "</a>" }'
          ) The percentage value you are choosing is most likely too low for a decision that can have a potentially significant impact on a person's life. Please consider using a {a_}supermajority threshold{_a}.
</template>

<script>
import L from '@view-utils/translations.js'
import { RULE_PERCENTAGE, RULE_DISAGREEMENT } from '@model/contracts/voting/rules.js'
import BannerSimple from '@components/banners/BannerSimple.vue'
import SliderContinuous from '@components/SliderContinuous.vue'
import TransitionExpand from '@components/TransitionExpand.vue'

const SUPERMAJORITY = 0.67

export default {
  name: 'VotingSystemInput',
  components: {
    BannerSimple,
    SliderContinuous,
    TransitionExpand
  },
  props: {
    type: {
      type: String,
      validator: (type) => [RULE_PERCENTAGE, RULE_DISAGREEMENT].includes(type)
    },
    value: [String, Number]
  },
  data: () => ({
    config: {
      [RULE_PERCENTAGE]: {
        slideLabel: L('What percentage of members need to agree?'),
        slideMin: 0,
        slideMax: 100,
        slideUnit: '%'
      },
      [RULE_DISAGREEMENT]: {
        slideLabel: L('Maximum number of "no" votes'),
        slideMin: 1,
        slideMax: 60,
        slideUnit: ''
      }
    },
    ephemeral: {
      sliderClass: ''
    }
  }),
  watch: {
    value (value) {
      this.ephemeral.sliderClass = this.warnMajority ? 'is-warning' : ''
    }
  },
  computed: {
    warnMajority () {
      return this.type === RULE_PERCENTAGE && this.value / 100 < SUPERMAJORITY
    }
  },
  methods: {
    handleInput (e) {
      this.$emit('update', e.target.value)
    }
  }
}
</script>

<style lang="scss" scoped>
@import "@assets/style/_variables.scss";

.c-slider {
  &.is-warning ::v-deep .slider {
    color: $warning_0;
  }
}

.c-banner {
  margin-top: 1.5rem;
}
</style>
