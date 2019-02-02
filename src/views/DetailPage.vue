<template>
  <div v-if="item !== null">
    <div
      class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3"
    >
      <h1 class="h2">{{ item.title }}</h1>
      <div class="btn-toolbar mb-2 mb-md-0">
        <div class="btn-group mr-2">
          <button
            type="button"
            @click="onScreenSelected('details')"
            class="btn btn-sm btn-outline-secondary"
          >Details</button>
          
          <button
            type="button"
            @click="onScreenSelected('tasks')"
            class="btn btn-sm btn-outline-secondary"
          >Tasks</button>
          
          <button
            type="button"
            @click="onScreenSelected('chitchat')"
            class="btn btn-sm btn-outline-secondary"
          >Chitchat</button>
        </div>
      </div>
    </div>

    <template v-if="selectedDetailsScreen === 'details'">
      <PtItemDetails/>
    </template>
    <template v-else-if="selectedDetailsScreen === 'tasks'">
      <PtItemTasks/>
    </template>
    <template v-else-if="selectedDetailsScreen === 'chitchat'">
      <PtItemChitchat/>
    </template>
  </div>
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
import PtItemDetails from "@/components/detail/ItemDetails.vue";
import PtItemTasks from "@/components/detail/ItemTasks.vue";
import PtItemChitchat from "@/components/detail/ItemChitchat.vue";
import { getIndicatorClass } from "@/shared/helpers/priority-styling";

import { DetailScreenType } from "@/shared/models/ui/types/detail-screens";

@Component({
  components: {
    PtItemDetails,
    PtItemTasks,
    PtItemChitchat
  }
})
export default class DetailPage extends Vue {
  private store: Store = new Store();
  private backlogRepo: BacklogRepository = new BacklogRepository();
  private backlogService: BacklogService = new BacklogService(
    this.backlogRepo,
    this.store
  );

  public selectedDetailsScreen: DetailScreenType = "details";
  private itemId: number = 0;
  private item: PtItem = null;

  constructor() {
    super();
  }

  @Watch("$route")
  public onRouteChange(val: Route, oldVal: Route) {
    // this.refresh();
  }

  public created() {
    this.selectedDetailsScreen = this.$route.params.screen as DetailScreenType;
    this.itemId = Number(this.$route.params.id);
    this.refresh();
  }

  private refresh() {
    this.backlogService.getPtItem(this.itemId).then(item => {
      this.item = item;

      //this.tasks$.next(item.tasks);
      //this.comments$.next(item.comments);
    });
  }

  public onScreenSelected(screen: DetailScreenType) {
    this.selectedDetailsScreen = screen;
    this.$router.push(`/detail/${this.itemId}/${screen}`);
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
