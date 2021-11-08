<template>
  <div>
    <div
      class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3"
    >
      <h1 class="h2">Backlog</h1>
      <div class="btn-toolbar mb-2 mb-md-0">
        <PresetFilter :selectedPreset="currentPreset" @onPresetSelected="onSelectPresetTap"/>

        <div class="btn-group mr-2">
          <button type="button" @click="toggleModal" class="btn btn-sm btn-outline-secondary">Add</button>
        </div>
      </div>
    </div>

     <Grid
      :data-items="gridData"
      :cell-render="'cellTemplate'"
      :columns="columns"
      @rowclick="onSelectionChange"
      :pageable="true"
      :skip="skip"
      :take="take"
      :total="total"
      @pagechange="onPageChange"
      :sortable="true"
      :sort="sort"
      @sortchange="onSortChange"
      style="height: 400px"
    >
      <template v-slot:cellTemplate="{props}">
        <td :class="props.className" v-html="getNestedValue(props)">
        </td>
      </template>
    </Grid>

    <transition v-if="showAddModal">
      <div class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="modal-header">
              <h4 class="modal-title" id="modal-basic-title">Add New Item</h4>
              <button type="button" class="close" @click="toggleModal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>

            <div class="modal-body">
              <form>
                <div class="form-group row">
                  <label class="col-sm-2 col-form-label">Title</label>
                  <div class="col-sm-10">
                    <input class="form-control" v-model="newItem.title" name="title">
                  </div>
                </div>

                <div class="form-group row">
                  <label class="col-sm-2 col-form-label">Description</label>
                  <div class="col-sm-10">
                    <textarea class="form-control" v-model="newItem.description" name="description"></textarea>
                  </div>
                </div>

                <div class="form-group row">
                  <label class="col-sm-2 col-form-label">Item Type</label>
                  <div class="col-sm-10">
                    <select class="form-control" v-model="newItem.typeStr" name="itemType">
                      <option v-for="t in itemTypesProvider" :key="t" :value="t">{{t}}</option>
                    </select>
                  </div>
                </div>
              </form>
            </div>

            <div class="modal-footer">
              <button class="btn" @click="toggleModal">Cancel</button>
              <button class="btn btn-primary" @click="onAddSave">OK</button>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch, computed } from "vue";
import { useRouter, useRoute } from "vue-router";
import { BacklogService } from "@/services/backlog-service";
import { BacklogRepository } from "@/repositories/backlog-repository";
import { EMPTY_STRING } from "@/core/helpers";
import { Store } from "@/core/state/app-store";

import { PresetType } from "@/core/models/domain/types";
import { PtItem, PtUser } from "@/core/models/domain";
import { ItemType } from "@/core/constants";
import { PtNewItem } from "@/shared/models/dto/pt-new-item";
import PresetFilter from "@/components/PresetFilter.vue";
import { getIndicatorClass } from "@/shared/helpers/priority-styling";
import { GridColumnProps, Grid } from "@progress/kendo-vue-grid";
import { orderBy, SortDescriptor } from "@progress/kendo-data-query";

export default defineComponent({
  name: "BacklogPage",
  components: {
    PresetFilter,
    Grid,
  },
  setup() {
    const router = useRouter();
    const route = useRoute();
    const currentPreset = ref<PresetType>("open");
    const items = ref<PtItem[]>([]);
    const itemTypesProvider = ref(ItemType.List.map(t => t.PtItemType));
    const showAddModal = ref(false);
    const listItemTap = (item: PtItem) => {
      // navigate to detail page
      router.push(`/detail/${item.id}/details`);
    };

    const columns = ref<GridColumnProps[]>([
      { field: 'type', title: ' ', width: 40 },
      {
        field: 'assignee',
        title: 'Assignee',
        width: 260,
      },
      { field: 'title', title: 'Title' },
      { field: 'priority', title: 'Priority', width: 100 },
      { field: 'estimate', title: 'Estimate', width: 100 },
      { field: 'dateCreated', title: 'Created', width: 160 },
    ]);
    const skip = ref(0);
    const take = ref(10);
    const sort = ref<SortDescriptor[]>([{ field: 'title', dir: 'asc' }]);

    const total = computed(() => {
      return items.value ? items.value.length : 0;
    });

    const gridData = computed(() => {
        return items.value
            ? orderBy(
                  items.value.slice(skip.value, take.value + skip.value),
                  sort.value
              )
            : [];
    });

    const getIndicatorImage = (item: PtItem) => {
      return ItemType.imageResFromType(item.type);
    };

    const getPriorityClass = (item: PtItem): string => {
      const indicatorClass = getIndicatorClass(item.priority);
      return indicatorClass;
    };

    const onAddSave = () => {
      if (store.value.currentUser) {
        backlogService
          .addNewPtItem(newItem.value, store.value.currentUser)
          .then((nextItem: PtItem) => {
            showAddModal.value = false;
            newItem.value = initModalNewItem();
            items.value = [nextItem, ...items.value];
          });
      }
    };

    const refresh = () => {
      backlogService.getItems(currentPreset.value).then(ptItems => {
          items.value = ptItems;
      });
    };

    const onSelectionChange = (event: any) => {
        const selItem = event.dataItem as PtItem;
        router.push(`/detail/${selItem.id}/details`);
    };

    const onSelectPresetTap = (preset: PresetType) => {
      currentPreset.value = preset;
      router.push('/backlog/' + preset);
    };

    const toggleModal = () => {
      showAddModal.value = !showAddModal.value;
    };

    const onPageChange = (event: any) => {
        skip.value = event.page.skip;
        take.value = event.page.take;
    };

    const onSortChange = (event: any) => {
        sort.value = event.sort;
    }

    const initModalNewItem = (): PtNewItem => {
      return {
        title: EMPTY_STRING,
        description: EMPTY_STRING,
        typeStr: 'PBI',
      };
    };

    const getItemTypeCellMarkup = (item: PtItem) => {
        return `<img src="${getIndicatorImage(
            item
        )}" class="backlog-icon" />`;
    };

    const getAssigneeCellMarkup = (user: PtUser) => {
        return `
        <div>
          <img src="${user.avatar}" class="li-avatar rounded mx-auto" />
          <span style="margin-left: 10px;">${user.fullName}</span>
        </div>
      `;
    };

    const getPriorityCellMarkup = (item: PtItem) => {
        return `<span class="${'badge ' + getPriorityClass(item)}">${
            item.priority
        }</span>`;
    };

    const getCreatedDateCellMarkup = (item: PtItem) => {
        return `<span class="li-date">${item.dateCreated.toDateString()}</span>`;
    };

    const getNestedValue = (props: any) => {
        const fieldName = props.field;
        const dataItem = props.dataItem as PtItem;
        switch (fieldName) {
            case 'type':
                return getItemTypeCellMarkup(dataItem);
            case 'assignee':
                return getAssigneeCellMarkup(dataItem.assignee);
            case 'priority':
                return getPriorityCellMarkup(dataItem);
            case 'dateCreated':
                return getCreatedDateCellMarkup(dataItem);
            default:
                return (dataItem as any)[fieldName];
        }
    };

    const newItem = ref<PtNewItem>(initModalNewItem());
    let store: Store = new Store();
    let backlogRepo: BacklogRepository = new BacklogRepository();
    let backlogService: BacklogService = new BacklogService(backlogRepo, store);
    currentPreset.value = route.params.preset as PresetType;
    refresh();

    watch(route, () => {
      refresh();
    });

    return {
      onAddSave,
      toggleModal,
      getPriorityClass,
      getIndicatorImage,
      listItemTap,
      currentPreset,
      itemTypesProvider,
      newItem,
      onSelectPresetTap,
      onSelectionChange,
      refresh,
      items,
      showAddModal,
      columns,
      skip,
      take, 
      sort,
      onPageChange,
      onSortChange,
      gridData,
      total,
      getNestedValue
    };
  },
});
</script>

<style scoped>
.backlog-icon {
  height: 20px;
}

.li-indicator {
  height: 58px;
  width: 10px;
  text-align: left;
}

.li-indicator div {
  width: 5px;
  height: 58px;
}

.li-info-wrapper {
  margin-left: 5px;
}

.li-title {
  font-size: 14px;
  color: #4b5833;
}

.pt-table-row {
  cursor: pointer;
}
</style>
