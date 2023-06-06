<template>

  <main class="table">
        <section class="table__header">
            <h1>Personel Ara</h1>
            <div class="input-group">
                <img src="src/images/search.png" alt="">
                <input v-model="filter" placeholder="İsme göre filtreleyin.." />
            </div>
            <span><p>Toplam Kişi Sayısı: {{ totalItems }}</p>
            
            </span>
            <span @click="selectStatus('')" class="status-filter {{ selectedStatus === '' ? 'active' : '' }}">Hepsi</span>
            <span @click="selectStatus('passive')" class="status-filter {{ selectedStatus === 'passive' ? 'active' : '' }}">Passive</span>
            <span @click="selectStatus('active')" class="status-filter {{ selectedStatus === 'active' ? 'active' : '' }}">Active</span>
            
            
          
    

            <div class="export__file">
    <label for="export-file" class="export__file-btn" title="Export File"></label>
    <input type="checkbox" id="export-file" style="display: none" class="details-popup"/>
    <div class="export__file-options">
      <label for="toEXCEL" id="toEXCEL" @click="exportToExcel">EXCEL <img src="src/images/excel.png" alt=""  /></label>
    </div>
  </div>
          
        </section>
        <section class="table__body">
            <table>
                <thead>
                    <tr>
                        <th> Id </th>
                        <th> Name </th>
                        <th> Status </th>
                        <th> Detay.. </th>
                    </tr>
                </thead>
                <tbody>

        <tr v-for="(item, index) in displayedItems" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.status }}</td>
          <td><span
                @mouseenter="showDetails(item, $event)"
                @mouseleave="hideDetails(item)"
                class="hover-details"
              >
                Kurs detayları...
              </span>
              
              <div v-if="item.showDetails" v-for="item in courses" class="details-popup" :style="{ top: item.popupTop, left: item.popupLeft }">
                <p><li><strong>Kurs Adı: </strong>{{ item.courses.course_name }}</li></p>
                <p><li><strong>Saat - Dk - Sn: </strong>{{ getCurrentTime() }}</li></p>
                <p><li><strong>Kurs Bitirme Tarihi: </strong>{{ item.courses.completed_at }}</li></p>
                
              </div></td>
        </tr>
        
        
              </tbody>
                <nav class="pagination">
      <ul>
        <li>
          <button @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">
            Geri
          </button>
        </li>
        <li v-for="page in totalPages" :key="page">
          <button @click="goToPage(page)" :class="{ active: page === currentPage }">{{ page }}</button>
        </li>
        <li>
          <button @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">
            İleri
          </button>
        </li>
      </ul>
    </nav>
            </table>
        </section>
    </main>
</template>

<script>

import jsonData from './components/jsonData.json'
import coursesData from './components/courses.json';

export default {

  data() {
    return {
      filter: '',
      items: jsonData,
      courses: coursesData,
      currentPage: 1,
      pageSize: 10,
      selectedStatus: '',
      exportToExcel,
    };
  },


  
  computed: {
  filteredItems() {
    return this.items.filter(item => {
      const nameMatch = item.name.toLowerCase().includes(this.filter.toLowerCase());
      const statusMatch = this.selectedStatus === '' || item.status.toLowerCase() === this.selectedStatus;
      return nameMatch && statusMatch;
      
    });
  },
  totalItems() {
    return this.filteredItems.length;
  },
  totalPages() {
    return Math.ceil(this.filteredItems.length / this.pageSize);
  },
  displayedItems() {
    const startIndex = (this.currentPage - 1) * this.pageSize;
    const endIndex = startIndex + this.pageSize;
    return this.filteredItems.slice(startIndex, endIndex);
  },
  filteredItemPage() {
    const index = this.filteredItems.findIndex(item => {
      return item.name.toLowerCase().includes(this.filter.toLowerCase());
    });
    return index === -1 ? 1 : Math.ceil((index + 1) / this.pageSize);
  }
},
watch: {
  filter: function(newFilter) {
    this.currentPage = this.filteredItemPage || 1;
  },
  currentPage: function(newPage) {
    const startIndex = (newPage - 1) * this.pageSize;
    const endIndex = startIndex + this.pageSize;
    const currentPageItems = this.filteredItems.slice(startIndex, endIndex);
    if (!currentPageItems.some(item => item.name.toLowerCase().includes(this.filter.toLowerCase()))) {
      this.filter = '';
    }
  }
},
methods: {
  goToPage(page) {
    this.currentPage = page;
  },
  getCurrentTime() {
    const now = new Date();
    const hours = now.getHours().toString().padStart(2, '0');
    const minutes = now.getMinutes().toString().padStart(2, '0');
    const seconds = now.getSeconds().toString().padStart(2, '0');
    const time = `${hours}:${minutes}:${seconds}`;
    
    
    setTimeout(() => {
      this.$forceUpdate(); 
    }, 1000);
    
    return time;
  },
  selectStatus(status) {
    this.selectedStatus = status;
    this.filterItems();
  },
  filterItems() {
    this.currentPage = 1;
  },
  previousPage() {
    if (this.currentPage > 1) {
      this.currentPage--;
    }
  },
  nextPage() {
    if (this.currentPage < this.totalPages) {
      this.currentPage++;
    }
  },
  showDetails(item) {
      item.showDetails = true;
    },
    hideDetails(item) {
      item.showDetails = false;
    },
    mounted() {
    this.items.forEach(item => {
      item.courses = this.getCoursesByPersonId(item.id);
    });
  },
  methods: {
    getCoursesByPersonId(personId) {
      return this.courses.filter(course => course.personId === personId);
    }
  }
  
}}
function exportToExcel() {
  const tableToExcel = (function () {
    const uri = "data:application/vnd.ms-excel;base64,";
    const template =
      `<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40"><head><meta charset="UTF-8"></head><body><table>{table}</table></body></html>`;
    const base64 = function (s) {
      return window.btoa(unescape(encodeURIComponent(s)));
    };
    const format = function (s, c) {
      return s.replace(/{(\w+)}/g, function (m, p) {
        return c[p];
      });
    };

    return function (table, name) {
      if (!table.nodeType) table = document.getElementById(table);
      const ctx = { worksheet: name || "Worksheet", table: table.innerHTML };
      const href = uri + base64(format(template, ctx));
      const link = document.createElement("a");
      link.href = href;
      link.download = `${name || "table"}.xls`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    };
  })();

  const table = document.querySelector("table");
  tableToExcel(table, "personel_table");
}



</script>

<style>
@import './assets/style.css'
</style>
<style scoped>
.pagination {
  margin-top: 1rem;
  display: flex;
  justify-content: center;
}

.pagination ul {
  list-style: none;
  padding: 0;
  display: flex;
}

.pagination li {
  margin: 0 0.5rem;
}

.pagination button {
  border: none;
  background: none;
  cursor: pointer;
}

.pagination button:disabled {
  color: gray;
  cursor: not-allowed;
}

.pagination button.active {
  font-weight: bold;
}
</style>
<style scoped>
/* ... */
.details-popup {
  position: absolute;
  background-color: white;
  border: 1px solid gray;
  padding: 25px;
  width: 375px;
}
</style>

<style scoped>
.status-filter {
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 5px;
  background-color: #f0f0f0;
  color: #555555;
  margin-right: 5px;
}

.status-filter:last-child {
  margin-right: 0;
}

.status-filter.active {
  background-color: #555555;
  color: #ffffff;
}

.status-filter:not(:last-child)::after {
  content: "";
  display: inline-block;
  width: 5px;
}
</style>