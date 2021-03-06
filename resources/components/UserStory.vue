<template>
  <a-alert :type="story.isSystem ? 'info' : 'warning'" class="user-story">
    <template slot="message">
      US-{{ String(story.id).padStart(2, '0') }}

      <a-button-group style="float: right">
        <a-button
          :type="showDetails ? undefined : 'primary'"
          size="small"
          icon="eye"
          @click="showDetails = !showDetails"
        />
        <a-button
          v-if="!readOnly"
          v-show="model.title.length > 0"
          size="small"
          icon="scissor"
          @click="$emit('breakdown', story)"
        />
        <a-button
          v-if="!readOnly"
          size="small"
          :icon="story.isSystem ? 'laptop' : 'user'"
          @click=";(model.isSystem = !model.isSystem), update()"
        />
        <a-popconfirm
          v-if="!readOnly"
          title="Are you sure delete this story?"
          ok-text="Yes"
          cancel-text="No"
          placement="bottom"
          @confirm="$emit('delete')"
        >
          <a-button type="danger" size="small" icon="delete" />
        </a-popconfirm>
      </a-button-group>

      <div style="float: right">
        <a-select
          v-model="model.priority"
          style="min-width: 64px; text-aling: center; margin-right: 3px"
          :class="{ 'invalid-order': wrongPosition }"
          size="small"
          :default-value="model.priority"
          :disabled="readOnly"
          @change="update"
        >
          <a-select-option v-for="(item, i) in scale" :key="i" :value="item">
            {{ item }}
          </a-select-option>
        </a-select>
      </div>
    </template>

    <template slot="description">
      <a-textarea
        v-model="model.title"
        :disabled="readOnly"
        placeholder="As a < type of user >, I want < some goal > so that < some reason >."
        :autosize="{ minSize: 2 }"
        class="user-story-text-area"
        @input="update"
      />

      <div v-show="showDetails">
        <p>
          <strong>Business Rules</strong>
          <StringArrayModel
            v-model="model.businessRules"
            add-text="Add Business Rule"
            placeholder="Only do < action > when < condition >"
            class-name="user-story-text-area"
            :disabled="readOnly"
            @input="update"
          />
        </p>

        <p>
          <strong>Acceptance Scenarios</strong>
          <StringArrayModel
            v-model="model.acceptanceScenarios"
            add-text="Add Acceptance Scenario"
            :placeholder="
              `GIVEN < state >
WHEN < condition >
THEN < expectation >`
            "
            class-name="user-story-text-area"
            :disabled="readOnly"
            @input="update"
          />
        </p>
      </div>
    </template>
  </a-alert>
</template>

<script>
import StringArrayModel from '@/components/StringArrayModel'
import pDebounce from 'p-debounce'

export default {
  name: 'UserStory',

  components: {
    StringArrayModel
  },

  props: {
    story: {
      type: Object,
      required: true
    },

    wrongPosition: {
      type: Boolean,
      default: false
    },

    readOnly: {
      type: Boolean,
      default: false
    },

    scale: {
      type: Array,
      default: () => []
    }
  },

  data: () => ({
    showDetails: false,
    model: {}
  }),

  created() {
    const model = JSON.parse(JSON.stringify(this.story))

    if (!this.scale.includes(model.priority)) {
      model.priority = ''
    }

    this.model = model
  },

  methods: {
    update: pDebounce(function update() {
      this.$emit('update', this.model)
    }, 500)
  }
}
</script>

<style>
.user-story {
  margin-bottom: 10px;
  transition: all 0.3s;
}

.user-story .user-story-text-area {
  resize: none;
  border: none;
  background-color: rgba(0, 0, 0, 0);
}

.invalid-order .ant-select-selection {
  border-color: red;
}
</style>
