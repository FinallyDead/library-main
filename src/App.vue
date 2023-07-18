<template>
<div class="row mb-5 align-items-start">
  <div class="logo">
    <img src="./assets/logoFinal.png" class="" alt="Responsive image" v-if="page == 1">
  </div>
  <div class="col-sm me-5">
    <div class="d-flex">
      <input v-if="page == 1" type="file" class="form-control inputFile" @change="onChange" />
      <div v-if="page == 1" style="margin-top: 48px;">
        <label class="d-inline">Кол-во записей на странице:</label>  
        <input 
        class="form-control d-inline ms-2" 
        style="width: 15%;"
        type="text" 
        :placeholder="this.limit" 
        @keydown.enter="this.limit = parseInt($event.target.value)"
        @keyup.enter="$event.target.value = '' "
        />
      </div>
    </div>
    <div style="margin-top: 5px;">
      <div class="d-flex">
        Количество страниц: {{ this.totalPages }}
        <div v-if="searchedCollections.length == 0 && searchQuery == '' || searchOption == -1"  style="margin-left: 9.2%;">
          Количество записей: {{ this.collections.length }}
        </div>
        <div v-if="searchedCollections.length >= 0 && searchQuery != '' && searchOption != -1" class="ms-5">
          Количество записей: {{ this.searchedCollections.length }}
        </div>  
      </div>
      
      <div v-if="collections.length > 0 && page == 1">
        <label class="d-inline">Сортировка по:</label>
        <select class="form-select d-inline w-auto ms-2" v-model="sortOption" @change="sortBooks">
          <option selected value="0"> Отсутствует </option>
          <option
          v-for="keyFromArr in selectValues" 
          :key="keyFromArr" 
          :value="keyFromArr"
          >{{ keyFromArr }}</option>
        </select>
        <div class="ms-2 d-inline">
          <label>Поиск по столбцу:</label>
          <span v-if="this.searchOption != 'category' && this.searchOption != 'themen'">
            <input 
            v-model="searchQuery"
            class="form-control d-inline ms-2"
            style="width: 30%;" 
            placeholder="Поиск по..."
            @input="search"
            id="searchByInput"
            />
          </span>
          <select 
          v-if="collections.length > 0 && page == 1 && this.searchOption == 'category'" 
          class="form-select d-inline w-auto ms-2" 
          v-model="searchQuery"
          @change="search"
          id="searchByCategory"
          >
            <option
            v-for="str in categoryArray"
            :key="str"
            :value="str"
            >{{ str }} </option>
          </select>
          <select 
          v-if="collections.length > 0 && page == 1 && this.searchOption == 'themen'" 
          class="form-select d-inline w-auto ms-2" 
          v-model="searchQuery"
          @change="search"
          id="searchByThemen"
          >
            <option
            v-for="str in themenArray"
            :key="str"
            :value="str"
            >{{ str }} </option>
          </select>
          <select v-model="searchOption" class="form-select d-inline w-auto ms-2" @change="search">
            <option 
            v-for="str in keyNames" 
            :key="str"
            :value="str"
            > {{ str }}</option>
          </select>
        </div>
      </div>
      <div v-if="this.collectionsPage.length > 0 && page == 1">
        <button class="_btn" @click="if(categoryAttr == 1){categoryAttr = 0; themenAttr = 0;}else{categoryAttr = 1; themenAttr = 0;}">Список категорий</button>
        <button class="_btn ms-2" @click="if(themenAttr == 1){categoryAttr = 0; themenAttr = 0;}else{categoryAttr = 0; themenAttr = 1;}" >Список тем</button>
        <button class="_btn ms-2" @click="exportToExcel('xlsx')"> Сохранить таблицу </button>
      </div>
    </div>
  </div>

  <div v-if="categoryAttr == 1" style="margin-left: 15px;">
    <button class="_btn" @click="if(addCategoryAttr == 1){addCategoryAttr = 0; deleteCategoryAttr = 0;}else{addCategoryAttr = 1; deleteCategoryAttr = 0;} this.addedCategory='';">Добавить категорию</button>
    <button class="_btn" style="margin-left: 10px;" @click="if(deleteCategoryAttr == 1){addCategoryAttr = 0; deleteCategoryAttr = 0;}else{addCategoryAttr = 0; deleteCategoryAttr = 1;} this.addedCategory='';">Удалить категорию</button>
    <div v-if="addCategoryAttr == 1">
      <input class="form-control input" style="width: 40%; margin-left: 1px;" type="text" placeholder="Категория для добавления" v-model="this.addedCategory">
      <button class="_btn" @click="this.categoryArray.push(this.addedCategory); this.addCategoryAttr = 0;">Добавить категорию</button>
    </div>
    <div v-if="deleteCategoryAttr == 1">
      <select v-model="this.deletedCategory" class="form-select from_select" style="width: 50%;">
          <option 
          v-for="str in categoryArray" 
          :key="str"
          :value="str"
          > {{ str }}</option>
        </select>
      <button class="_btn" @click="deleteCategoryFromArray">Удалить категорию</button>
    </div>
    <div v-for="str in categoryArray" :key="str">
      <label>{{ str }}</label>
    </div>
  </div>

  <div v-if="themenAttr == 1" style="margin-left: 15px;">
    <button class="_btn" @click="if(addThemenAttr == 1){addThemenAttr = 0; deleteThemenAttr = 0;}else{addThemenAttr = 1; deleteThemenAttr = 0;} this.addedThemen='';">Добавить тему</button>
    <button class="_btn" style="margin-left: 10px;" @click="if(deleteThemenAttr == 1){addThemenAttr = 0; deleteThemenAttr = 0;}else{addThemenAttr = 0; deleteThemenAttr = 1;} this.addedThemen='';">Удалить тему</button>
    <div v-if="addThemenAttr == 1">
      <input class="form-control input" style="width: 40%; margin-left: 1px;" type="text" placeholder="Тема для добавления" v-model="this.addedThemen">
      <button class="_btn" @click="this.themenArray.push(this.addedThemen); this.addThemenAttr = 0;">Добавить тему</button>
    </div>
    <div v-if="deleteThemenAttr == 1">
      <select v-model="this.deletedThemen" class="form-select from_select" style="width: 50%;">
          <option 
          v-for="str in themenArray" 
          :key="str"
          :value="str"
          > {{ str }}</option>
        </select>
      <button class="_btn" @click="deleteThemenFromArray">Удалить Тему</button>
    </div>
    <div v-for="str in themenArray" :key="str">
      <label>{{ str }}</label>
    </div>
  </div>
</div>
<div v-if="this.selectedRow && !(collections.length > 0 && collectionsPage.length == 0)" style="margin-left: 15px;" >
  
    <button class="_btn " @click="this.addVisible = 1; this.changeVisible = 0; this.closeAttr = 0;">Добавить</button>
    <button class="_btn " style="margin-left:15px;" @click="this.changeVisible = 1; this.addVisible = 0; this.closeAttr = 0;">Изменить</button>
    <button class="_btn " style="margin-left:15px;" @click="removeFromTable">Удалить</button>
  
</div>

<p v-if="errors.length" style="margin-top: 10px;" class="waitLabel">
    <b>Пожалуйста исправьте указанные ошибки:</b>
    <ul>
      <li style="list-style-type: none;" v-for="error in errors" :key="error">{{ error }}</li>
    </ul>
</p>

<form class="form" v-if="this.addVisible == 1 && closeAttr == 0" @submit.prevent>
  <div style="margin-top: 15px; align-self: flex-end;">
    <button class="btn_in_div d-inline" style="background-color: rgba(28,28,28,0); border: 0;  font-weight: 600;" @click="this.closeAttr = 1; this.addVisible = 0">X</button>
  </div>
  <input class="form-control input" type="text" placeholder="Автор(-ы)" v-model="this.inputAuthor" />
  <input class="form-control input" type="text" placeholder="Название" v-model="this.inputName" />
  <input class="form-control input" type="number" placeholder="Шкаф" v-model="this.inputCloset" />
  <input class="form-control input" type="number" placeholder="Год" v-model="this.inputYear" />
  <select v-model="this.inputThemen" class="form-select from_select">
          <option value="" disabled selected>Тема</option>
          <option 
          v-for="str in themenArray" 
          :key="str"
          :value="str"
          > {{ str }}</option>
        </select>
  <select v-model="this.inputCategory" class="form-select from_select">
          <option value="" disabled selected>Категория</option>
          <option 
          v-for="str in categoryArray" 
          :key="str"
          :value="str"
          > {{ str }}</option>
        </select>
  <div style="margin-top: 15px; align-self: flex-end;">
    <button class="btn_in_div d-inline" @click="addIntoTable">Записать</button>
  </div>
</form>

<form class="form" v-if="this.changeVisible == 1 && closeAttr == 0" @submit.prevent>
  <div style="margin-top: 15px; align-self: flex-end;">
    <button class="btn_in_div d-inline" style="background-color: rgba(28,28,28,0); border: 0;  font-weight: 600;" @click="this.closeAttr = 1; this.changeVisible = 0; ">X</button>
  </div>
  <input ref="changeAuthor" class="form-control input" type="text" placeholder="Автор(-ы)"  :value="this.collections[this.selectedRow-1].author"/>
  <input ref="changeName" class="form-control input" type="text" placeholder="Название"  :value="this.collections[this.selectedRow-1].name"/>
  <input ref="changeCloset" class="form-control input" type="number" placeholder="Шкаф"  :value="this.collections[this.selectedRow-1].closet" />
  <input ref="changeYear" class="form-control input" type="number" placeholder="Год"  :value="this.collections[this.selectedRow-1].year"/>
  <select ref="changeCategory" class="form-select from_select" :value="this.collections[this.selectedRow-1].category">
          <option 
          v-for="str in categoryArray" 
          :key="str"
          > {{ str }}</option>
        </select>
  <select ref="changeThemen" class="form-select from_select" :value="this.collections[this.selectedRow-1].themen">
          <option 
          v-for="str in themenArray" 
          :key="str"
          > {{ str }}</option>
        </select>
  <div style="margin-top: 15px; align-self: flex-end;">
    <button class="btn_in_div d-inline"  @click="changeIntoTable">Записать изменения</button>
  </div>
</form>
<div class="page__wrapper" v-if="totalPages > 1">
  <div 
  v-if="this.page > 1"
  class="page"
  @click="changeToEdgeOfPages('lower')"
  >
    &lt;&lt;&lt;
  </div>
  <div
  v-if="this.page > 1"
  class="page"
  @click="this.changePageWithButton('lower')"
  >
    &lt;&lt;
  </div>
  <div 
  v-for="pageNumber in totalPages"
  :key="pageNumber"
  :value='pageNumber * this.limit'
  @click="event => changePage(event, pageNumber)"
  >
    <div
    v-if="!((pageNumber > page+3) || (page-3 > pageNumber)) 
    && totalPages >= 20 
    || (page === totalPages && pageNumber > page - 7)
    || (page === totalPages-1 && pageNumber > page - 6)
    || (page === totalPages-2 && pageNumber > page - 5)
    || (page === 1 && pageNumber < page + 7)
    || (page === 2 && pageNumber < page + 6)
    || (page === 3 && pageNumber < page + 5)"
    class="page"
    :class="{
      'current-page': page === pageNumber
    }"
    >
      {{ pageNumber }}
    </div>
    <div 
    v-else-if="totalPages < 20"
    class="page"
    :class="{
      'current-page': page === pageNumber
    }" 
    >
    {{ pageNumber }}
    </div>
</div>
  <div
  v-if="this.page < this.totalPages"
  class="page"
  @click="this.changePageWithButton('higher')"
  >
    >>
  </div>
  <div 
  v-if="this.page < this.totalPages"
  class="page"
  @click="changeToEdgeOfPages('higher')"
  >
    >>>
  </div>
</div> 
 <section v-if="collectionsPage.length > 0" class="tableSec">
    <table id="tbl_exporttable_to_xls" class="table table-bordered table-striped">
      <thead>
        <tr>
          <th 
          v-for="header in keyNames" 
          :key="header"
          >
            {{ header }}
          </th>
        </tr>  
      </thead>
      <tbody >
        <tr 
        v-for="strings in collectionsPage" 
        :key="strings.id"
        @click="selectRow"
        :class="{
          'activeItem': strings.id === this.selectedRow
        }"
        style="user-select: none;"
        >
          <td 
          v-for="(item,key) in strings" 
          :key="key" 
          :value="item"
          >
            {{ item }}
          </td>
        </tr>
      </tbody>
    </table> 
  </section>
  <div class="waitLabel" v-if="collections.length == 0">
      Выберите файл для отображения
  </div>
  <div class="waitLabel" v-if="collections.length > 0 && collectionsPage.length == 0">
      Ничего не найдено
  </div>
  <div class="page__wrapper" v-if="totalPages > 1">
    <div 
    v-if="this.page > 1"
    class="page"
    @click="changeToEdgeOfPages('lower')"
    >
      &lt;&lt;&lt;
    </div>
    <div
    v-if="this.page > 1"
    class="page"
    @click="this.changePageWithButton('lower')"
    >
      &lt;&lt;
    </div>
    <div 
    v-for="pageNumber in totalPages"
    :key="pageNumber"
    :value='pageNumber * this.limit'
    @click="event => changePage(event, pageNumber)"
    >
      <div
      v-if="!((page-3 > pageNumber) || (pageNumber > page+3)) 
      && totalPages >= 20 
      || (page === totalPages && pageNumber > page - 7)
      || (page === totalPages-1 && pageNumber > page - 6)
      || (page === totalPages-2 && pageNumber > page - 5)
      || (page === 1 && pageNumber < page + 7)
      || (page === 2 && pageNumber < page + 6)
      || (page === 3 && pageNumber < page + 5)"
      class="page"
      :class="{
        'current-page': page === pageNumber
      }"
      >
        {{ pageNumber }}
      </div>
      <div 
      v-else-if="totalPages < 20"
      class="page"
      :class="{
        'current-page': page === pageNumber
      }" 
      >
      {{ pageNumber }}
      </div>
  </div>
    <div
    v-if="this.page < this.totalPages"
    class="page"
    @click="this.changePageWithButton('higher')"
    >
      >>
    </div>
    <div 
    v-if="this.page < this.totalPages"
    class="page"
    @click="changeToEdgeOfPages('higher')"
    >
      >>>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";
import "bootstrap/dist/css/bootstrap.min.css";


export default {
  data() {
    return {
      file: null,
      fileName: "",
      keyNames: [],
      collections: [],
      collectionsPage: [],
      currentIndex: 0,
      pastIndex: 0,
      selectedRow: null,
      page: 1,
      limit: 25,
      totalPages: 0,
      pagesPerBlock: 10,
      inputAuthor: "",
      inputName: "",
      inputCloset: "",
      inputYear: "",
      inputThemen: "",
      inputCategory: "",
      sortOption: 0,
      selectValues: [],
      searchQuery: "",
      searchOption: -1,
      searchedCollections: [],
      sortedCollections: [],
      closeAttr: 0,
      categoryArray: [],
      themenArray: [],
      addVisible: 0,
      changeVisible: 0,
      categoryAttr: 0,
      themenAttr: 0,
      addedCategory: "",
      addedThemen: "",
      deletedCategory: "",
      deletedThemen: "",
      addCategoryAttr: 0,
      addThemenAttr: 0,
      deleteCategoryAttr: 0,
      deleteThemenAttr: 0,
      errors: [],
      internationalNames:{
        "номер":"id",
        "автор":"author",
        "название":"name",
        "год":"year",
        "тема":"themen",
        "категория":"category",
        "шкаф":"closet"
    }
    };
  },
  methods: {
    onChange(event) {
      this.file = event.target.files ? event.target.files[0] : null;
      this.fileName = this.file.name;
      const reader = new FileReader();
      reader.onload = (e) => {
              
              const bstr = e.target.result;
              const wb = XLSX.read(bstr, { type: "binary" });
              
              const wsname = wb.SheetNames[0];
              const ws = XLSX.utils.sheet_to_row_object_array(wb.Sheets[wsname]);
              
              let tempArr = [];
              this.collections = JSON.parse(JSON.stringify(ws));
              
              if ("limit" in this.collections[0]){
                this.limit = this.collections[0].limit;
                delete this.collections[0].limit;
              }
                 
              tempArr = this.collections.map(x => x.category);
              tempArr =Array.from(new Set(tempArr))
              this.categoryArray = tempArr.filter(x => {
                return x !== undefined
              }).slice(0)

              tempArr = this.collections.map(x => x.themen);
              tempArr =Array.from(new Set(tempArr))
              this.themenArray = tempArr.filter(x => {
                return x !== undefined
              }).slice(0)

              this.keyNames = Object.keys(this.collections[0]);
              this.selectValues = this.keyNames.slice(1);
              this.totalPages = Math.ceil(this.collections.length / this.limit);
              this.currentIndex = this.limit;
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
              this.keyNames.forEach(x =>{
                (x == 'id') && (this.keyNames[this.keyNames.indexOf(x)] ='номер') ||
                (x == 'author') && (this.keyNames[this.keyNames.indexOf(x)] ='автор') ||
                (x == 'name') && (this.keyNames[this.keyNames.indexOf(x)] ='название') ||
                (x == 'year') && (this.keyNames[this.keyNames.indexOf(x)] ='год') ||
                (x == 'themen') && (this.keyNames[this.keyNames.indexOf(x)] ='тема') ||
                (x == 'category') && (this.keyNames[this.keyNames.indexOf(x)] ='категория') ||
                (x == 'closet') && (this.keyNames[this.keyNames.indexOf(x)] ='шкаф')  
              })
            };

            reader.readAsBinaryString(this.file);
            this.file=null;
            
    },
    selectRow(e){
      this.selectedRow = parseInt(e.currentTarget.querySelector('td').innerText);
      this.closeAttr = 0;
    },
    changeIndexOfPage(targetValue){
      if(this.currentIndex < targetValue){
        this.pastIndex = targetValue - this.limit;
        this.currentIndex = parseInt(targetValue);
        if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
        this.selectedRow = null;
      }
      else{
        this.currentIndex = parseInt(targetValue);
        this.pastIndex = this.currentIndex - this.limit;
        if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
        this.selectedRow = null;
      }
    },
    changePage(e, pageNumber){
        this.page = pageNumber;
        let targetValue = e.currentTarget.getAttribute("value");
        this.changeIndexOfPage(targetValue);
    },
    changePageWithButton(side){
      switch(side){
          case "lower":
            --this.page;
            this.currentIndex = parseInt(this.pastIndex);
            this.pastIndex = this.currentIndex - this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;
          case "higher":
            ++this.page;
            this.pastIndex = parseInt(this.currentIndex);
            this.currentIndex = parseInt(this.currentIndex) + parseInt(this.limit);
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;  
        }
    },
    changeToEdgeOfPages(side){
        switch(side){
          case "lower":
            this.page = 1;
            this.pastIndex = 0;
            this.currentIndex = this.page * this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;
          case "higher":
            this.page = this.totalPages;
            this.currentIndex = this.page * this.limit;
            this.pastIndex = this.currentIndex - this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;  
        }
    },
    addIntoTable(){
      if (this.inputAuthor && this.inputName && this.inputCloset && this.inputYear && this.inputThemen && this.inputCategory){
        this.errors = [];
        let lastElement = this.collections.at(-1);
        this.collections.push({id: lastElement.id+1 , author: this.inputAuthor, name: this.inputName, 
          year: this.inputYear, category: this.inputCategory , themen: this.inputThemen, closet: this.inputCloset});
        this.selectedRow = null;
        this.inputAuthor = "";
        this.inputName = "";
        this.inputCloset = "";
        this.inputYear = "";
        this.inputThemen = "";
        this.inputCategory = "";
        
        this.addVisible = 0;
        this.changeVisible = 0;

        if (this.sortOption == 0 && this.searchQuery == ""){
          this.totalPages = Math.ceil(this.collections.length / this.limit);
          this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
        }
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else{
                this.totalPages = Math.ceil(this.sortedCollections.length / this.limit);
                this.sortBooks();
              }
      }else{
        this.errors = []
        if (!this.inputAuthor)
          this.errors.push("Заполните имя(-ена) автора(-ов)");
        if (!this.inputName)
          this.errors.push("Заполните название книги");
        if (!this.inputCloset)
          this.errors.push("Заполните номер шкафа");
        if (!this.inputYear)
          this.errors.push("Заполните год выпуска");
        if (!this.inputThemen)
          this.errors.push("Выберите тему");
        if (!this.inputCategory)
          this.errors.push("Выберите категорию");
      }
    },
    removeFromTable(){
      let arrPart = this.collections.splice(this.collections.indexOf(this.collections.find(x => x.id == this.selectedRow))+1,
      this.collections.length);
      this.collections.pop();
      arrPart.forEach(element => { 
        element.id--;
        this.collections.push(element);
      });
      this.selectedRow = null;
      this.totalPages = Math.ceil(this.collections.length / this.limit);
      this.addVisible = 0;
      this.changeVisible = 0;
      if (this.sortOption == 0){
        if (this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex).length>0){
         this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex)
        }else{
         this.collectionsPage = this.collections.filter(p => p.id<=(this.currentIndex-this.limit) && p.id>(this.pastIndex-this.limit))
         --this.page
        }
      }else {
        this.sortBooks();
      }
    },
    changeIntoTable(){
       this.collections[this.selectedRow-1].author = this.$refs.changeAuthor.value;
       this.collections[this.selectedRow-1].name = this.$refs.changeName.value;
       this.collections[this.selectedRow-1].closet = this.$refs.changeCloset.value;
       this.collections[this.selectedRow-1].year = this.$refs.changeYear.value;
       this.collections[this.selectedRow-1].themen = this.$refs.changeThemen.value;
       this.collections[this.selectedRow-1].category = this.$refs.changeCategory.value;
       this.changeVisible = 0;
       this.addVisible = 0;
    },
    isNumeric(n) {
      return !isNaN(parseFloat(n)) && isFinite(n);
    },
    sortBooks() {
      if (this.sortOption == 0 && this.searchQuery == ""){
        this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex)
        this.sortedCollections = []
      }else{
        if (this.sortOption == 0 && this.searchQuery != ""){
            this.search();
        }
        else{
          if (this.searchQuery == "" || this.searchOption == 0){
            this.selectedRow = 0;
            this.sortedCollections = this.collections.slice(0)
            if (this.sortOption=='closet' || this.sortOption=='year')
              this.sortedCollections.sort((book1, book2) => book1[this.sortOption]-book2[this.sortOption]);
            else{
              this.sortedCollections.sort((book1, book2) => book1[this.sortOption].localeCompare(book2[this.sortOption]));
            }
              this.collectionsPage = this.sortedCollections.slice(this.pastIndex, this.currentIndex);
          }
          if (this.searchQuery != "" && this.searchOption != 0){
            this.selectedRow = 0;
            this.sortedCollections = this.searchedCollections.slice(0)
            if (this.sortOption=='closet' || this.sortOption=='year')
              this.sortedCollections.sort((book1, book2) => book1[this.sortOption]-book2[this.sortOption]);
            else
              this.sortedCollections.sort((book1, book2) => book1[this.sortOption].localeCompare(book2[this.sortOption]));
            this.collectionsPage = this.sortedCollections.slice(this.pastIndex, this.currentIndex);
          }
        }
    }
    },
    search(){ 
      //меняем русское слово на английское чтобы потом объект мог вести поиск по своему полю(ибо они на английском)
      this.searchOption=Object.keys(this.internationalNames).includes(this.searchOption)? this.internationalNames[this.searchOption]:this.searchOption

      if (this.searchQuery == "" && this.sortOption == 0){
        this.searchedCollections = [];
        this.totalPages = Math.ceil(this.collections.length / this.limit);
        this.collectionsPage = this.collections.slice(this.pastIndex, this.currentIndex);
        return;
      }
      
      if (this.searchQuery == "" && this.sortOption != 0){
        this.selectedRow = 0;
        this.searchedCollections = [];
        this.sortBooks();
        this.totalPages = Math.ceil(this.sortedCollections.length / this.limit);
      }
      
      if (this.searchOption != -1 && this.sortOption == 0 && this.searchQuery != ""){
        this.selectedRow = 0;
        if(this.searchOption instanceof String){
          this.searchedCollections = this.collections.filter(book => book[this.searchOption].includes(this.searchQuery));
        
        }
        else
          this.searchedCollections = this.collections.filter(book => book[this.searchOption].toString().includes(this.searchQuery));
        this.totalPages = Math.ceil(this.searchedCollections.length / this.limit);
        this.collectionsPage = this.searchedCollections.slice(this.pastIndex, this.currentIndex);
        }
      console.log(321123)
      if (this.searchOption != -1 && this.sortOption != 0 && this.searchQuery != ""){
        this.selectedRow = 0;
        if(this.searchOption instanceof String)
          this.searchedCollections = this.collections.filter(book => book[this.searchOption].includes(this.searchQuery));
        else
          this.searchedCollections = this.collections.filter(book => book[this.searchOption].toString().includes(this.searchQuery));
        this.totalPages = Math.ceil(this.searchedCollections.length / this.limit);
        this.sortBooks()
      }
    },
    deleteCategoryFromArray(){
      let count = 0;
      this.collections.reduce((accumulator,currentValue) =>{

        if (Object.values(currentValue)[4] == this.deletedCategory){
          ++count;
          return;
        }
      }, 0)
      if (count == 0){
        this.deleteCategoryAttr = 0;
        this.categoryArray.splice(this.categoryArray.indexOf(this.deletedCategory),1);
      }
        
      
    },
    deleteThemenFromArray(){
      let count = 0;
      this.collections.reduce((accumulator,currentValue) =>{

        if (Object.values(currentValue)[5] == this.deletedThemen){
          ++count;
          return;
        }
      }, 0)
      if (count == 0){
        this.deleteThemenAttr = 0;
        this.themenArray.splice(this.themenArray.indexOf(this.deletedThemen),1);
      }
    },
    exportToExcel(type, fn){
      new Promise(resolve =>{
        setTimeout(() =>{
          let originOptions = [];
          let origL = this.limit;
          originOptions.push(origL);

          let origSearchOption = this.searchOption;
          originOptions.push(origSearchOption);

          let origSortOption = this.sortOption;
          originOptions.push(origSortOption);

          let origSearchQuery = this.searchQuery;
          originOptions.push(origSearchQuery);

          this.limit = this.collections.length;
          this.searchOption = 0;
          this.sortOption = 0;
          this.searchQuery = "";

          this.collections[0].limit = origL;
          this.keyNames.push("limit")

          resolve(originOptions)
        }, 1000)
      }).then((value) =>{
        let elt = document.getElementById('tbl_exporttable_to_xls');
        let wb = XLSX.utils.table_to_book(elt, { sheet: "sheet1" });
        this.limit = value[0];
        this.searchOption = value[1];
        this.sortOption = value[2];
        this.searchQuery = value[3];
        delete this.collections[0].limit;
        this.keyNames.pop();
        return XLSX.writeFile(wb, fn || (this.fileName));
      }) 
    }
  },
  watch: {
      limit(){
        this.currentIndex = this.limit;
        if (this.sortOption == 0 && this.searchQuery == ""){
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
              this.totalPages = Math.ceil(this.collections.length / this.limit);
        }
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
      }
  },
  
};
</script>

<style>
.logo{
  height: 70%;
  width: fit-content;
  margin-top: 15px;
  margin-left: 15px;
  padding: 0;
}
.page__wrapper{
  display: flex;
  margin-top: 15px;
  margin-bottom: 15px;
  margin-left: 10px;
  margin-right: auto;
}
.page{
  border: 1px solid black;
  padding: 10px;
  margin-left: 5px;
  user-select: none;
}
.page:hover{
  background-color: rgb(0, 211, 211);
  color: rgb(0, 58, 124);
  text-shadow: 0 0 1px rgb(0, 58, 124);
  cursor: pointer;
}
.current-page{
  border: 2px solid teal;
}
.tableSec{
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 98%;
}
.waitLabel{
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  width: 95%;
  font-size: 25px;
  color: red;
}
.inputFile{
  margin-right: 5px;
  margin-top: 48px;
  width: 32%;
  height: 75%;
}
.activeItem{
  border: 10px solid #4285B4;
  background-color: #4285B4;
}
.form{
  display: flex;
  flex-direction: column;
  margin: 20px;
}
.input{
  width: 100%;
  border: 1px solid #4285B4;
  padding: 10px 15px;
  margin-top: 15px;
  margin-left: auto;
  margin-right: auto;
}
._btn{
  margin-top: 15px;
  align-self: flex-end;
  padding: 10px 15px;
  background-color: none;
  color: #4285B4;
  border: 1px solid #4285B4;
}
._btn:hover{
  background-color: #86c5f2;
  color: black;
}
.btn_in_div{
  padding: 10px 15px;
  background-color: none;
  color: #4285B4;
  border: 1px solid #4285B4;
}
.btn_in_div:hover{
  background-color: #86c5f2;
  color: black;
}
.from_select{
  border: 1px solid #4285B4;
  padding: 10px 15px;
  margin-top: 15px;
}
</style>