<template lang='pug'>
  ul.c-wrapper
    li.cardBox(
      v-for='type in votingSytemSorted'
      :class='{ isActive: isVotingActive(type) }'
      :aria-current='isVotingActive(type)'
    )
      p.is-title-4 {{ config[type].title }}
      p.has-text-1.c-expl(v-html='config[type].explanation')
      i18n.pill.is-primary.c-active(v-if='isVotingActive(type)') Active

      dl.c-status(v-if='isVotingActive(type)')
        dt.c-status-term {{ config[type].status }}
        dd.c-status-desc
          span(v-html='votingValue(type)')
          i18n.link(
            tag='button'
            @click='openVotingProposal(type)'
          ) Propose change

      banner-simple.c-banner(
        severity='info'
        v-if='isVotingRuleAdjusted(type)'
      )
        i18n \* This value was automatically adjusted because your group is currently smaller than the disagreement number.

      i18n.link(
        v-if='!isVotingActive(type)'
        tag='button'
        @click='openVotingProposal(type)'
      ) Propose changing to this voting system
</template>

<script>
import sbp from '~/shared/sbp.js'
import { RULE_PERCENTAGE, RULE_DISAGREEMENT } from '@model/contracts/voting/rules.js'
import { OPEN_MODAL } from '@utils/events.js'
import L from '@view-utils/translations.js'
import BannerSimple from '@components/banners/BannerSimple.vue'

export default {
  name: 'GroupVotingSystem',
  components: {
    BannerSimple
  },
  data: () => ({
    config: {
      [RULE_PERCENTAGE]: {
        title: L('Percentage based'),
        explanation: L('Proposals are accepted when the required percentage of members agree to the proposal.'),
        status: L('Percentage of members that need to agree:')
      },
      [RULE_DISAGREEMENT]: {
        title: L('Disagreement number'),
        explanation: L('Proposals are rejected when a certain number of members disagree with the proposal.'),
        status: L('Maximum number of "no" votes:')
      }
    }
  }),
  computed: {
    votingSytemSorted () {
      return [RULE_DISAGREEMENT, RULE_PERCENTAGE] // TODO sort order based on groupSettings.
    }
  },
  methods: {
    openVotingProposal (type) {
      sbp('okTurtles.events/emit', OPEN_MODAL, 'ChangeVotingProposal', { type })
    },
    isVotingActive (option) {
      /*
      TODO: This should be based on the value of store.getters.groupSettings.proposals[PROPOSAL_GROUP_SETTING_CHANGE].rule.

      Note: technically it can be based on the value of store.getters.groupSettings.proposals[PROPOSAL_INVITE_MEMBER].rule
      or store.getters.groupSettings.proposals[PROPOSAL_REMOVE_MEMBER].rule or any of the other proposal types,
      since we've decided that we are making it the same voting rule for all proposal types. */
      return option === this.votingSytemSorted[0]
    },
    isVotingRuleAdjusted (option) {
      if (!this.isVotingActive(option)) return false

      return option === RULE_DISAGREEMENT // TODO this
    },
    votingValue (option) {
      const HTMLTags = {
        b_: '<span class="has-text-bold">',
        _b: '</span>',
        sm_: '<span class="has-text-1 has-text-small">',
        _sm: '</span>'
      }

      if (option === RULE_PERCENTAGE) {
        const percent = '75%'
        return L('{b_}{percent}{_b} {sm_}({count} out of {total} members){_sm}', {
          percent,
          count: 8,
          total: 10,
          ...HTMLTags
        })
      } else {
        const count = 5
        const adjusted = 1

        if (count !== adjusted) {
          return L('{b_}{count}{_b} {sm_}(adjusted to {nr}*){_sm}', { count, nr: adjusted, ...HTMLTags })
        }
        return L('{b_}{count}{_b}', { count, ...HTMLTags })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import "@assets/style/_variables.scss";

.c-wrapper {
  margin-top: 1.5rem;
}

.c-active {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.c-expl {
  margin-bottom: 1.5rem;
}

.c-status {
  &-term,
  &-desc {
    display: inline;
  }

  &-term,
  &-desc > :first-child {
    margin-right: 0.5rem;
  }

  &-desc {
    .link {
      margin-top: 0.5rem;
    }

    @include phone {
      display: block;
    }
  }
}

.c-banner {
  margin-top: 1rem;
}

</style>
