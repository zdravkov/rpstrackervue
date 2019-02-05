<template>
  <form>
    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Title</label>
      <div class="col-sm-10">
        <input class="form-control" v-model="itemForm.title" @blur="onBlurTextField" name="title">
      </div>
    </div>

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Description</label>
      <div class="col-sm-10">
        <textarea
          class="form-control"
          v-model="itemForm.description"
          @blur="onBlurTextField"
          name="description"
        ></textarea>
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

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Status</label>
      <div class="col-sm-10">
        <select
          class="form-control"
          v-model="itemForm.statusStr"
          @change="onNonTextFieldChange"
          name="itemStatus"
        >
          <option v-for="s in statusesProvider" :value="s" :key="s">{{ s }}</option>
        </select>
      </div>
    </div>

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Estimate</label>
      <div class="col-sm-10">
        <input
          class="form-control"
          type="range"
          step="1"
          min="0"
          max="20"
          v-model="itemForm.estimate"
          @blur="onBlurTextField"
          style="width: 300px"
          name="estimate"
        >
      </div>
    </div>

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Priority</label>
      <div class="col-sm-10">
        <select
          class="form-control"
          v-model="itemForm.priorityStr"
          @change="onNonTextFieldChange"
          name="itemPrority"
        >
          <option v-for="p in prioritiesProvider" :value="p" :key="p">{{ p }}</option>
        </select>
      </div>
    </div>

    <div class="form-group row">
      <label class="col-sm-2 col-form-label">Assignee</label>

      <div class="col-sm-10">
        <img :src="this.selectedAssignee.avatar" class="li-avatar rounded">
        <span>{{itemForm.assigneeName}}</span>
        
        <button
          type="button"
          class="btn btn-sm btn-outline-secondary"
          @click="assigneePickerOpen"
        >Pick assignee</button>
      </div>
    </div>
  </form>
</template>

<script lang="ts">
import { Component, Vue, Prop, Emit } from 'vue-property-decorator';
import { Observable } from 'rxjs';

import { PtItem, PtUser } from '@/core/models/domain';
import {
    ItemType,
    PT_ITEM_STATUSES,
    PT_ITEM_PRIORITIES,
} from '@/core/constants';
import {
    PtItemDetailsEditFormModel,
    ptItemToFormModel,
} from '@/shared/models/forms/pt-item-details-edit-form';
import { EMPTY_STRING } from '@/core/helpers';
import { PriorityEnum, StatusEnum } from '@/core/models/domain/enums';

@Component
export default class PtItemDetails extends Vue {
    @Prop() public item!: PtItem;

    public itemTypesProvider = ItemType.List.map(t => t.PtItemType);
    public statusesProvider = PT_ITEM_STATUSES;
    public prioritiesProvider = PT_ITEM_PRIORITIES;

    public showAddModal: boolean = false;
    public users: PtUser[] = [];
    private itemForm: PtItemDetailsEditFormModel | undefined;
    private selectedAssignee: PtUser | undefined;
    // @Prop() public usersObs!: Observable<PtUser[]>;
    // @Emit("usersRequested")
    // public usersRequested() {}
    @Emit('itemSaved')
    public itemSaved(item: PtItem): void {}

    public created() {
        if (this.item) {
            this.itemForm = ptItemToFormModel(this.item);
            this.selectedAssignee = this.item.assignee;
        }
    }

    public assigneePickerOpen() {
        /*
    this.usersObs.subscribe((users: PtUser[]) => {
      debugger;
      if (users.length > 0) {
        this.users = users;
        this.showAddModal = true;
      }
    });
 */
        // this.usersRequested();
    }

    public onNonTextFieldChange() {
        this.notifyUpdateItem();
    }

    public onBlurTextField() {
        this.notifyUpdateItem();
    }

    private toggleModal() {
        this.showAddModal = !this.showAddModal;
        return false;
    }

    private notifyUpdateItem() {
        if (!this.itemForm) {
            return;
        }
        const updatedItem = this.getUpdatedItem(
            this.item!,
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
            assignee,
        });

        return updatedItem;
    }
}
</script>
