<script lang="ts" setup>
import { isVirtualCol } from 'nocodb-sdk'
import { inject, provide, useViewData } from '#imports'
import Row from '~/components/smartsheet/Row.vue'
import type { Row as RowType } from '~/composables'
import {
  ActiveViewInj,
  ChangePageInj,
  FieldsInj,
  IsFormInj,
  IsGridInj,
  MetaInj,
  OpenNewRecordFormHookInj,
  PaginationDataInj,
  ReadonlyInj,
} from '~/context'
import ImageIcon from '~icons/mdi/file-image-box'

interface Attachment {
  url: string
}

const meta = inject(MetaInj)
const view = inject(ActiveViewInj)
const reloadViewDataHook = inject(ReloadViewDataHookInj)
const openNewRecordFormHook = inject(OpenNewRecordFormHookInj, createEventHook())

const expandedFormDlg = ref(false)
const expandedFormRow = ref<RowType>()
const expandedFormRowState = ref<Record<string, any>>()

const {
  loadData,
  paginationData,
  formattedData: data,
  loadGalleryData,
  galleryData,
  changePage,
  addEmptyRow,
} = useViewData(meta, view as any)

const { isUIAllowed } = useUIPermission()

provide(IsFormInj, ref(false))
provide(IsGridInj, false)
provide(PaginationDataInj, paginationData)
provide(ChangePageInj, changePage)
provide(ReadonlyInj, !isUIAllowed('xcDatatableEditable'))

const fields = inject(FieldsInj, ref([]))

const fieldsWithoutCover = computed(() => fields.value.filter((f) => f.id !== galleryData.value?.fk_cover_image_col_id))

const coverImageColumn: any = $(
  computed(() =>
    meta?.value.columnsById
      ? meta.value.columnsById[galleryData.value?.fk_cover_image_col_id as keyof typeof meta.value.columnsById]
      : {},
  ),
)

watch(
  [meta, view],
  async () => {
    if (meta?.value && view?.value) {
      await loadData()
      await loadGalleryData()
    }
  },
  { immediate: true },
)

const isRowEmpty = (record: any, col: any) => {
  const val = record.row[col.title]
  if (!val) return true

  return Array.isArray(val) && val.length === 0
}

const attachments = (record: any): Array<Attachment> => {
  try {
    return coverImageColumn?.title && record.row[coverImageColumn.title] ? JSON.parse(record.row[coverImageColumn.title]) : []
  } catch (e) {
    return []
  }
}

const reloadAttachments = ref(false)

reloadViewDataHook?.on(async () => {
  await loadData()
  await loadGalleryData()
  reloadAttachments.value = true
  nextTick(() => {
    reloadAttachments.value = false
  })
})

const expandForm = (row: RowType, state?: Record<string, any>) => {
  if (!isUIAllowed('xcDatatableEditable')) return
  expandedFormRow.value = row
  expandedFormRowState.value = state
  expandedFormDlg.value = true
}

const expandFormClick = async (e: MouseEvent, row: RowType) => {
  const target = e.target as HTMLElement
  if (target && !target.closest('.gallery-carousel')) {
    expandForm(row)
  }
}

openNewRecordFormHook?.on(async () => {
  const newRow = await addEmptyRow()
  expandForm(newRow)
})
</script>

<template>
  <div class="flex flex-col h-full w-full overflow-auto">
    <div class="nc-gallery-container grid gap-2 my-4 px-3">
      <div v-for="record in data" :key="`record-${record.row.id}`">
        <Row :row="record">
          <a-card
            hoverable
            class="!rounded-lg h-full overflow-hidden break-all max-w-[450px]"
            @click="expandFormClick($event, record)"
          >
            <template #cover>
              <a-carousel v-if="!reloadAttachments && attachments(record).length" autoplay class="gallery-carousel" arrows>
                <template #customPaging>
                  <a>
                    <div class="pt-[12px]"><div></div></div>
                  </a>
                </template>
                <template #prevArrow>
                  <div style="z-index: 1"></div>
                </template>
                <template #nextArrow>
                  <div style="z-index: 1"></div>
                </template>
                <img
                  v-for="(attachment, index) in attachments(record).filter((attachment) => attachment.url)"
                  :key="`carousel-${record.row.id}-${index}`"
                  class="h-52"
                  :src="attachment.url"
                />
              </a-carousel>
              <ImageIcon v-else class="w-full h-48 my-4 text-cool-gray-200" />
            </template>

            <div
              v-for="col in fieldsWithoutCover"
              :key="`record-${record.row.id}-${col.id}`"
              class="flex flex-col space-y-1 px-4 mb-6 bg-gray-50 rounded-lg w-full"
            >
              <div class="flex flex-row w-full justify-start border-b-1 border-gray-100 py-2.5">
                <div class="w-full text-gray-600">
                  <SmartsheetHeaderVirtualCell v-if="isVirtualCol(col)" :column="col" :hide-menu="true" />
                  <SmartsheetHeaderCell v-else :column="col" :hide-menu="true" />
                </div>
              </div>

              <div class="flex flex-row w-full pb-3 pt-2 pl-2 items-center justify-start">
                <div v-if="isRowEmpty(record, col)" class="h-3 bg-gray-200 px-5 rounded-lg"></div>
                <template v-else>
                  <SmartsheetVirtualCell v-if="isVirtualCol(col)" v-model="record.row[col.title]" :column="col" :row="record" />
                  <SmartsheetCell v-else v-model="record.row[col.title]" :column="col" :edit-enabled="false" :read-only="true" />
                </template>
              </div>
            </div>
          </a-card>
        </Row>
      </div>
    </div>
    <div class="flex-1" />
    <SmartsheetPagination />
    <SmartsheetExpandedForm
      v-if="expandedFormRow && expandedFormDlg"
      v-model="expandedFormDlg"
      :row="expandedFormRow"
      :state="expandedFormRowState"
      :meta="meta"
    />
  </div>
</template>

<style scoped>
.nc-gallery-container {
  grid-auto-rows: 1fr;
  grid-template-columns: repeat(auto-fit, minmax(230px, 1fr));
}
:depp(.slick-dots li button) {
  background-color: black;
}
.ant-carousel.gallery-carousel :deep(.slick-dots) {
  position: relative;
  height: auto;
  bottom: 0px;
}
.ant-carousel.gallery-carousel :deep(.slick-dots li div > div) {
  background: #000;
  border: 0;
  border-radius: 1px;
  color: transparent;
  cursor: pointer;
  display: block;
  font-size: 0;
  height: 3px;
  opacity: 0.3;
  outline: none;
  padding: 0;
  transition: all 0.5s;
  width: 100%;
}
.ant-carousel.gallery-carousel :deep(.slick-dots li.slick-active div > div) {
  opacity: 1;
}
.ant-carousel.gallery-carousel :deep(.slick-prev) {
  left: 0;
  height: 100%;
  top: 12px;
  width: 50%;
}
.ant-carousel.gallery-carousel :deep(.slick-next) {
  right: 0;
  height: 100%;
  top: 12px;
  width: 50%;
}
</style>
