<script lang="ts" setup>
import * as XLSX from 'xlsx'
import { ExportTypes } from 'nocodb-sdk'
import FileSaver from 'file-saver'
import { message } from 'ant-design-vue'
import { useI18n } from 'vue-i18n'
import {
  ActiveViewInj,
  FieldsInj,
  IsLockedInj,
  IsPublicInj,
  MetaInj,
  extractSdkResponseErrorMsg,
  inject,
  ref,
  useNuxtApp,
  useProject,
  useUIPermission,
} from '#imports'
const { t } = useI18n()

const sharedViewListDlg = ref(false)

const isPublicView = inject(IsPublicInj, ref(false))

const isView = false

const { project } = useProject()

const { $api } = useNuxtApp()

const meta = inject(MetaInj)

const fields = inject(FieldsInj, ref([]))

const selectedView = inject(ActiveViewInj)

const { sorts, nestedFilters } = useSmartsheetStoreOrThrow()

const isLocked = inject(IsLockedInj)

const showWebhookDrawer = ref(false)

const quickImportDialog = ref(false)

const { isUIAllowed } = useUIPermission()

const exportFile = async (exportType: ExportTypes) => {
  let offset = 0
  let c = 1
  const responseType = exportType === ExportTypes.EXCEL ? 'base64' : 'blob'

  try {
    while (!isNaN(offset) && offset > -1) {
      let res
      if (isPublicView.value) {
        const { exportFile: sharedViewExportFile } = useSharedView()
        res = await sharedViewExportFile(fields.value, offset, exportType, responseType)
      } else {
        res = await $api.dbViewRow.export(
          'noco',
          project?.value.title as string,
          meta?.value.title as string,
          selectedView?.value.title as string,
          exportType,
          {
            responseType,
            query: {
              fields: fields.value.map((field) => field.title),
              offset,
              sortArrJson: JSON.stringify(sorts.value),
              filterArrJson: JSON.stringify(nestedFilters.value),
            },
          } as any,
        )
      }
      const { data, headers } = res
      if (exportType === ExportTypes.EXCEL) {
        const workbook = XLSX.read(data, { type: 'base64' })
        XLSX.writeFile(workbook, `${meta?.value.title}_exported_${c++}.xlsx`)
      } else if (exportType === ExportTypes.CSV) {
        const blob = new Blob([data], { type: 'text/plain;charset=utf-8' })
        FileSaver.saveAs(blob, `${meta?.value.title}_exported_${c++}.csv`)
      }
      offset = +headers['nc-export-offset']
      if (offset > -1) {
        // Downloading more files
        message.info(t('msg.info.downloadingMoreFiles'))
      } else {
        // Successfully exported all table data
        message.success(t('msg.success.tableDataExported'))
      }
    }
  } catch (e: any) {
    message.error(await extractSdkResponseErrorMsg(e))
  }
}
</script>

<template>
  <div>
    <a-dropdown>
      <a-button v-t="['c:actions']" class="nc-actions-menu-btn nc-toolbar-btn">
        <div class="flex gap-1 items-center">
          <MdiFlashOutline />

          <!-- More -->
          <span class="!text-sm font-weight-medium">{{ $t('general.more') }}</span>

          <MdiMenuDown class="text-grey" />
        </div>
      </a-button>

      <template #overlay>
        <div class="bg-gray-50 py-2 shadow-lg !border">
          <div>
            <div v-t="['a:actions:download-csv']" class="nc-menu-item" @click="exportFile(ExportTypes.CSV)">
              <MdiDownloadOutline class="text-gray-500" />
              <!-- Download as CSV -->
              {{ $t('activity.downloadCSV') }}
            </div>

            <div v-t="['a:actions:download-excel']" class="nc-menu-item" @click="exportFile(ExportTypes.EXCEL)">
              <MdiDownloadOutline class="text-gray-500" />
              <!-- Download as XLSX -->
              {{ $t('activity.downloadExcel') }}
            </div>

            <div
              v-if="isUIAllowed('csvImport') && !isView && !isPublicView"
              v-t="['a:actions:upload-csv']"
              class="nc-menu-item"
              :class="{ disabled: isLocked }"
              @click="!isLocked ? (quickImportDialog = true) : {}"
            >
              <MdiUploadOutline class="text-gray-500" />
              <!-- Upload CSV -->
              {{ $t('activity.uploadCSV') }}
            </div>

            <div
              v-if="isUIAllowed('sharedViewList') && !isView && !isPublicView"
              v-t="['a:actions:shared-view-list']"
              class="nc-menu-item"
              @click="sharedViewListDlg = true"
            >
              <MdiViewListOutline class="text-gray-500" />
              <!-- Shared View List -->
              {{ $t('activity.listSharedView') }}
            </div>
            <div
              v-if="isUIAllowed('webhook') && !isView && !isPublicView"
              v-t="['c:actions:webhook']"
              class="nc-menu-item"
              @click="showWebhookDrawer = true"
            >
              <MdiHook class="text-gray-500" />
              {{ $t('objects.webhooks') }}
            </div>
          </div>
        </div>
      </template>
    </a-dropdown>

    <DlgQuickImport v-if="quickImportDialog" v-model="quickImportDialog" import-type="csv" :import-only="true" />

    <WebhookDrawer v-if="showWebhookDrawer" v-model="showWebhookDrawer" />

    <a-modal v-model:visible="sharedViewListDlg" :title="$t('activity.listSharedView')" width="max(900px,60vw)" :footer="null">
      <SmartsheetToolbarSharedViewList v-if="sharedViewListDlg" />
    </a-modal>
  </div>
</template>
