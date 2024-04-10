<script>
import { mapState, mapWritableState } from 'pinia';
import BasePanel from '~/components/base/BasePanel.vue';
import { useAuthStore } from '~/store/auth';
import { useFormStore } from '~/store/form';
import { IdentityMode, IdentityProviders } from '~/utils/constants';
import { formService } from '~/services';

export default {
  components: {
    BasePanel,
  },
  data() {
    return {
      githubLinkBulkUpload:
        'https://github.com/bcgov/common-hosted-form-service/wiki/Allow-multiple-draft-upload',
      githubLinkCopyFromExistingFeature:
        'https://github.com/bcgov/common-hosted-form-service/wiki/Copy-an-existing-submission',
      githubLinkScheduleAndReminderFeature:
        'https://github.com/bcgov/common-hosted-form-service/wiki/Schedule-and-Reminder-notification',
      githubLinkEventSubscriptionFeature:
        'https://github.com/bcgov/common-hosted-form-service/wiki/Event-Subscription',
      githubLinkWideFormLayout:
        'https://github.com/bcgov/common-hosted-form-service/wiki/Wide-Form-Layout',
      cdogsTemplateDocumentation:
        'https://developer.gov.bc.ca/docs/default/component/chefs-techdocs/Capabilities/Functionalities/CDOGS-Template-Upload/',
      showDropbox: false,
      showTemplateFile: false,
      cdogsTemplateId: '',
      cdogsTemplateFilename: '',
      cdogsTemplateDate: '',
    };
  },
  computed: {
    ...mapState(useAuthStore, ['identityProvider']),
    ...mapState(useFormStore, ['isFormPublished', 'isRTL', 'lang']),
    ...mapWritableState(useFormStore, ['form']),
    ID_MODE() {
      return IdentityMode;
    },
    idirUser() {
      return this.identityProvider === IdentityProviders.IDIR;
    },
  },
  mounted() {
    if (this.form.enableDocumentTemplates && this.form.id) {
      this.fetchCdogsTemplate();
      this.showTemplateFile = true;
    }
  },
  methods: {
    enableSubmitterDraftChanged() {
      if (!this.form.enableSubmitterChanged) {
        this.form.allowSubmitterToUploadFile = false;
      }
    },
    allowSubmitterToUploadFileChanged() {
      if (
        this.form.allowSubmitterToUploadFile &&
        !this.form.enableSubmitterDraft
      ) {
        this.form.enableSubmitterDraft = true;
      }
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const formStore = useFormStore();
      formStore.setTemplateFile(file);
    },
    enableDropbox() {
      this.showDropbox = !this.showDropbox;
    },
    async fetchCdogsTemplate() {
      if (this.form.id && this.form.enableDocumentTemplates) {
        const response = await formService.documentTemplateList(this.form.id);
        this.cdogsTemplateId = response.data[0].id;
        this.cdogsTemplateFilename = response.data[0].filename;
        this.cdogsTemplateDate = response.data[0].createdAt.split('T')[0];
      }
    },
    async handleDelete() {
      await formService.documentTemplateDelete(
        this.form.id,
        this.cdogsTemplateId
      );
      this.cdogsTemplateId = '';
      this.cdogsTemplateFilename = '';
      this.cdogsTemplateDate = '';
      this.showDropbox = true;
      this.showTemplateFile = false;
    },
  },
};
</script>

<template>
  <BasePanel class="fill-height">
    <template #title
      ><span :lang="lang">{{
        $t('trans.formSettings.formFunctionality')
      }}</span></template
    >
    <v-checkbox
      v-model="form.enableSubmitterDraft"
      hide-details="auto"
      class="my-0"
      :disabled="form.userType === ID_MODE.PUBLIC"
      @update:model-value="enableSubmitterDraftChanged"
    >
      <template #label>
        <span
          :class="{ 'mr-2': isRTL }"
          :lang="lang"
          v-html="$t('trans.formSettings.canSaveAndEditDraftLabel')"
        ></span>
      </template>
    </v-checkbox>

    <v-checkbox
      v-model="form.enableStatusUpdates"
      hide-details="auto"
      class="my-0"
    >
      <template #label>
        <span
          :class="{ 'mr-2': isRTL }"
          :lang="lang"
          v-html="$t('trans.formSettings.canUpdateStatusAsReviewer')"
        ></span>
      </template>
    </v-checkbox>

    <v-checkbox
      v-model="form.allowSubmitterToUploadFile"
      hide-details="auto"
      class="my-0"
      :disabled="form.userType === ID_MODE.PUBLIC"
      @update:model-value="allowSubmitterToUploadFileChanged"
    >
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span
            :lang="lang"
            v-html="$t('trans.formSettings.allowMultiDraft')"
          />
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-flask"
              />
            </template>
            <span :lang="lang"
              >{{ $t('trans.formSettings.experimental') }}
              <a
                :href="githubLinkBulkUpload"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>

    <v-checkbox
      v-if="!isFormPublished"
      v-model="form.schedule.enabled"
      disabled
      hide-details="auto"
      class="my-0"
    >
      <template #label>
        <span :class="{ 'mr-2': isRTL }" :lang="lang"
          >{{ $t('trans.formSettings.formSubmissinScheduleMsg') }}
        </span>
      </template>
    </v-checkbox>

    <v-checkbox
      v-if="isFormPublished"
      v-model="form.schedule.enabled"
      hide-details="auto"
      class="my-0"
    >
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span :lang="lang">{{
            $t('trans.formSettings.formSubmissionsSchedule')
          }}</span>
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-flask"
              ></v-icon>
            </template>
            <span :lang="lang"
              >{{ $t('trans.formSettings.experimental') }}
              <a
                :href="githubLinkScheduleAndReminderFeature"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>

    <v-checkbox
      v-model="form.enableCopyExistingSubmission"
      hide-details="auto"
      class="my-0"
      :disabled="form.userType === ID_MODE.PUBLIC"
    >
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span
            style="max-width: 80%"
            :lang="lang"
            v-html="$t('trans.formSettings.submitterCanCopyExistingSubmissn')"
          />
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-flask"
              ></v-icon>
            </template>
            <span :lang="lang"
              >{{ $t('trans.formSettings.experimental') }}
              <a
                :href="githubLinkCopyFromExistingFeature"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>
    <v-checkbox
      v-model="form.subscribe.enabled"
      hide-details="auto"
      class="my-0"
      :disabled="idirUser === false || !isFormPublished"
    >
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span
            style="max-width: 80%"
            :lang="lang"
            v-html="$t('trans.formSettings.allowEventSubscription')"
          />
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-flask"
              ></v-icon>
            </template>
            <span :lang="lang"
              >{{ $t('trans.formSettings.experimental') }}
              <a
                :href="githubLinkEventSubscriptionFeature"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>
    <v-checkbox v-model="form.wideFormLayout" hide-details="auto" class="my-0">
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span
            style="max-width: 80%"
            :lang="lang"
            v-html="$t('trans.formSettings.wideFormLayout')"
          />
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-flask"
              ></v-icon>
            </template>
            <span :lang="lang"
              >{{ $t('trans.formSettings.experimental') }}
              <a
                :href="githubLinkWideFormLayout"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>
    <v-checkbox v-model="form.enableDocumentTemplates" @click="enableDropbox">
      <template #label>
        <div :class="{ 'mr-2': isRTL }">
          <span
            style="max-width: 80%"
            :lang="lang"
            v-html="$t('trans.printOptions.uploadCDOGSTemplate')"
          />
          <v-tooltip location="bottom">
            <template #activator="{ props }">
              <v-icon
                color="primary"
                class="ml-3"
                :class="{ 'mr-2': isRTL }"
                v-bind="props"
                icon="mdi:mdi-help-circle-outline"
              ></v-icon>
            </template>
            <span :lang="lang">
              <a
                :href="cdogsTemplateDocumentation"
                class="preview_info_link_field_white"
                :target="'_blank'"
                :hreflang="lang"
              >
                {{ $t('trans.formSettings.learnMore') }}
                <v-icon
                  icon="mdi:mdi-arrow-top-right-bold-box-outline"
                ></v-icon></a
            ></span>
          </v-tooltip>
        </div>
      </template>
    </v-checkbox>
    <v-window v-if="form.enableDocumentTemplates" direction="vertical">
      <v-window-item v-if="showDropbox">
        <v-file-input
          :class="[{ label: isRTL }]"
          :style="isRTL ? { gap: '10px' } : null"
          counter
          :clearable="true"
          :label="$t('trans.printOptions.uploadTemplateFile')"
          required
          mandatory
          show-size
          :lang="lang"
          @change="handleFileUpload($event)"
        />
      </v-window-item>
      <v-window-item>
        <v-table
          style="color: gray; border: 1px solid lightgray; border-radius: 8px"
          class="mb-5 mt-3 mx-10"
        >
          <thead>
            <tr>
              <th class="text-left">
                {{ $t('trans.printOptions.fileName') }}
              </th>
              <th class="text-left">
                {{ $t('trans.printOptions.uploadDate') }}
              </th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ cdogsTemplateFilename }}</td>
              <td>{{ cdogsTemplateDate }}</td>
              <td>
                <v-tooltip location="bottom">
                  <template #activator="{ props }">
                    <v-icon
                      color="red"
                      v-bind="props"
                      v-on="on"
                      @click="handleDelete"
                    >
                      mdi-delete-outline
                    </v-icon>
                  </template>
                  <span>Delete</span>
                </v-tooltip>
              </td>
            </tr>
          </tbody>
        </v-table>
      </v-window-item>
    </v-window>
  </BasePanel>
</template>

<style scoped>
.test {
  background-color: red;
  margin-bottom: 0;
}
</style>
