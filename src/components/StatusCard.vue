<template>
  <v-flex>
    <v-card v-bind:style="{ 'background-color':'#ddd' }">
      <div class="header">
        <h4 class="display-1">{{title}}</h4>
      </div>
      <draggable v-model="kanbans" group="list" @start="drag=true" @end="drag=false">
        <KanbanCard v-for="kanban in kanbans" :key="kanban.id" :kanban="kanban" :title="title"></KanbanCard>
      </draggable>
    </v-card>
  </v-flex>
</template>

<script>
  import db from "../db"
  import draggable from 'vuedraggable';
  import KanbanCard from './KanbanCard';
  export default {
    components: {
      KanbanCard,
      draggable
    },
    props: ['title', 'status'],
    data() {
      return {
        kanbans: []
      }
    },
    created() {
      db.collection("Task").where("status", "==", this.status)
        .onSnapshot((querySnapshot) => {
          let arr = []
          this.kanbans = []
          querySnapshot.forEach((doc) => {
            arr.push({
              id: doc.id,
              ...doc.data()
            });
          });
          for (const kanban of arr) {
            this.kanbans.push(kanban);
          }
        });
    },
    watch: {
      kanbans: function () {
        // console.log(this.kanbans)
        let newKanban = this.kanbans.filter((kanban) => {
          if (kanban.status != this.status) {
            return kanban;
          }
        });

        newKanban.forEach((kanban) => {
          db.collection("Task").doc(kanban.id).update({
            status: this.status
          }).then(() => {
            console.log("Document successfully updated!");
          }).catch(function (error) {
            this.dialog = false
            console.error("Error removing document: ", error);
          });
        })
      }
    }
  }
</script>

<style>
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    border-bottom: 1px solid #777;
    height: 50px;
    background-color: #41B883;
    color: white;
  }
</style>
