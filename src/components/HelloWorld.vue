<template>
  <v-container>
    <v-card v-if="teste">
      <v-card-actions>
        <v-btn color="blue darken-1 white--text" @click="printWindow()" right>IMPRIMIR</v-btn>
        <v-spacer></v-spacer>
        <v-btn class="m-2" @click="teste = !teste">COM FILTRO</v-btn>
      </v-card-actions>
      <v-data-table :headers="headers" :items="desserts" sort-by="calories" item-key="name" class="elevation-10"
        show-select>
        <template v-for="(col, i) in filters" v-slot:[`header.${i}`]="{ header }">
          <v-row :key="i">
            <v-col style="display: inline-block; padding: 16px 0">{{ header.text }}</v-col>
            <div style="float: right; margin-top: 8px">
              <v-menu :close-on-content-click="false" :nudge-width="200" offset-y transition="slide-y-transition" left
                fixed style="position: absolute; right: 0">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn color="indigo" icon v-bind="attrs" v-on="on">
                    <v-icon small :color="
                      activeFilters[header.value] &&
                      activeFilters[header.value].length < filters[header.value].length
                        ? 'red'
                        : 'default'
                    ">
                      mdi-filter-variant
                    </v-icon>
                  </v-btn>
                </template>
                <v-list flat dense class="pa-0">
                  <v-list-item-group multiple v-model="activeFilters[header.value]" class="py-2">
                    <template>
                      <v-list-item v-for="item in filters[header.value]" :key="`${item}`" :ripple="false" :value="item">
                        <template v-slot:default="{ active, toggle }">
                          <v-list-item-action>
                            <v-checkbox :input-value="active" :true-value="item" @click="toggle" color="primary"
                              :ripple="false" dense></v-checkbox>
                          </v-list-item-action>
                          <v-list-item-content>
                            <v-list-item-title v-text="item"></v-list-item-title>
                          </v-list-item-content>
                        </template>
                      </v-list-item>
                    </template>
                  </v-list-item-group>
                  <v-divider></v-divider>
                  <v-row no-gutters>
                    <v-col cols="6">
                      <v-btn text block @click="toggleAll(header.value)" color="success">Todos</v-btn>
                    </v-col>
                    <v-col cols="6">
                      <v-btn text block @click="clearAll(header.value)" color="warning">Limpar</v-btn>
                    </v-col>
                  </v-row>
                </v-list>
              </v-menu>
            </div>
          </v-row>
        </template>

        <template v-slot:header="{ props: { headers } }">
          <thead>
            <tr>
              <th :colspan="headers.length">This is a header</th>
            </tr>
          </thead>
        </template>

        <template v-slot:top>
          <v-toolbar flat color="white">
            <v-toolbar-title>My CRUD</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>

            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn v-bind="attrs" v-on="on" class="m-2" color="primary" dark>New Item</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.carbs" label="Carbs (g)"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.protein" label="Protein (g)"></v-text-field>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                  <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>

        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize">Reset</v-btn>
        </template>
      </v-data-table>
      
    </v-card>


    <v-card v-else>
      <v-card-actions>
        <v-btn color="blue darken-1 white--text" @click="printWindow()" right>IMPRIMIR</v-btn>
        <v-spacer></v-spacer>
        <v-btn class="m-2" @click="teste = !teste">SEM FILTRO</v-btn>
      </v-card-actions>

      <v-data-table :headers="headersSemFiltro" :items="dessertsSemFiltro" :search="search" class="elevation-10">

        <template v-slot:header="{ props: { headers } }">
          <thead>
            <tr>
              <th :colspan="headers.length">This is a header</th>
            </tr>
          </thead>
        </template>

        <template v-slot:top>
          <v-toolbar flat color="white">
            <v-toolbar-title>My CRUD</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>

            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark class="m-2" v-bind="attrs" v-on="on">New Item</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.carbs" label="Carbs (g)"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field v-model="editedItem.protein" label="Protein (g)"></v-text-field>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                  <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
          <v-row>
            <v-col>
              <v-text-field shaped v-model="search" placeholder="Pesquisar" />
            </v-col>
          </v-row>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>

        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize">Reset</v-btn>
        </template>

      </v-data-table>
      
    </v-card>


  </v-container>
</template>

<script>
export default {
  name: "HelloWorld",

  data: () => ({
    teste: false,
    search: "",
    dialog: false,
    filters: { name: [], calories: [], status: [] },
    activeFilters: {},
    desserts: [],
    dessertsSemFiltro: [],
    editedIndex: -1,
    editedItem: {
      name: "",
      calories: 0,
      fat: 0,
      carbs: 0,
      protein: 0,
    },
    defaultItem: {
      name: "",
      calories: 0,
      fat: 0,
      carbs: 0,
      protein: 0,
    },
    headersSemFiltro: [
      {
        text: "Dessert (100g serving)",
        align: "start",
        sortable: true,
        value: "name",
      },
      {
        text: "Calories",
        value: "calories",
      },
      {
        text: "Status",
        value: "status",
      },
      { text: "Fat (g)", value: "fat" },
      { text: "Carbs (g)", value: "carbs" },
      { text: "Protein (g)", value: "protein" },
      { text: "Actions", value: "actions", sortable: false },
    ]
  }),

  computed: {
    headers() {
      return [
        {
          text: "Dessert (100g serving)",
          align: "start",
          sortable: false,
          value: "name",
          filter: (value) => {
            return this.activeFilters.name
              ? this.activeFilters.name.includes(value)
              : true;
          },
        },
        {
          text: "Calories",
          value: "calories",
          sortable: false,
          filter: (value) => {
            return this.activeFilters.calories
              ? this.activeFilters.calories.includes(value)
              : true;
          },
        },
        {
          text: "Status",
          value: "status",
          sortable: false,
          filter: (value) => {
            return this.activeFilters.status
              ? this.activeFilters.status.includes(value)
              : true;
          },
        },
        { text: "Fat (g)", value: "fat" },
        { text: "Carbs (g)", value: "carbs" },
        { text: "Protein (g)", value: "protein" },
        { text: "Actions", value: "actions", sortable: false },
      ];
    },
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    desserts() {
      this.initFilters();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      this.desserts = [
        {
          name: "Frozen Yogurt",
          calories: 159,
          fat: 6.0,
          carbs: 24,
          protein: 4.0,
          status: "DIET",
        },
        {
          name: "Ice cream sandwich",
          calories: 237,
          fat: 9.0,
          carbs: 37,
          protein: 4.3,
          status: "NO DIET",
        },
        {
          name: "Eclair",
          calories: 262,
          fat: 16.0,
          carbs: 23,
          protein: 6.0,
          status: "DIET",
        },
        {
          name: "Cupcake",
          calories: 305,
          fat: 3.7,
          carbs: 67,
          protein: 4.3,
          status: "NO DIET",
        },
        {
          name: "Gingerbread",
          calories: 356,
          fat: 16.0,
          carbs: 49,
          protein: 3.9,
          status: "DIET",
        },
        {
          name: "Jelly bean",
          calories: 375,
          fat: 0.0,
          carbs: 94,
          protein: 0.0,
          status: "NO DIET",
        },
        {
          name: "Lollipop",
          calories: 392,
          fat: 0.2,
          carbs: 98,
          protein: 0,
          status: "NO DIET",
        },
        {
          name: "Honeycomb",
          calories: 408,
          fat: 3.2,
          carbs: 87,
          protein: 6.5,
          status: "NO DIET",
        },
        {
          name: "Donut",
          calories: 452,
          fat: 25.0,
          carbs: 51,
          protein: 4.9,
          status: "FAT DIET",
        },
        {
          name: "KitKat",
          calories: 518,
          fat: 26.0,
          carbs: 65,
          protein: 7,
          status: "FAT DIET",
        },
      ];
      this.dessertsSemFiltro = [
        {
          name: "Frozen Yogurt",
          calories: 159,
          fat: 6.0,
          carbs: 24,
          protein: 4.0,
          status: "DIET",
        },
        {
          name: "Ice cream sandwich",
          calories: 237,
          fat: 9.0,
          carbs: 37,
          protein: 4.3,
          status: "NO DIET",
        },
        {
          name: "Eclair",
          calories: 262,
          fat: 16.0,
          carbs: 23,
          protein: 6.0,
          status: "DIET",
        },
        {
          name: "Cupcake",
          calories: 305,
          fat: 3.7,
          carbs: 67,
          protein: 4.3,
          status: "NO DIET",
        },
        {
          name: "Gingerbread",
          calories: 356,
          fat: 16.0,
          carbs: 49,
          protein: 3.9,
          status: "DIET",
        },
        {
          name: "Jelly bean",
          calories: 375,
          fat: 0.0,
          carbs: 94,
          protein: 0.0,
          status: "NO DIET",
        },
        {
          name: "Lollipop",
          calories: 392,
          fat: 0.2,
          carbs: 98,
          protein: 0,
          status: "NO DIET",
        },
        {
          name: "Honeycomb",
          calories: 408,
          fat: 3.2,
          carbs: 87,
          protein: 6.5,
          status: "NO DIET",
        },
        {
          name: "Donut",
          calories: 452,
          fat: 25.0,
          carbs: 51,
          protein: 4.9,
          status: "FAT DIET",
        },
        {
          name: "KitKat",
          calories: 518,
          fat: 26.0,
          carbs: 65,
          protein: 7,
          status: "FAT DIET",
        },
      ];
    },

    printWindow: function () {
      window.print();
    },

    initFilters() {
      for (let col in this.filters) {
        this.filters[col] = this.desserts
          .map((d) => {
            return d[col];
          })
          .filter((value, index, self) => {
            return self.indexOf(value) === index;
          });
      }
      this.activeFilters = Object.assign({}, this.filters);
    },

    toggleAll(col) {
      this.activeFilters[col] = this.desserts
        .map((d) => {
          return d[col];
        })
        .filter((value, index, self) => {
          return self.indexOf(value) === index;
        });
    },

    clearAll(col) {
      this.activeFilters[col] = [];
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      const index = this.desserts.indexOf(item);
      confirm("Are you sure you want to delete this item?") &&
        this.desserts.splice(index, 1);
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.editedItem);
      } else {
        this.desserts.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
