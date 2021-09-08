<template>
  <SmartModal v-if="show" @close="hideModal" :title="$t('import_openapi')">
    <template #body>
      <div
        class="flex flex-col px-2"
      >
        <input
          id="serverUrlEdit"
          v-model="serverURL"
          class="input"
          placeholder=" "
          type="text"
          autocomplete="off"
        />
        <label for="serverUrlEdit">
          {{ $t("server_url") }}
        </label>
      </div>
      <label>Requests</label>
      <div
        v-for="(request, index) in imported.requests"
        :key="`${request.name}_${index}`"
        class="divide-x divide-dividerLight border-b border-dividerLight flex"
      >
          <input class="input bg-transparent flex flex-1 py-2 px-4" :value="request.name" disabled />
        <span>
          <ButtonSecondary
            v-tippy="{ theme: 'tooltip' }"
            :title="
                  importedRequests[index] === true
                    ? $t('turn_off')
                    : $t('turn_on')
            "
            :svg="
              importedRequests[index] === true
                  ? 'check-circle'
                  : 'circle'
            "
            color="green"
            @click.native="
              () => {
                  toggleEnabled(index)
                }
            "
          />
        </span>
      </div>
    </template>
    <template #footer>
      <span>
        <ButtonPrimary
          :label="$t('action.save')"
          @click.native="importCollection"
        />
        <ButtonSecondary
          :label="$t('action.cancel')"
          @click.native="hideModal"
        />
      </span>
    </template>
  </SmartModal>
</template>

<script>
import cloneDeep from "lodash/cloneDeep"
import { appendRESTCollections } from "~/newstore/collections"

export default {
  props: {
    show: Boolean,
    imported: {
      type: Object,
      default: () => ({
        requests: [],
      }),
    },
  },
  data() {
    return {
      importedRequests: this.imported.requests.map((_) => true),
      serverURL: "",
    }
  },
  watch: {
    imported(newVal, oldVal) {
      if (newVal.requests.length !== oldVal.requests.length)
        this.importedRequests = newVal.requests.map((_) => true)
    },
  },
  methods: {
    hideModal() {
      this.serverURL = ""
      this.$emit("hide-modal")
    },
    toggleEnabled(ind) {
      this.importedRequests = this.importedRequests.map((val, index) =>
        ind === index ? !val : val
      )
    },
    importCollection() {
      const collectionAdded = cloneDeep(this.imported)
      collectionAdded.requests = collectionAdded.requests
        .filter((_, i) => this.importedRequests[i])
        .map((req) => ({
          ...req,
          url: this.serverURL,
        }))
      console.log({ collectionAdded })
      appendRESTCollections([collectionAdded])
      this.hideModal()
    },
  },
}
</script>
