<template>
  <div class="flex flex-col transition-all duration-300 ease-out">
    <div
      class="bg-c-background-sec border-c-text-10 shadow-lg w-full flex flex-col flex-wrap justify-between items-center border rounded-lg overflow-hidden"
    >
      <!-- Top Part -->
      <div class="bg-c-text-05 w-full flex flex-row flex-wrap justify-between items-center py-2">
        <!-- Senders Address and Project Name -->
        <div class="w-full md:w-auto flex flex-row md:items-center px-4 py-2">
          <Jazzicon class="flex m-1" :diameter="20" :address="submission.creator" />
          <h5 class="text-left ml-2 mr-3">
            <a
              :href="'https://etherscan.io/address/'+submission.creator"
              class="hover:underline"
              target="_blank"
            >
              <span
                class="font-mono-jet font-bold"
              >{{submission.creator.substring(0, 6) + "..." + submission.creator.substring(submission.creator.length - 4)}}</span>
            </a>
            <span v-if="ubounty.title">-></span>
            <nuxt-link
              :to="localePath({name:'bountyplatform-bounty-id', params: {id: ubounty.id}})"
              v-if="ubounty.title && (page=='bountyHunter' || page == 'bountyManager')"
              class="font-extrabold hover:underline"
            >{{ubounty.title}}</nuxt-link>
            <span v-else-if="ubounty.title" class="font-extrabold">{{ubounty.title}}</span>
          </h5>
        </div>
        <!-- Bounty Amount and Status Tag or Approve Reject Options -->
        <div class="w-full md:w-auto flex flex-row flex-wrap items-center px-4 md:pr-0">
          <!-- Bounty Amount in DEV, ETH and USD -->
          <a
            v-if="submission.status== 'pending' || (submission.blockHash && submission.status == 'approved')"
            :href="submission.status== 'pending'?'https://etherscan.io/address/'+ubounty.bountyChest:'https://etherscan.io/tx/'+submission.blockHash"
            class="hover:underline flex flex-col my-2"
            target="_blank"
          >
            <h5
              :class="{
              'text-c-pending': (submission.status == 'pending'),
              'text-c-success': submission.status == 'approved',
              'text-c-danger': submission.status == 'rejected'}"
              class="font-bold text-left"
            >{{$store.state.devcashData.balancePrimary.symbol}}{{$store.state.devcashData.ethIsPrimary?ethAmount:amount}}</h5>
            <h6
              class="text-sm text-left mt-1"
            >+ {{$store.state.devcashData.balanceSecondary.symbol}}{{$store.state.devcashData.ethIsPrimary?amount:ethAmount}}</h6>
          </a>
          <!-- Bounty Amount in DEV, ETH and USD if there it's not pending and it doesn't have a block hash -->
          <div v-else class="flex flex-col my-2">
            <h5
              :class="{
              'text-c-pending': (submission.status == 'pending'),
              'text-c-success': submission.status == 'approved',
              'text-c-danger': submission.status == 'rejected'}"
              class="font-bold text-left"
            >{{$store.state.devcashData.balancePrimary.symbol}}{{$store.state.devcashData.ethIsPrimary?ethAmount:amount}}</h5>
            <h6
              class="text-sm text-left mt-1"
            >+ {{$store.state.devcashData.balanceSecondary.symbol}}{{$store.state.devcashData.ethIsPrimary?amount:ethAmount}}</h6>
          </div>
          <!-- Status Tag or Approve and Reject Buttons -->
          <div class="flex flex-col md:mx-4 my-2">
            <!-- If Perspective is Manager -->
            <div
              v-if="perspective=='manager' && submission.status == 'pending' && !isPendingApproved() && !isPendingRejection()"
              class="flex flex-row flex-wrap items-center"
            >
              <!-- Approve Button -->
              <button
                @click="approveClicked"
                class="hover:bg-c-success-15 focus:bg-c-success-15 flex flex-row items-center transition-all rounded-full duration-200 ease-out px-3 py-1 mx-1 my-1"
              >
                <Icon class="w-5 h-5" type="done" colorClass="text-c-success" />
                <h5
                  class="text-c-success font-extrabold mx-1"
                >{{$t('bountyPlatform.bountyManager.approve')}}</h5>
              </button>
              <!-- Reject Button -->
              <button
                @click="rejectClicked"
                class="hover:bg-c-danger-15 focus:bg-c-danger-15 flex flex-row items-center transition-all rounded-full duration-200 ease-out px-3 py-1 mx-1 my-1"
              >
                <Icon class="w-5 h-5" type="cancel" colorClass="text-c-danger" />
                <h5
                  class="text-c-danger font-extrabold mx-1"
                >{{$t('bountyPlatform.bountyManager.reject')}}</h5>
              </button>
            </div>
            <!-- If Perspective is Hunter -->
            <StatusTag
              v-if="perspective=='hunter' || submission.status == 'approved' || submission.status == 'rejected'"
              :status="getSimpleStatus()"
            />
          </div>
        </div>
      </div>
      <!-- Thin Status Bar -->
      <StatusDivider :status="getSimpleStatus()" />
      <!-- Bottom Part -->
      <div class="w-full flex flex-col flex-wrap px-4 md:px-6 py-6">
        <!-- Message -->
        <div class="editor-content" v-html="submissionData"></div>
        <div class="flex flex-row justify-between items-end flex-wrap">
          <!-- Contact Name & Contact Email & Date -->
          <div
            class="bg-c-text-05 border border-c-text-10 rounded-lg flex flex-row flex-wrap items-start mt-8 mr-6"
          >
            <div class="flex flex-col px-4 py-2 mr-4">
              <p
                class="text-sm opacity-75"
              >{{$t("bountyPlatform.singleBounty.submissions.submissionDate")}}</p>
              <p class="font-bold mt-1">{{ formatDateStr(currentLocale.iso) }}</p>
            </div>
            <div v-if="submission.contactName" class="flex flex-col px-4 py-2 mr-4">
              <p
                class="text-sm opacity-75"
              >{{$t("bountyPlatform.singleBounty.submissions.contactName")}}</p>
              <p class="font-bold mt-1">{{ submission.contactName }}</p>
            </div>
            <div v-if="submission.contactEmail" class="flex flex-col px-4 py-2 mr-4">
              <p
                class="text-sm opacity-75"
              >{{$t("bountyPlatform.singleBounty.submissions.contactEmail")}}</p>
              <p class="font-bold mt-1">{{submission.contactEmail}}</p>
            </div>
          </div>
          <!-- Show Feedback -->
          <button
            v-if="submission.feedback"
            @click.prevent="isFeedbackVisible=!isFeedbackVisible"
            class="whitespace-no-wrap hover:bg-c-text-15 focus:bg-c-text-15 flex flex-row items-center transition-all rounded-full duration-200 ease-out px-3 py-1 mt-4"
          >
            <Icon
              :class="isFeedbackVisible?'-rotate-180':''"
              class="w-5 h-5 transform transition-all duration-300 ease-out"
              type="arrow-down"
              colorClass="text-c-text"
            />
            <h5
              class="text-c-text font-extrabold mx-1"
            >{{isFeedbackVisible?$t('bountyPlatform.bountyHunter.hideFeedback'):$t('bountyPlatform.bountyHunter.showFeedback')}}</h5>
          </button>
        </div>
      </div>
    </div>
    <!-- If there is feedback -->
    <transition name="feedbackTransition">
      <feedback-card
        v-if="isFeedbackVisible && submission.feedback"
        class="mt-4"
        :feedbackMessage="submission.feedback"
        :address="ubounty.creator"
      />
    </transition>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
import { DevcashBounty } from "~/plugins/devcash/devcashBounty.client";
import { utils } from "ethers";
import Icon from "~/components/Icon.vue";
import Jazzicon from "~/components/Jazzicon.vue";
import StatusTag from "~/components/BountyPlatform/StatusTag.vue";
import StatusDivider from "~/components/BountyPlatform/StatusDivider.vue";
import FeedbackCard from "~/components/BountyPlatform/FeedbackCard.vue";
import marked from 'marked'
export default {
  components: {
    Icon,
    Jazzicon,
    StatusTag,
    StatusDivider,
    FeedbackCard
  },
  props: {
    ubounty: Object,
    submission: Object,
    perspective: String,
    approveClicked: Function,
    rejectClicked: Function,
    page: String
  },
  data(){
    return {
      isFeedbackVisible: true
    }
  },
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters({
      isLoggedIn: "devcashData/isLoggedIn",
      loggedInAccount: "devcashData/loggedInAccount"
    }),
    currentLocale() {
      for (let locale of this.$i18n.locales) {
        if (locale.code == this.$i18n.locale) {
          return locale;
        }
      }
    },
    amount() {
      let tokenDecimals = 8
      if (this.$store.state.devcash.connector) {
          tokenDecimals = this.$store.state.devcash.connector.tokenDecimals
      }
      if (this.submission.overrideAmount) {
        return utils.formatUnits(this.submission.overrideAmount.devcash, 8)
      }
      return DevcashBounty.formatAmountSingleSubmission(this.ubounty, tokenDecimals)      
    },
    ethAmount() {
      if (this.submission.overrideAmount) {
        return utils.formatEther(this.submission.overrideAmount.wei)
      }      
      return DevcashBounty.formatAmountSingleSubmissionEth(this.ubounty)
    },
    submissionData() {
      let renderer = new marked.Renderer()
      renderer.link = function( href, title, text ) {
        return `<a target="_blank" href="${!href.startsWith('http://') && !href.startsWith('https://') ? `https://${href}` : href}" title="${title}">${text}</a>`;
      }
      return this.$sanitize(marked(this.submission.submission_data, {renderer: renderer}))
    }
  },
  methods: {
    isPendingRejection() {
      for (const pendingSub of this.$store.state.devcashData.pendingSubStatus) {
        if (pendingSub.type == 'reject' && pendingSub.bounty == this.ubounty.id && pendingSub.submission == this.submission.submission_id) {
          return true
        }
      }
      return false 
    },
    isPendingApproved() {
      for (const pendingSub of this.$store.state.devcashData.pendingSubStatus) {
        if (pendingSub.type == 'approve' && pendingSub.bounty == this.ubounty.id && pendingSub.submission == this.submission.submission_id) {
          return true
        }
      }
      return false 
    },
    formatDateStr(locale) {
      return DevcashBounty.formatDateStr(locale, this.submission.createdAt)
    },
    getSimpleStatus() {
      return this.submission.status
    }
  }  
};
</script>
<style>
.feedbackTransition-enter-active {
  transition: all 0.3s ease-out;
  transform-origin: top center;
}
.feedbackTransition-leave-active {
  transition: all 0.2s ease-out;
  transform-origin: top center;
}
.feedbackTransition-enter {
  opacity: 0;
  transform: scaleY(0.5) translateY(-1.5rem);
}
.feedbackTransition-leave-to {
  opacity: 0;
  transform: scaleY(0.5) translateY(-1.5rem);
}
</style>