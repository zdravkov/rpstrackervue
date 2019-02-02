<template>
  <form>
    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Title</label>
      <div class="col-sm-10">
        <input class="form-control" v-model="itemForm.title" @blur="onBlurTextField" name="title">
      </div>
    </div>

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Item Type</label>
      <div class="col-sm-10">
        <select
          class="form-control"
          v-model="itemForm.typeStr"
          @change="onNonTextFieldChange"
          name="itemType"
        >
          <option v-for="t in itemTypesProvider" :value="t" :key="t">{{ t }}</option>
        </select>
      </div>
    </div>
  </form>
</template>

<script lang="ts">
import { Component, Vue, Prop, Emit } from "vue-property-decorator";
import { PtItem, PtUser } from "@/core/models/domain";
import {
  ItemType,
  PT_ITEM_STATUSES,
  PT_ITEM_PRIORITIES
} from "@/core/constants";
import {
  PtItemDetailsEditFormModel,
  ptItemToFormModel
} from "@/shared/models/forms/pt-item-details-edit-form";

@Component
export default class PtItemDetails extends Vue {
  @Prop() item: PtItem;
  @Emit("itemSaved")
  public itemSaved(item: PtItem): void {}

  public itemTypesProvider = ItemType.List.map(t => t.PtItemType);
  public statusesProvider = PT_ITEM_STATUSES;
  public prioritiesProvider = PT_ITEM_PRIORITIES;
  private itemForm: PtItemDetailsEditFormModel | undefined;
  private selectedAssignee: PtUser | undefined;

  public created() {
    this.itemForm = ptItemToFormModel(this.item);
    this.selectedAssignee = this.item.assignee;
  }

  public onNonTextFieldChange() {
    this.notifyUpdateItem();
  }

  public onBlurTextField() {
    this.notifyUpdateItem();
  }

  private notifyUpdateItem() {
    if (!this.itemForm) {
      return;
    }
    const updatedItem = this.getUpdatedItem(
      this.item,
      this.itemForm,
      this.selectedAssignee!
    );
    this.itemSaved(updatedItem);
  }

  private getUpdatedItem(
    item: PtItem,
    itemForm: PtItemDetailsEditFormModel,
    assignee: PtUser
  ): PtItem {
    const updatedItem = Object.assign({}, item, {
      title: itemForm.title,
      description: itemForm.description,
      type: itemForm.typeStr,
      status: itemForm.statusStr,
      priority: itemForm.priorityStr,
      estimate: itemForm.estimate,
      assignee: assignee
    });
    return updatedItem;
  }
}
</script>
