<script>
import { useAlert } from 'dashboard/composables';
import {
  DuplicateContactException,
  ExceptionWithMessage,
} from 'shared/helpers/CustomErrors';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: {
    NextButton,
  },
  props: {
    name: {
      type: String,
      default: '',
    },
    phoneNumber: {
      type: String,
      default: '',
    },
  },
  computed: {
    formattedPhoneNumber() {
      return this.phoneNumber.replace(/\s|-|[A-Za-z]/g, '');
    },
    rawPhoneNumber() {
      return this.phoneNumber.replace(/\D/g, '');
    },
  },
  methods: {
    async addContact() {
      try {
        let contact = await this.filterContactByNumber(this.rawPhoneNumber);
        if (!contact) {
          contact = await this.$store.dispatch(
            'contacts/create',
            this.getContactObject()
          );
          useAlert(this.$t('CONTACT_FORM.SUCCESS_MESSAGE'));
        }
        this.openContactNewTab(contact.id);
      } catch (error) {
        if (error instanceof DuplicateContactException) {
          if (error.data.includes('phone_number')) {
            useAlert(this.$t('CONTACT_FORM.FORM.PHONE_NUMBER.DUPLICATE'));
          }
        } else if (error instanceof ExceptionWithMessage) {
          useAlert(error.data);
        } else {
          useAlert(this.$t('CONTACT_FORM.ERROR_MESSAGE'));
        }
      }
    },
    getContactObject() {
      const contactItem = {
        name: this.name,
        phone_number: `+${this.rawPhoneNumber}`,
      };
      return contactItem;
    },
    async filterContactByNumber(phoneNumber) {
      const query = {
        attribute_key: 'phone_number',
        filter_operator: 'equal_to',
        values: [phoneNumber],
        attribute_model: 'standard',
        custom_attribute_type: '',
      };

      const queryPayload = { payload: [query] };
      const contacts = await this.$store.dispatch('contacts/filter', {
        queryPayload,
        resetState: false,
      });
      return contacts.shift();
    },
    openContactNewTab(contactId) {
      const accountId = window.location.pathname.split('/')[3];
      const url = `/app/accounts/${accountId}/contacts/${contactId}`;
      window.open(url, '_blank');
    },
  },
};
</script>

<template>
  <div class="contact--group">
    <fluent-icon icon="call" class="file--icon" size="18" />
    <div class="meta">
      <p
        class="overflow-hidden whitespace-nowrap text-ellipsis margin-bottom-0"
      >
        {{ phoneNumber }}
      </p>
    </div>
    <div v-if="formattedPhoneNumber" class="link-wrap">
      <NextButton
        ghost
        xs
        :label="$t('CONVERSATION.SAVE_CONTACT')"
        @click.prevent="addContact"
      />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.contact--group {
  align-items: center;
  display: flex;
  margin-top: var(--space-smaller);

  .meta {
    flex: 1;
    margin-left: var(--space-small);
  }

  .link-wrap {
    margin-left: var(--space-small);
  }
}
</style>
