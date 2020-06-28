<template>
   <div>
      <h1>{{ msg }}</h1>
      <p>Please select a JSON or CSV file to convert.</p>
      <div class="converter">
         <label for="button" id="labelButton">{{ label }}</label>
         <input type="file" id="button" accept="application/JSON, .csv" @change="loadTextFromFile" />
         <br />
         <br />
      </div>
      <div class="link">
         <a id="lnkDwnldLnk" hidden="hidden">Download</a>
      </div>
   </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Converter extends Vue {
   @Prop() private msg!: string;
   label = 'Select a file';

   loadTextFromFile(event: Event) {   
      const target = event.target as HTMLInputElement;
      const file: File = (target.files as FileList)[0];
      const link = document.querySelector('#lnkDwnldLnk');

      if(!file) {
         this.label = 'Select a file';
         link && link.setAttribute('hidden', 'true');
         return;
      }

      const filename = file.name.split('.');
      
      if (filename[1].includes('json')) {
         this.convertToCSV(file, link, filename);
      } else if (filename[1].includes('csv')) {
         this.convertToJSON(file, link, filename);
      } else {
         alert('Invalid file');
      }
   }

   convertToCSV(file: File, link: Element | null, filename: string[]) {    
      if (file) {
         return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.onerror = reject;
            reader.onload = () => {
               const loadedFile = String(reader.result);              
               const validJSON = this.verifyJSON(loadedFile);
               
               if(!validJSON) {
                  this.label = 'Select a file';
                  link && link.setAttribute('hidden', 'true');
                  alert('Invalid JSON File');
                  return;
                }

               this.label = filename[0] + '.json';
               const jsonFile = JSON.parse(loadedFile);    

               if (!jsonFile.length) {
                  const list: string[] = [];
                  list.unshift();
                  list.push(jsonFile);
                  resolve(list);
               } else {
                  resolve(jsonFile);
               }
            }
            reader.readAsText(file);
         }).then((json: any) => {
            if (link) {
               link.removeAttribute('hidden');
            }

            const items = json;
            const replacer = (key: string, value: string) => (value === null ? '' : value);
            const header = items ? Object.keys(items[0]) : [];

            let csv = items
               ? items.map((row: Record<string, string>) =>
                    header.map((fieldName) => JSON.stringify(row[fieldName], replacer)).join(',')
                 )
               : [];
            csv.unshift(header.join(','));
            csv = csv.join('\r\n');

            const blob = new Blob([csv], { type: 'text/csv' });
            const csvUrl = window.webkitURL.createObjectURL(blob);
            const filenameConverted = `${filename[0]}.csv`;

            if (link) {
               link.setAttribute('download', filenameConverted);
               link.setAttribute('href', csvUrl);
            }
         });
      } else {
         if (link) {
            link.setAttribute('hidden', 'true');
         }
      }
   }

   verifyJSON(json: string) {
      let data = '';
      try {
         data = JSON.parse(json);
         if (data && typeof data === 'object' && data !== null) {
            return true;
         }
      } catch(ex) {
         return false;
      }
   }

   convertToJSON(file: File, link: Element | null, filename: string[]) {
      if (file) {
         this.label = filename[0] + '.csv';
         return new Promise<string>((resolve, reject) => {
            const reader = new FileReader();
            reader.onerror = reject;
            reader.onload = () => {
               const loadedFile = String(reader.result);

               resolve(loadedFile);
            };
            reader.readAsText(file);
         }).then((csv) => { 
            try{
               let item = csv.trim(); 
               item = item.replace(/"/g, '');

               if (link) {
                  link.removeAttribute('hidden');
               }
               const lines = item.split('\n');
               const result = [];
               const headers = lines[0].split(',');

               for (let i = 1; i < lines.length; i++) {
                  const obj: Record<string, string> = {};
                  const currentline = lines[i].split(',');

                  for (let j = 0; j < headers.length; j++) {
                     obj[headers[j].trim()] = currentline[j].trim();
                  }

                  result.push(obj);
               }

               const finalJSON = JSON.stringify(result).replace('\\r', '');

               const blob = new Blob([finalJSON], { type: 'text/json' });
               const csvUrl = window.webkitURL.createObjectURL(blob);
               const filenameConverted = `${filename[0]}.json`;

               if (link) {
                  link.setAttribute('download', filenameConverted);
                  link.setAttribute('href', csvUrl);
               }
            }catch(ex) {
               this.label = 'Select a file';
               link && link.setAttribute('hidden', 'true');
               alert('Invalid CSV File');
               return;
            }
         });
      } else {
         if (link) {
            link.setAttribute('hidden', 'true');
         }
      }
   }
}
</script>

<style scoped>
h1 {
   text-align: center;
   margin-top: 26px;
}

p {
   text-align: center;
   margin-top: 26px;
}

.converter {
   display: flex;
   justify-content: center;
}

input[type='file'] {
   display: none;
}

label {
   background-color: #e02041;
   border-radius: 5px;
   color: #fff;
   cursor: pointer;
   margin: 10px;
   padding: 8px 20px;
   transition: filter 0.2s;
}

label:hover {
   filter: brightness(90%);
}

.link {
   display: flex;
   justify-content: center;
   font-size: 18px;
   margin-top: 6px;
}

a {
   text-decoration: none;
   color: #e02041;
}
</style>
