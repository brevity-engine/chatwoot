<template>
  <modal :show.sync="show" :on-close="onClose">
    <div class="column content-box">
      <woot-modal-header
        :header-title="$t('CAMPAIGN.ADD.TITLE')"
        :header-content="$t('CAMPAIGN.ADD.DESC')"
      />
      <form class="row" @submit.prevent="addCampaign">
        <div class="medium-12 columns">
          <label :class="{ error: $v.title.$error }">
            {{ $t('CAMPAIGN.ADD.FORM.TITLE.LABEL') }}
            <input
              v-model.trim="title"
              type="text"
              :placeholder="$t('CAMPAIGN.ADD.FORM.TITLE.PLACEHOLDER')"
              @input="$v.title.$touch"
            />
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ error: $v.message.$error }">
            {{ $t('CAMPAIGN.ADD.FORM.MESSAGE.LABEL') }}
            <textarea
              v-model.trim="message"
              rows="5"
              type="text"
              :placeholder="$t('CAMPAIGN.ADD.FORM.MESSAGE.PLACEHOLDER')"
              @input="$v.message.$touch"
            />
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ error: $v.selectedSender.$error }">
            {{ $t('CAMPAIGN.ADD.FORM.SENT_BY.LABEL') }}
            <select v-model="selectedSender">
              <option
                v-for="sender in sendersAndBotList"
                :key="sender.name"
                :value="sender.id"
              >
                {{ sender.name }}
              </option>
            </select>
            <span v-if="$v.selectedSender.$error" class="message">
              {{ $t('CAMPAIGN.ADD.FORM.SENT_BY.ERROR') }}
            </span>
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ error: $v.endPoint.$error }">
            {{ $t('CAMPAIGN.ADD.FORM.END_POINT.LABEL') }}
            <input
              v-model.trim="endPoint"
              type="text"
              :placeholder="$t('CAMPAIGN.ADD.FORM.END_POINT.PLACEHOLDER')"
              @input="$v.endPoint.$touch"
            />
            <span v-if="$v.endPoint.$error" class="message">
              {{ $t('CAMPAIGN.ADD.FORM.END_POINT.ERROR') }}
            </span>
          </label>
        </div>
        <div class="medium-12 columns">
          <label :class="{ error: $v.timeOnPage.$error }">
            {{ $t('CAMPAIGN.ADD.FORM.TIME_ON_PAGE.LABEL') }}
            <input
              v-model.trim="timeOnPage"
              type="number"
              :placeholder="$t('CAMPAIGN.ADD.FORM.TIME_ON_PAGE.PLACEHOLDER')"
              @input="$v.timeOnPage.$touch"
            />
            <span v-if="$v.timeOnPage.$error" class="message">
              {{ $t('CAMPAIGN.ADD.FORM.TIME_ON_PAGE.ERROR') }}
            </span>
          </label>
        </div>

        <div class="medium-12 columns">
          <label>
            <input
              v-model="enabled"
              type="checkbox"
              value="enabled"
              name="enabled"
            />
            {{ $t('CAMPAIGN.ADD.FORM.ENABLED') }}
          </label>
        </div>

        <div class="modal-footer">
          <div class="medium-12 columns">
            <woot-submit-button
              :disabled="buttonDisabled"
              :loading="uiFlags.isCreating"
              :button-text="$t('CAMPAIGN.ADD.CREATE_BUTTON_TEXT')"
            />
            <button class="button clear" @click.prevent="onClose">
              {{ $t('CAMPAIGN.ADD.CANCEL_BUTTON_TEXT') }}
            </button>
          </div>
        </div>
      </form>
    </div>
  </modal>
</template>

<script>
import { mapGetters } from 'vuex';
import { required, url, minLength } from 'vuelidate/lib/validators';
import Modal from 'dashboard/components/Modal';
import alertMixin from 'shared/mixins/alertMixin';

export default {
  components: {
    Modal,
  },
  mixins: [alertMixin],
  props: {
    senderList: {
      type: Array,
      default: () => [],
    },
    onClose: {
      type: Function,
      default: () => {},
    },
  },
  data() {
    return {
      title: '',
      message: '',
      selectedSender: 0,
      endPoint: '',
      timeOnPage: 10,
      show: true,
      enabled: true,
    };
  },
  validations: {
    title: {
      required,
    },
    message: {
      required,
    },
    selectedSender: {
      required,
    },
    endPoint: {
      required,
      minLength: minLength(7),
      url,
    },
    timeOnPage: {
      required,
    },
  },

  computed: {
    ...mapGetters({
      uiFlags: 'campaigns/getUIFlags',
    }),
    buttonDisabled() {
      return (
        this.$v.message.$invalid ||
        this.$v.title.$invalid ||
        this.$v.selectedSender.$invalid ||
        this.$v.endPoint.$invalid ||
        this.$v.timeOnPage.$invalid ||
        this.uiFlags.isCreating
      );
    },
    sendersAndBotList() {
      return [
        {
          id: 0,
          name: 'Bot',
        },
        ...this.senderList,
      ];
    },
  },

  methods: {
    async addCampaign() {
      try {
        await this.$store.dispatch('campaigns/create', {
          title: this.title,
          message: this.message,
          inbox_id: this.$route.params.inboxId,
          sender_id: this.selectedSender || null,
          enabled: this.enabled,
          trigger_rules: {
            url: this.endPoint,
            time_on_page: this.timeOnPage,
          },
        });
        this.showAlert(this.$t('CAMPAIGN.ADD.API.SUCCESS_MESSAGE'));
        this.onClose();
      } catch (error) {
        this.showAlert(this.$t('CAMPAIGN.ADD.API.ERROR_MESSAGE'));
      }
    },
  },
};
</script>
