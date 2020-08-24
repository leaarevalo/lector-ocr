<template>
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta
        name="viewport"
        content="width=device-width, initial-scale=1, shrink-to-fit=no"
      />
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        <div>
          <b-jumbotron header="Lector OCR" lead="">
            <p>
              Cargar una imagen desde assets dentro de la carpeta del proyecto
              luego presionar 'Comprobar'
            </p>
            <div class="form">
              <div class="input">
                <b-form-file
                  v-model="file.imagen"
                  :state="Boolean(file.imagen)"
                  @change="cargarImagen"
                  placeholder="Choose a file or drop it here..."
                  drop-placeholder="Drop file here..."
                >
                </b-form-file>
              </div>
              <div class="button">
                <b-button @click="postImage">
                  Comprobar
                </b-button>
              </div>
            </div>
            <div class="card">
              <b-card sub-title="A pagar:" class="text-center">
                <span v-if="typeof keywords[0] !== 'undefined'">{{
                  keywords[0]
                }}</span>
              </b-card>
            </div>
            <div class="card">
              <b-card sub-title="Texto encontrado: " class="text-center">
                <div
                  v-if="typeof fileResponse.data !== 'undefined'"
                  class="textoMuestra"
                >
                  {{ fileResponse.data.ParsedResults[0].ParsedText }}
                </div>
              </b-card>
            </div>
            <div class="image">
              <b-card sub-title="Imagen:" class="text-center">
                <div class="bordeImagen">
                  <div class="texto"></div>

                  <div class="figura">
                    <figure>
                      <img
                        width="200px"
                        height="200px"
                        :src="imagenMiniatura"
                        alt=""
                      />
                    </figure>
                  </div>
                </div>
              </b-card>
            </div>
          </b-jumbotron>
        </div>
      </div>
    </body>
  </html>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      file: {
        imagen: "",
      },
      fileResponse: {},
      keywords: [],
      imagenMiniatura: "",
    };
  },
  methods: {
    postImage() {
      console.log();

      if (
        this.file.imagen.name.split(".")[1] === "png" ||
        this.file.imagen.name.split(".")[1] === "jpg"
      ) {
        var data = new FormData();
        data.append("language", "eng");
        data.append("isOverlayRequired", "true");
        data.append("file", this.file.imagen);
        data.append("iscreatesearchablepdf", "false");
        data.append("issearchablepdfhidetextlayer", "false");

        console.log(this.file.imagen);
        var config = {
          method: "post",
          url: "https://api.ocr.space/parse/image",
          headers: {
            apikey: "7df0e1289088957",
            ...data,
          },
          data: data,
        };
        axios(config)
          .then((res) => {
            this.fileResponse = res;
            console.log(res);
            this.searchWords(res.data.ParsedResults[0].TextOverlay.Lines);
            //console.log(JSON.stringify(res.data));
          })
          .catch((err) => {
            console.log(err);
          });
      } else {
        alert("Formato de archivo no valido");
      }
    },
    searchWords(words) {
      words.forEach((e) => {
        if (e.LineText.includes("TOTAL")) {
          this.keywords[0] = e.LineText;
        }
      });
    },
    cargarImagen(e) {
      this.fileResponse = {};
      this.keywords = [];
      let file = e.target.files[0];
      let reader = new FileReader();
      reader.onload = (e) => {
        this.imagenMiniatura = e.target.result;
      };
      reader.readAsDataURL(file);
    },
  },
  computed: {
    imagen() {
      return this.imagenMiniatura;
    },
  },
};

//apiKey = 7df0e1289088957
</script>

<style>
#app {
  display: block;
  width: 100%;
  height: 400px;
}
.form {
  display: flex;
  flex-direction: column;
  width: 100%;
  justify-content: center;
  align-items: center;
  margin: 20px;
}

.card {
  width: 500px;
  height: auto;
  margin-left: auto;
  margin-right: auto;
}
.input {
  margin: 20px;
}
</style>
