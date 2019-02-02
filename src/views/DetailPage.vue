<template>
  <div>details</div>
</template>

<script lang="ts">
import { Component, Vue, Model, Watch } from "vue-property-decorator";
import { Route } from "vue-router";

import { BacklogService } from "@/services/backlog-service";
import { BacklogRepository } from "@/repositories/backlog-repository";
import { EMPTY_STRING } from "@/core/helpers";
import { Store } from "@/core/state/app-store";

import { PresetType } from "@/core/models/domain/types";
import { PtItem } from "@/core/models/domain";
import { ItemType } from "@/core/constants";
import { PtNewItem } from "@/shared/models/dto/pt-new-item";
import PresetFilter from "@/components/PresetFilter.vue";
import { getIndicatorClass } from "@/shared/helpers/priority-styling";

@Component({
  components: {
    PresetFilter
  }
})
export default class DetailPage extends Vue {
  private store: Store = new Store();
  private backlogRepo: BacklogRepository = new BacklogRepository();
  private backlogService: BacklogService = new BacklogService(
    this.backlogRepo,
    this.store
  );

  public currentPreset: PresetType;
  public items: PtItem[];
  public showAddModal: boolean;
  public newItem: PtNewItem;

  constructor() {
    super();

    this.currentPreset = "open";
    this.items = [];
    this.showAddModal = false;
    this.newItem = this.initModalNewItem();
  }

  @Watch("$route")
  public onRouteChange(val: Route, oldVal: Route) {
    this.refresh();
  }

  public created() {
    this.currentPreset = this.$route.params.preset as PresetType;
    this.refresh();
  }

  private refresh() {
    this.backlogService.getItems(this.currentPreset).then(ptItems => {
      this.items = ptItems;
    });
  }

  private onSelectPresetTap(preset: PresetType) {
    this.currentPreset = preset;
    this.$router.push("/backlog/" + preset);
  }

  private initModalNewItem(): PtNewItem {
    return {
      title: EMPTY_STRING,
      description: EMPTY_STRING,
      typeStr: "PBI"
    };
  }

  public getIndicatorImage(item: PtItem) {
    return ItemType.imageResFromType(item.type);
  }

  public getPriorityClass(item: PtItem): string {
    const indicatorClass = getIndicatorClass(item.priority);
    return indicatorClass;
  }
}
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
