<template>
  <div
    class="state-card"
  >
    <v-card
      class="mx-auto shadow-sm"
      outlined
      @click="openDialog = true"
      @mouseenter="isHovering = true"
      @mouseleave="isHovering = false"
    >
      <v-btn
        class="card-remove"
        align="right"
        icon
        color="dark-grey"
        @click="deleteCard(card)"
      >
        <v-icon
          v-show="isHovering"
          x-small
        >
          fa fa-times
        </v-icon>
      </v-btn>
      <v-card-title class="text-h6 card-title">
        {{ card.assignment ? card.assignment.title : card.title }}
      </v-card-title>
      <v-card-subtitle class="card-subtitle">
        {{ card.assignment ? card.assignment.desc : card.desc }}
      </v-card-subtitle>
      <v-list style="padding: 10px;">
          <v-chip
            small
            v-if="card.assignment && card.assignment.classroom"
            text-color="blue"
            color="white"
          >
            <v-icon x-small> fas fa-fire-alt </v-icon>&nbsp;
            {{ card.assignment.classroom.name }}
          </v-chip>
        <v-list-item
          v-for="tag in card.tags"
          :key="tag.id"
          style="padding: 0 5px; display: inline"
        >
          <v-chip
            :color="tag.color_hex || 'pink'"
            small
            label
            :text-color="getContrastYIQ(tag.color_hex)"
          >
            {{ tag.label }}
          </v-chip>
        </v-list-item>
      </v-list>
    </v-card>
    <card-dialog
      :open-dialog="openDialog"
      :readonly="readonly || !!card.assignment"
      :hide-tags="hideTags"
      :card="card.assignment ? {...card, ...card.assignment, id: card.id} : card"
      @add-tag="$emit('add-tag', $event)"
      @update="$emit('updateCard', $event)"
      @remove-tag="$emit('removeTag', $event)"
      @close="closeCardDialog"
    />
  </div>
</template>
<script>
import CardDialog from './CardDialog';

export default {
  components: {
    'card-dialog': CardDialog
  },
  props: {
    card: {
      type: Object,
      default: () => ({})
    },
    readonly: {
      type: Boolean,
      default: false
    },
    hideTags: {
      type: Boolean,
      default: false
    },
  },
  data() {
    return {
      isHovering: false,
      openDialog: false
    }
  },
  methods: {
    closeCardDialog() {
      console.log('Close')
      this.openDialog = false;
    },
    getContrastYIQ(hexcolor){
      hexcolor = hexcolor.replace('#', '');
      var r = parseInt(hexcolor.substr(0,2),16);
      var g = parseInt(hexcolor.substr(2,2),16);
      var b = parseInt(hexcolor.substr(4,2),16);

      var yiq = ((r*299)+(g*587)+(b*114))/1000;
      return (yiq >= 128) ? 'black' : 'white';
    },
    deleteCard(card) {
      if (!confirm(`Are you sure you want to delete card ${card.title}`)) return;

      this.$http.delete(`cards/${card.id}`)
        .then(() => {
          this.$emit('deleteCard', card);
        })
        .catch(err => {
          console.error(err);
        });
    },
  },
};
</script>
<style scoped lang="scss">
 .card {
    &-remove {
      float: right;
    }

    &-title {
      text-overflow: ellipsis;
      contain: content;
    }

    &-subtitle {
      text-overflow: ellipsis;
      max-height: 20px;
      contain: content;
      font-size: 9px;
    }
 }
</style>
