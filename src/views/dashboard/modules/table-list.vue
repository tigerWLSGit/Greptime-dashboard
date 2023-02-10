<template lang="pug">
a-scrollbar.tree-scrollbar
  a-tree.table-tree(v-if="!ifTableLoading" :key="tableKey" :data="tableList" :load-more="loadMore" size="small")
    template(#title)
    template(#extra="nodeData")
      img(:src="getIconUrl(nodeData.iconType)" alt="" height="14")
      span.tree-title
        | {{ nodeData.title }}
      span.data-type
        | {{ nodeData.dataType }}
      a-tooltip(:content="$t('dataExplorer.insertName')" mini)
        svg.icon.copy-icon.pointer(name="copy" @click="insertName(nodeData.title)")
          use(href="#copy")
</template>

<script lang="ts" setup>
  import { storeToRefs } from 'pinia'
  import { useDataBaseStore, useAppStore } from '@/store'
  import useDataExplorer from '@/hooks/data-explorer'
  import usePythonCode from '@/hooks/python-code'

  const { insertNameToCode } = useDataExplorer()
  const { codeType, guideModal } = storeToRefs(useAppStore())
  const { insertNameToPyCode } = usePythonCode()

  const { fetchOneTable, fetchDataBaseTables } = useDataBaseStore()
  const { tableList, ifTableLoading, tableKey } = storeToRefs(useDataBaseStore())

  const loadMore = (nodeData: any) => {
    return new Promise<void>((resolve, reject) => {
      fetchOneTable(nodeData.title)
        .then((result: any) => {
          const { output } = result
          const {
            records: { rows },
          } = output[0]
          const rowArray: any = []
          rows.forEach((row: any) => {
            rowArray.push({
              title: row[0],
              key: row[0],
              isLeaf: true,
              dataType: row[1],
              iconType: row[4],
            })
          })
          // todo: change computed data might not be the best option.
          nodeData.children = rowArray
          resolve()
          // TODO: change key to update component might not be the best option.
          tableKey.value += 1
        })
        .catch(() => {
          reject()
        })
    })
  }

  const insertName = (name: string) => {
    return codeType.value === 'sql' ? insertNameToCode(name) : insertNameToPyCode(name)
  }

  const ICON_MAP: { [key: string]: string } = {
    'VALUE': 'value-icon.png',
    'PRIMARY KEY': 'key-icon.png',
    'TIME INDEX': 'time-icon.png',
  }
  // TODO: Better use iconPark.
  const getIconUrl = (type: any) => {
    return new URL(`../../../assets/images/${ICON_MAP[type]}`, import.meta.url).href
  }

  if (!guideModal.value) {
    fetchDataBaseTables()
  }
</script>