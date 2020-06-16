<template>
   <div class="hello">
      <h1>{{ msg }}</h1>
      <div>
         <input type="file" accept="application/JSON" @change="loadTextFromFile" />
         <br /><br />
         <a id="lnkDwnldLnk" hidden="hidden">Download</a>
      </div>
   </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Converter extends Vue {
   @Prop() private msg!: string;

   loadTextFromFile(event: any) {
      const file = event.target.files[0];
      const link = document.querySelector('#lnkDwnldLnk');
      const json: object[] = [{}];

      if (file) {
         const reader = new FileReader();
         reader.onload = (e: any) => {
            const loadedFile = String(reader.result);
            const jsonFile = JSON.parse(loadedFile);
            json.shift();

            jsonFile.map((obj: any) => {
               json.push(obj);
            });
         };
         reader.readAsText(file);

         const filename = file.name.split('.');
         if (link) {
            link.removeAttribute('hidden');
         }

         this.convertToCSV(json, filename[0], link as Element);
      } else {
         if (link) {
            link.setAttribute('hidden', 'true');
         }
      }
   }

   convertToCSV(json: any, filename: string, link: Element) {
      const items = json;
      const replacer = (key: string, value: string) => (value === null ? '' : value);
      const header = items ? Object.keys(items[0]) : [];

      let csv = items ? items.map((row: any) => header.map((fieldName) => JSON.stringify(row[fieldName], replacer)).join(',')) : [];
      csv.unshift(header.join(','));
      csv = csv.join('\r\n');

      const blob = new Blob([csv], { type: 'text/csv' });
      const csvUrl = window.webkitURL.createObjectURL(blob);
      const filenameConverted = `${filename}.csv`;

      if (link) {
         link.setAttribute('download', filenameConverted);
         link.setAttribute('href', csvUrl);
      }
   }
}
</script>

<style scoped>
h3 {
   margin: 40px 0 0;
}
ul {
   list-style-type: none;
   padding: 0;
}
li {
   display: inline-block;
   margin: 0 10px;
}
a {
   color: #42b983;
}
</style>
