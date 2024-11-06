<script setup lang="ts">
import { ref, computed } from 'vue';
import DiffMatchPatch from 'diff-match-patch';

const dmp = new DiffMatchPatch();

const baseText = ref('foo, bar, baz, qux, quux, foobar');

const fields = ref([
  { value: 'foo, bar, baz, quux, foobar' },
  { value: 'foo, quux, foobar, xyzzy' },
]);

const changeObjects = computed(() => {
  return fields.value.map((field) => {
    const diffs = dmp.diff_main(baseText.value, field.value);
    dmp.diff_cleanupSemantic(diffs);
    return diffs;
  });
});

</script>

<template>
  <main>
    <h1>multi-diff</h1>

    <div class="forms">
      <div class="form-and-text">
        <textarea v-model="baseText" class="text-input"></textarea>
        <div class="result">{{ baseText }}</div>
      </div>
      <hr>

      <transition-group name="list">
        <template v-for="(field, i) in fields" :key="i">
          <div class="form-and-text">
            <textarea v-model="field.value" class="text-input"></textarea>
            <div class="result">
              <span v-for="(diff, j) in changeObjects[i]" :key="j" :class="diff[0] === 1 ? 'added' : diff[0] === -1 ? 'removed' : ''">
                {{ diff[1] }}
              </span>
            </div>
          </div>
          <hr>
        </template>
      </transition-group>

      <button @click="fields.push({ value: '' })">Add inputs</button>
    </div>
  </main>
</template>

<style scoped>
.form-and-text {
  display: flex;
  gap: 1rem;
  font-family: Hack, monospace;
  text-align: left;
}

.text-input {
  flex: 1;
  min-height: 100px;
}

.result {
  flex: 1;
  min-width: 0;
  overflow-wrap: break-word;
  white-space: pre-wrap;
}

.added {
  font-weight: 600;
  background: rgb(71, 142, 88);
  border: 1px solid rgb(171, 242, 188);
}

.removed {
  font-weight: 600;
  background: rgb(142, 71, 71);
  border: 1px solid rgb(242, 171, 171);
}

@media (prefers-color-scheme: light) {
  .added {
    background: rgb(171, 242, 188);
    border: 1px solid rgb(71, 142, 88);
  }

  .removed {
    background: rgb(242, 171, 171);
    border: 1px solid rgb(142, 71, 71);
  }
}

.list-enter-active, .list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from, .list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>
