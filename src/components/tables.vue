<template>
  <div id="app">
  <v-app id="inspire">
     
    <v-data-table
      :headers="headers"
      :items="desserts"
      :search="search"
      
      class="elevation-1"
    >
      <template v-slot:top> 
        <v-card-title>
        <v-spacer></v-spacer>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="ค้นหาเมนู"
          single-line
          hide-details
        ></v-text-field>
        </v-card-title>
        <v-toolbar
          flat
        >
          <v-toolbar-title class="text-h5">Menu</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="primary"
                dark
                class="mb-2"
                v-bind="attrs"
                v-on="on"
              >
                เพิ่มเมนู
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="menu.name_product"
                        label="ชื่อเมนู"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="menu.price_product"
                        label="ราคา(บาท)"
                        required
                        type="number"
                        min=1
                      ></v-text-field>
                    </v-col>
                     <v-container>
                    <v-col cols="12"
                      sm="6"
                      md="4">
                          <v-row v-for="desserts2 in desserts2" :key="desserts2">
                            <v-checkbox
                              v-model="desserts2.id_ingre"
                              item-text="name_ingre"
                              attach
                              item-value="id_ingre"
                              hide-details
                              class="shrink mr-2 mt-0"
                              return-object
                            ></v-checkbox>{{desserts2.name_ingre}}
                            <v-text-field
                              label= 'กรัม'
                              item-value="id_ingre"
                              v-model="quantity"
                            ></v-text-field>
                          </v-row>
                    </v-col>
                     </v-container>
                  </v-row>
                </v-container>
              </v-card-text>
  
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  class="ma-2"
                  color="primary"
                  dark
                  @click="sendData"
                >
                  บันทึก
                  <v-icon
                    dark
                    right
                  >
                    mdi-checkbox-marked-circle
                  </v-icon>
                </v-btn>

                 <v-btn
                  class="ma-2"
                  color="red"
                  dark
                  @click="close"
                >
                  ยกเลิก
                  <v-icon
                    dark
                    right
                  >
                    mdi-cancel
                  </v-icon>
                </v-btn>
              
              </v-card-actions>
            </v-card>
            
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5 mb-6 text-center" >ต้องการลบหรือไม่</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        > 
          mdi-pencil
        </v-icon>
        <v-icon
          small
          v-on:click="reloadPage(),deletetable(item.id_product)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <!-- <v-btn
          color="primary"
          @click="initialize"
        >
          Reset
        </v-btn> -->
      </template>
    </v-data-table>
  </v-app>
</div>
</template>

<script>
import axios from 'axios';
  export default {
    name: 'Addmenu',
data() {
  return {
    search: '',
    dialog: false,
    dialogDelete: false,
    enabled: false,
    menu:{ 
     name_product:"",
     price_product :"",
     quantity :""
    },
    headers: [
      {
        text: 'ชื่อเมนู',
        align: 'start',
        sortable: false,
        value: 'name_product',
      },
      // { text: 'เนื้อหมู(กรัม)', value: 'pig' },
      // { text: 'เนื้อวัว(กรัม)', value: 'cow' },
      // { text: 'เนื้อไก่(กรัม)', value: 'chicken' },
      // { text: 'ตับ(กรัม)', value: 'liver' },
      // { text: 'กุ้ง(ตัว)', value: 'shrimp' },
      // { text: 'ปลาหมึก(กรัม)', value: 'squid' },
      // { text: 'ลูกชิ้น(ลูก)', value: 'meatball' },
      // { text: 'เส้น(กรัม)', value: 'noodle' },
      { text: 'ราคา(บาท)', value: 'price_product'},
      { text: 'Actions', value: 'actions', sortable: false },
    ],
    desserts: [],
    desserts2: [],
    editedIndex: -1,
    // menu: {
    //   name12: '',
    //   pig: '',
    //   cow: '',
    //   chicken: '',
    //   liver : '',
    //   shrimp: '',
    //   squid: '',
    //   meatball: '',
    //   noodle: '',
    //   price : ''
    // },
    defaultItem: {
      name_product: 'ก๋วยเตี๋ยว',
      price_product: ''
    },
}},

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'เพิ่มเมนู ' : 'แก้ไขเมนู'
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    },
  },

  created () {
    // this.initialize()
    this.consultaritems()
    this.consultaritems2()
    this.deletetable();
  },
 

  methods: {
    // initialize () {
    //   this.desserts = [
    //     {
    //       name12: 'TestMenu น้ำใส',
    //       pig: 200,
    //       cow: "-",
    //       chicken: 100,
    //       liver : 200,
    //       shrimp: 2,
    //       squid: 100,
    //       meatball: 4,
    //       noodle: 200,
    //       price : 70,
    //     }, 
    //   ]
    // },

    editItem (item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.menu = Object.assign({}, item)
      this.dialog = true
    },
    checkbox(){
      this.desserts2.id_ingre
      this.enabled = true
    },

    deleteItem (item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.menu = Object.assign({}, item)
      this.dialogDelete = true
    },

    deleteItemConfirm () {
      this.desserts.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.menu = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.menu = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    save () {
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.menu)
      } else {
        this.desserts.push(this.menu)
      }
      this.close()
    },
    sendData(e){
        e.preventDefault();
        console.log("ส่งข้อมูลแล้ว")
        if(this.menu.name_product  != '' && this.menu.price_product ){
        axios.post("http://localhost/menunoodle/menuapi.php", {
        name_product:this.menu.name_product,
        price_product:this.menu.price_product,
        // id_ingre:this.desserts2.id_ingre,
        // quantity:this.menu.quantity
        
        })
        this.save();this.dialog=false;this.reloadPage()
    }},
    consultaritems(){
            fetch('http://localhost/menunoodle/menuapi2.php')
            .then(respuesta=>respuesta.json())
            .then((datosRespuesta)=>{
                console.log(datosRespuesta)
                // this.menulist=[]
                // this.menulist.calories=Response
                if(typeof datosRespuesta[0].success==='undefined'){
                    this.desserts=datosRespuesta;
                }
            }).catch(console.log)
        },
         deletetable(id_product){
         console.log(id_product);
         fetch('http://localhost/menunoodle/menuapi2.php/?deletetablee='+id_product)
            .then(respuesta=>respuesta.json())
            .then((desserts)=>{
              console.log(desserts)
              
            }).catch(console.log)
          },
          consultaritems2(){
            fetch('http://localhost/menunoodle/menuapi8.php')
            .then(respuesta=>respuesta.json())
            .then((datosRespuesta)=>{
                console.log(datosRespuesta)
                // this.menulist=[]
                // this.menulist.calories=Response
                if(typeof datosRespuesta[0].success==='undefined'){
                    this.desserts2=datosRespuesta;
                }
            }).catch(console.log)
        },
          reloadPage() {
          window.location.reload();
          },
  }
}
</script>