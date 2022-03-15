<template lang="pug">
div(v-if="view && classes")
  draggable.row(v-model="elements" handle=".handle")
    // TODO: Handle large/variable sized visualizations better
    div.col-md-6.col-lg-4.p-3(v-for="el, index in elements", :key="index", :class="{'col-md-12': isVisLarge(el), 'col-lg-8': isVisLarge(el)}")
      aw-selectable-vis(:id="index" :type="el.type" @onTypeChange="onTypeChange" @onRemove="onRemove" :editable="editing")

    div.col-md-6.col-lg-4.p-3(v-if="editing")
      b-button(@click="addVisualization" variant="outline-dark" block size="lg")
        icon(name="plus")
        span {{ $t('addViz') }}
</template>

<script>
import 'vue-awesome/icons/save';
import 'vue-awesome/icons/times';
import 'vue-awesome/icons/trash';
import 'vue-awesome/icons/undo';
import draggable from 'vuedraggable';

export default {
  name: 'ActivityView',
  components: {
    draggable: draggable,
  },
  props: {
    view_id: { type: String, default: 'default' },
    periodLength: {
      type: String,
      default: 'day',
    },
  },
  data() {
    return { editing: false };
  },
  computed: {
    classes: function () {
      return this.$store.state.categories?.classes?.length ? this.$store.state.categories.classes : null;
    },
    view: function () {
      if (this.view_id == 'default') {
        return this.$store.state.views.views[0];
      } else {
        return this.$store.state.views.views.find(v => v.id == this.view_id);
      }
    },
    elements: {
      get() {
        return this.view.elements;
      },
      set(elements) {
        this.$store.commit('views/setElements', { view_id: this.view.id, elements });
      },
    },
  },
  methods: {
    addVisualization: function () {
      this.$store.commit('views/addVisualization', { view_id: this.view.id, type: 'top_apps' });
    },
    async onTypeChange(id, type) {
      await this.$store.commit('views/editView', { view_id: this.view.id, el_id: id, type });
    },
    async onRemove(id) {
      await this.$store.commit('views/removeVisualization', { view_id: this.view.id, el_id: id });
    },
    isVisLarge(el) {
      return el.type == 'sunburst_clock';
    },
  },
};
</script>
