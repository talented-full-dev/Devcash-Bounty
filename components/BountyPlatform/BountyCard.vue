<template>
  <nuxt-link
    :to="localePath({name:'bountyplatform-bounty-id', params: {id: bounty.id}})"
    :class="{
      'bg-c-background-sec border-c-background-sec': !type,
      'bg-c-background-qua border-c-background-qua': type == 'secondary',
    } "
    class="w-full flex flex-row flex-wrap justify-between items-center shadow-lg border-2 hover:border-c-primary focus:border-c-primary relative rounded-tl-3xl rounded-br-3xl rounded-tr-lg rounded-bl-lg pt-4 pb-5 px-2 md:px-4 transition-colors duration-300 ease-out"
  >
    <!-- Bounty Name and Address -->
    <div class="w-full md:w-3/7 flex flex-col flex-wrap justify-center items-start px-4">
      <div class="flex flex-row flex-wrap items-center">
        <h4 class="font-extrabold text-xl text-left mr-3">{{ bounty.title }}</h4>
        <!-- Status Badge -->
        <div
          v-if="status == 'completed' || status == 'expired'"
          :class="status == 'expired'?'text-c-pending bg-c-pending-10 border-c-pending-40':'text-c-success bg-c-success-10 border-c-success-40'"
          class="border text-sm font-bold px-3 py-0_5 rounded-full my-1_5"
        >{{isReclaimable ? $t("bountyPlatform.bountyCard.tag.reclaimable"): status=="expired" ?$t("bountyPlatform.bountyCard.tag.expired"):status=="completed"?$t("bountyPlatform.bountyCard.tag.completed"):''}}</div>
      </div>
      <div class="flex flex-row items-center mt-1">
        <Jazzicon class="flex" :diameter="20" :address="bounty.bountyChest" />
        <h5 class="font-mono-jet text-md text-left ml-2 opacity-75">
          <span class="font-medium">
            {{
            bounty.bountyChest.substring(0, 6) +
            "..." +
            bounty.creator.substring(bounty.bountyChest.length - 4)
            }}
          </span>
        </h5>
      </div>
    </div>
    <!-- Divider -->
    <div class="bg-text md:hidden w-full h-px rounded-full opacity-5 my-3"></div>
    <!-- Submissions Left and Remaining Time -->
    <div
      class="w-full md:w-2/7 flex flex-col justify-center order-last md:order-none items-start md:items-end px-4"
    >
      <!-- Submissions Left -->
      <div class="text-right text-sm">
        <Icon class="w-4 h-4 mb-0_5 mr-1 inline-block" colorClass="text-c-text" type="award" />
        <span class="font-bold">{{`${bounty.available}` }}</span>
        <span class="opacity-75">
          {{
          $t("bountyPlatform.bountyCard.bountiesLeft")
          }}
        </span>
      </div>
      <!-- Remaining Time -->
      <div class="text-right text-sm mt-2">
        <Icon class="w-4 h-4 mb-0_5 mr-1 inline-block" colorClass="text-c-text" type="clock" />
        <span v-if="status=='active'" class="font-bold">{{ formatTimeLeft() }}</span>
        <span
          v-else-if="status=='expired'"
          class="font-bold"
        >{{ $t('bountyPlatform.bountyCard.tag.expired') }}</span>
        <span v-else class="font-bold">{{ $t('bountyPlatform.bountyCard.tag.completed') }}</span>
        <span v-if="status == 'active'" class="opacity-75">
          {{
          $t("bountyPlatform.bountyCard.remaining")
          }}
        </span>
      </div>
    </div>
    <!-- Price in Devcash, Ethereum and Dollars -->
    <div class="w-full md:w-2/7 flex flex-col justify-center items-start md:items-end px-4">
      <h4
        class="text-c-primary font-extrabold text-xl text-left md:text-right"
      >{{$store.state.devcashData.balancePrimary.symbol}}{{$store.state.devcashData.ethIsPrimary?ethAmount:amount}}</h4>
      <h5
        class="text-lg text-left md:text-right mt-1"
      >+ {{$store.state.devcashData.balanceSecondary.symbol}}{{$store.state.devcashData.ethIsPrimary?amount:ethAmount}}</h5>
    </div>
    <!-- Divider -->
    <div class="bg-c-text md:hidden w-full h-px rounded-full opacity-5 my-3"></div>
  </nuxt-link>
</template>
<script>
import Icon from "~/components/Icon.vue";
import Jazzicon from "~/components/Jazzicon.vue";
import { mapGetters } from "vuex";
import { utils } from 'ethers'
import { DevcashBounty } from "~/plugins/devcash/devcashBounty.client"
export default {
  components: {
    Icon,
    Jazzicon
  },
  props: {
    bounty: null,
    type: null
  },
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters({
      isLoggedIn: "devcashData/isLoggedIn",
      loggedInAccount: "devcashData/loggedInAccount"
    }),
    status() {
      if (new Date().getTime() / 1000 >= this.bounty.deadline) {
        return "expired"
      } else  if (this.bounty.submissions.filter(sub => sub.status == 'approved').length >= this.bounty.available) {
        return"completed"
      }
      return "active"
    },
    amount() {
      let tokenDecimals = 8
      if (this.$store.state.devcash.connector) {
          tokenDecimals = this.$store.state.devcash.connector.tokenDecimals
      }   
      return DevcashBounty.formatAmountSingleSubmission(this.bounty, tokenDecimals)      
    },
    ethAmount() {
      return DevcashBounty.formatAmountSingleSubmissionEth(this.bounty)
    },
    isReclaimable() {
      if (this.$store.state.devcashData.pendingReclaim.includes(this.bounty.id)) {
        return false
      }
      return (this.isLoggedIn && this.bounty && this.bounty.creator.toLowerCase() == this.loggedInAccount.toLowerCase() && this.bounty.available > 0 && this.status == 'expired')
    }
  },
  methods: {  
    formatTimeLeft() {
      return DevcashBounty.formatTimeLeft(this.bounty)
    } 
  }
};
</script>
