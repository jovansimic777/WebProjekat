<template>
  <div class="overflow-auto">
  
    <b-pagination class="p"
        v-model="currentPage"
        :total-rows="rows"
        :per-page="perPage"
        aria-controls="my-table"
      ></b-pagination>

      <p class="mt-3">Current Page: {{ currentPage }}</p>

    <b-table
      id="my-table"
      :items="items[0]"
      :fields="fields"
      @row-clicked = "rowClicked"
      small
      :per-page="perPage"
      :current-page="currentPage"
    >
    
    <template #cell(actions)="row">
        <b-button size="sm" @click="izmeni(row.item)" class="mr-1">
            Izmeni
        </b-button>
        <b-button size="sm" @click="obrisi(row.item)">
            Obrisi
        </b-button>
      </template>


    </b-table>

    <br>

    <button @click="clicked()">Dodaj vest</button>
    <br><br>

    <div v-if="this.form.clicked == 1">
        <b-form @submit.stop.prevent id="formId">

        <b-form-group label="Naslov:" label-for="title" class="grp">
        <b-form-input id="title" v-model="form.title" required></b-form-input>
        </b-form-group>

        <b-form-group label="Tekst:" label-for="content" class="grp">
        <b-form-input id="content" v-model="form.content" required></b-form-input>
        </b-form-group>

        <b-form-group label="Tagovi:" label-for="tags" class="grp">
        <b-form-input id="tags" v-model="form.tags" required></b-form-input>
        </b-form-group>

        <b-form-group label="Kategorija:" label-for="category" class="grp">
        <b-form-select id="tags" v-model="selectedCategory" :options="this.categoryArr" required></b-form-select>
        </b-form-group>

        <br>
        <b-button @click="onSubmit" type="submit" variant="primary">Unesi</b-button>
        
        <br>
    </b-form>
    </div>

    <div v-if="this.form.clickedUpdate == 1">
        <b-form @submit="update" id="formId">

        <b-form-group label="Naslov:" label-for="title" class="grp">
        <b-form-input id="title" v-model="form.title" required></b-form-input>
        </b-form-group>

        <b-form-group label="Tekst:" label-for="content" class="grp">
        <b-form-input id="content" v-model="form.content" required></b-form-input>
        </b-form-group>

        <b-form-group label="Tagovi:" label-for="tags" class="grp">
        <b-form-input id="tags" v-model="form.tags" required></b-form-input>
        </b-form-group>

        <b-form-group label="Kategorija:" label-for="category" class="grp">
        <b-form-select id="tags" v-model="selectedCategory" :options="this.categoryArr" required></b-form-select>
        </b-form-group>

        <br>
        <b-button type="submit" variant="primary">Promeni</b-button>
        
        <br>
        </b-form>
    </div>

  </div>
</template>

<script>

  import { mapActions, mapState } from 'vuex';

  export default {
    name: 'VestiList',

    data() {
      return {
        fields: [
          {
            key: 'naslov',
            sortable: false
          },
          {
            key: 'text',
            sortable: false
          },
          {
            key: 'vreme_kreiranja',
            label: 'Datum kreiranja',
            sortable: false
          },
          {
            key: 'broj_poseta',
            label: 'Broj poseta',
            sortable: false
          },
          {
            key: 'actions',
            label: 'Actions'
          }
        ],
        items: [],
        currentPage: 1,
        perPage: 2,
        clickded: 0,
        clickedUpdate: 0,
        form:{
            clicked: 0,
            clickedUpdate: 0,
            title:'',
            content:'',
            tags: '',
            
        },
        selectedCategory: '',
        categoryArr: [],
        tagovi: []
      }
    },

    computed: {
      ...mapState([
         'vestiDatum',
         'kategorijeList',
         'tagoviVest'
      ]),
      rows() {
        return this.vestiDatum.length
      }
    },


    mounted() {
    
        this.fetchVestiDatum().then(data => {
            this.items.push(data);
        })

        this.fetchKategorijeList().then(data => {
            
            data.forEach(el => {

              this.categoryArr.push(el.ime)

            })
            
            this.selectedCategory = data[0].ime
        })
        
        
      
        
    },

    methods: {
      ...mapActions([
        'fetchVestiDatum',
        'updateBrojPoseta',
        'fetchKategorijeList',
        'addVest',
        'getTagoviVest',
        'addTag',
        'addVestTag',
        'deleteVest'
      ]),

      clicked(){

            this.form.clickedUpdate = 0;
            this.form.title = '';
            this.form.content = '';
            this.form.tags = '';

            if(this.form.clicked == 1){
                this.form.clicked = 0;
            }else{
                this.form.clicked = 1;
            }

        },
      obrisi(item){

        this.deleteVest(item.id);
        window.location.reload();

      },
        izmeni(item){

          console.log(item);

            this.form.clicked = 0;
            this.form.title = item.naslov;
            this.form.content = item.text;
            //this.form.tags = item.tags;

            this.fetchKategorijeList().then(data => {
            
              data.forEach(el => {

                if(el.id == item.fk_kategorija){
                  this.selectedCategory = el.ime
                }

              })
            
            }) // kategorija

            this.getTagoviVest(item.id).then(data => {

              var tagZ = "";
              //console.log(data);
              data.forEach(el => {

                //console.log(el);
                tagZ = tagZ + el.tag + " ";          

              })
              this.form.tags = tagZ;

            })

            if(this.form.clickedUpdate == 1){
                this.form.clickedUpdate = 0;
            }else{
                this.form.clickedUpdate = 1;
            }

        },

        onSubmit(){

            var id;

            this.fetchKategorijeList().then(data => {
            
              data.forEach(el => {

                //console.log(el.ime);
                //console.log(this.selectedCategory);

                if(el.ime == this.selectedCategory){
                  
                    id = el.id
                    vest.fk_kategorija = id

                    this.addVest(vest);

                    var tagZ = this.form.tags;
          
                    const myArray = tagZ.split(" ");

                    myArray.forEach( el => {
                        var send = {
                          tag: el
                        }
                      this.addTag(send);
                    })


                    this.getTagoviVest(id).then(data => {

          
                      data.forEach(el => {
                        
                        myArray.forEach( eli => {
                          console.log("");
                          if(eli == el.tag){
                                console.log("USAO U IF");
                                var slanje = {
                                vest_id : id,
                                tag_id: el.id
                                }  
                                console.log(slanje);
                                this.addVestTag(slanje); 
                          }

                        })
                        

                      })
    

                    })

                    



                    window.location.reload();

                }

              })
            
            })


            var vest = {

                naslov: this.form.title,
                text: this.form.content,
                vremeKreiranja: "2023-06-13",
                brojPoseta: 0,
                fk_korisnik: parseInt(localStorage.userId),
                fk_kategorija: id,

          }


          

        },

        update(){
            console.log("USAO U UPDATE");
        },

      rowClicked(item){


          var vest = {

            id: item.id,
            naslov: item.naslov,
            text: item.text,
            vremeKreiranja: item.vreme_kreiranja,
            brojPoseta: item.broj_poseta,
            fk_korisnik: 1,
            fk_kategorija: item.fk_kategorija,

          }

          this.updateBrojPoseta(vest)

          console.log(item);

          let routeData = this.$router.resolve({name: 'ClickedVest', params: {id: item.id }});
            window.open(routeData.href, '_blank');
            window.location.reload();

          

      }

     
    }
  }

</script>

<style scoped>

.p{
        text-align: center;
        justify-content: center;
    }

</style>