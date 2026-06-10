<template>
  <section class="table-panel">
    <div class="panel-head">
      <div>
        <p class="eyebrow">Attraction Library</p>
        <h3>景点路线设计素材</h3>
      </div>
      <span>{{ attractions.length }} 个景点</span>
    </div>
    <div class="attraction-list">
      <article v-for="item in attractions" :key="item.id">
        <div>
          <h4>{{ item.name }}</h4>
          <p>{{ item.highlight }}</p>
        </div>
        <span>{{ item.city }}</span>
        <span>{{ item.category_label }}</span>
        <strong>{{ item.duration_hours }}h ·</strong>
        <div class="ticket-edit">
          <template v-if="editingId === item.id">
            <input
              v-model.number="editPrice"
              type="number"
              min="0"
              step="0.01"
              class="price-input"
              @keyup.enter="savePrice(item)"
              @keyup.escape="cancelEdit"
            />
            <div class="edit-actions">
              <button type="button" class="btn-save" @click="savePrice(item)">保存</button>
              <button type="button" class="btn-cancel" @click="cancelEdit">取消</button>
            </div>
          </template>
          <template v-else>
            <span class="price-display" @dblclick="startEdit(item)">¥{{ item.ticket_price }}</span>
            <button type="button" class="btn-edit" @click="startEdit(item)">编辑</button>
          </template>
        </div>
      </article>
    </div>
    <div v-if="saving" class="state">正在更新门票价格...</div>
    <div v-if="saveError" class="state error">{{ saveError }}</div>
  </section>
</template>

<script setup>
import { ref } from "vue";
import { travelApi } from "../api/travel";

const props = defineProps({
  attractions: { type: Array, required: true },
});

const emit = defineEmits(["attraction-updated"]);

const editingId = ref(null);
const editPrice = ref(0);
const saving = ref(false);
const saveError = ref("");

function startEdit(item) {
  editingId.value = item.id;
  editPrice.value = Number(item.ticket_price);
  saveError.value = "";
}

function cancelEdit() {
  editingId.value = null;
  saveError.value = "";
}

async function savePrice(item) {
  if (editPrice.value < 0) {
    saveError.value = "门票价格不能为负数";
    return;
  }
  saving.value = true;
  saveError.value = "";
  try {
    const oldPrice = Number(item.ticket_price);
    const newPrice = Number(editPrice.value);
    const updated = await travelApi.updateAttraction(item.id, {
      ...item,
      ticket_price: newPrice,
    });
    cancelEdit();
    if (oldPrice !== newPrice) {
      emit("attraction-updated", updated);
    }
  } catch (err) {
    saveError.value = `更新失败：${err.message}`;
  } finally {
    saving.value = false;
  }
}
</script>

<style scoped>
.ticket-edit {
  display: flex;
  align-items: center;
  gap: 8px;
}

.price-display {
  font-weight: 700;
  cursor: pointer;
  padding: 2px 6px;
  border-radius: 4px;
  transition: background-color 0.15s;
}

.price-display:hover {
  background-color: rgba(99, 102, 241, 0.1);
}

.price-input {
  width: 100px;
  padding: 4px 8px;
  border: 2px solid #6366f1;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  outline: none;
}

.edit-actions {
  display: flex;
  gap: 4px;
}

.btn-edit,
.btn-save,
.btn-cancel {
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.15s;
}

.btn-edit {
  background-color: #eef2ff;
  color: #4f46e5;
}

.btn-edit:hover {
  background-color: #e0e7ff;
}

.btn-save {
  background-color: #10b981;
  color: white;
}

.btn-save:hover {
  background-color: #059669;
}

.btn-cancel {
  background-color: #f3f4f6;
  color: #4b5563;
}

.btn-cancel:hover {
  background-color: #e5e7eb;
}
</style>
