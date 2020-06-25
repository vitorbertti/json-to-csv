<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>Please select a JSON or CSV file to convert.</p>
    <div class="converter">
      <input type="file" accept="application/JSON, .csv" @change="loadTextFromFile" />
      <br />
      <br />
      <a id="lnkDwnldLnk" hidden="hidden">Download</a>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class Converter extends Vue {
  @Prop() private msg!: string;

  loadTextFromFile(event: any) {
    const file = event.target.files[0];
    const link = document.querySelector("#lnkDwnldLnk");
    const filename = file.name.split(".");
    if (filename[1].includes("json")) {
      this.convertToCSV(file, link, filename);
    } else if (filename[1].includes("csv")) {
      this.convertToJSON(file, link, filename);
    } else {
      console.log("nenhum");
    }
  }

  convertToCSV(file: any, link: any, filename: any) {
    if (file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onerror = reject;
        reader.onload = () => {
          const loadedFile = String(reader.result);
          const jsonFile = JSON.parse(loadedFile);
          resolve(jsonFile);
        };

        reader.readAsText(file);
      }).then(json => {
        if (link) {
          link.removeAttribute("hidden");
        }

        const items: any = json;
        const replacer = (key: string, value: string) =>
          value === null ? "" : value;
        const header = items ? Object.keys(items[0]) : [];

        let csv = items
          ? items.map((row: any) =>
              header
                .map(fieldName => JSON.stringify(row[fieldName], replacer))
                .join(",")
            )
          : [];
        csv.unshift(header.join(","));
        csv = csv.join("\r\n");

        const blob = new Blob([csv], { type: "text/csv" });
        const csvUrl = window.webkitURL.createObjectURL(blob);
        const filenameConverted = `${filename[0]}.csv`;

        if (link) {
          link.setAttribute("download", filenameConverted);
          link.setAttribute("href", csvUrl);
        }
      });
    } else {
      if (link) {
        link.setAttribute("hidden", "true");
      }
    }
  }

  convertToJSON(file: any, link: any, filename: any) {
    if (file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onerror = reject;
        reader.onload = () => {
          const loadedFile = String(reader.result);

          resolve(loadedFile);
        };
        reader.readAsText(file);
      }).then((csv: any) => {
        let item = csv.trim();
        item = item.replace(/"/g, "");

        if (link) {
          link.removeAttribute("hidden");
        }
        const lines = item.split("\n");
        const result = [];
        const headers = lines[0].split(",");

        for (let i = 1; i < lines.length; i++) {
          const obj: any = {};
          const currentline = lines[i].split(",");

          for (let j = 0; j < headers.length; j++) {
            obj[headers[j]] = currentline[j];
          }

          result.push(obj);
        }

        const finalJSON = JSON.stringify(result).replace("\\r", "");

        console.log(finalJSON);

        const blob = new Blob([finalJSON], { type: "text/json" });
        const csvUrl = window.webkitURL.createObjectURL(blob);
        const filenameConverted = `${filename[0]}.json`;

        if (link) {
          link.setAttribute("download", filenameConverted);
          link.setAttribute("href", csvUrl);
        }
      });
    } else {
      if (link) {
        link.setAttribute("hidden", "true");
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
</style>
