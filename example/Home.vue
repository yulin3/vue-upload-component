<style>
.home {
  position: relative;
}

.file-uploads {
  font-size: 18px;
  padding: .6em;
  font-weight: bold;
  border: 1px solid #888;
  background: #f3f3f3
}


.drop-active {
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  position: absolute;
  opacity: .4;
  background: #000;
}
button {
  padding: .6em;
}

table {
  margin-bottom: 2em
}
table th,table td {
  padding: .4em;
  border: 1px solid #ddd
}

</style>
<template>
  <main class="home">
    <div id="lists">
      <table>
        <thead>
          <tr>
            <th>Index</th>
            <th>ID</th>
            <th>Image</th>
            <th>Name</th>
            <th>Size</th>
            <th>Progress</th>
            <th>Speed</th>
            <th>Active</th>
            <th>Error</th>
            <th>Success</th>
            <th>Abort</th>
            <th>customError</th>
            <th>Delete</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(file, index) in files" :key="file.id">
              <td>{{index}}</td>
              <td>{{file.id}}</td>
              <td v-if="file.type.substr(0, 6) == 'image/' && file.blob">
                <img :src="file.blob" width="50" height="auto" />
              </td>
              <td v-else></td>
              <td>{{file.name}}</td>
              <td>{{file.size | formatSize}}</td>
              <td>{{file.progress}}</td>
              <td>{{file.speed | formatSize}}</td>
              <td>{{file.active}}</td>
              <td>{{file.error}}</td>
              <td>{{file.success}}</td>
              <td><button type="button" @click.prevent="abort(file)">Abort</button></td>
              <td><button type="button" @click.prevent="customError(file)">custom error</button></td>
              <td><button type="button" @click.prevent="remove(file)">x</button></td>
          </tr>
        </tbody>
      </table>
    </div>
    <div id="options">
      <table>
        <tbody>
          <tr>
            <td>
              <file-upload
                :post-action="postAction"
                :put-action="putAction"
                :extensions="extensions"
                :accept="accept"
                :multiple="multiple"
                :directory="directory"
                :size="size || 0"
                :thread="thread < 1 ? 1 : (thread > 5 ? 5 : thread)"
                :headers="headers"
                :data="data"
                :drop="drop"
                :drop-directory="dropDirectory"
                v-model="files"
                @input-filter="inputFilter"
                @input-file="inputFile"
                ref="upload">
                Add upload files
              </file-upload>
            </td>
            <td>
              <button @click.prevent="addDirectory">Add upload directory</button>
              <br/>
              <span v-show="$refs.upload && !$refs.upload.features.directory">Your browser does not support</span>
            </td>
            <td>
              <button v-show="!$refs.upload || !$refs.upload.active" @click.prevent="$refs.upload.active = true" type="button">Start upload</button>
              <button v-show="$refs.upload && $refs.upload.active" @click.prevent="$refs.upload.active = false" type="button">Stop upload</button>
            </td>
            <td>
              <label>
                  Auto start: <input type="checkbox" id="checkbox" v-model="auto">
              </label>
            </td>
            <td>
              <label>
                Accept: <input type="text" v-model="accept">
              </label>
            </td>
            <td>
              <label>
                Extensions: <input type="text" v-model="extensions">
              </label>
            </td>
            <td>
              <label>
                Drop: <input type="checkbox" id="checkbox" v-model="drop">
              </label>
            </td>
            <td>
              <label>
                Max file size: <input type="text" v-model.number="size">
              </label>
            </td>
            <td>
              <label>
                Multiple: <input type="checkbox" id="checkbox" v-model="multiple">
              </label>
            </td>
            <td>
              <label>
                Thread: <input type="text" v-model.number="thread">
              </label>
            </td>
          </tr>
          <tr>
            <td>
              Auto start: {{auto}}
            </td>
            <td>
              Active: {{$refs.upload ? $refs.upload.active : false}}
            </td>
            <td>
              Uploaded: {{$refs.upload ? $refs.upload.uploaded : true}}
            </td>
            <td>
              Drop active: {{$refs.upload ? $refs.upload.dropActive : false}}
            </td>
            <td>
              <label :for="name">Click</label>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <h1>Allow to drag and drop</h1>
    <div v-show="$refs.upload && $refs.upload.dropActive" class="drop-active">
      Drop ing
    </div>
    <br/>
    <br/>
    <button type="button" @click.prevent="files = []">Test overwrite files</button>
  </main>
</template>


<script>
import FileUpload from '../src'
export default {
  components: {
    FileUpload,
  },

  data() {
     return {
      files: [],
      accept: 'image/png,image/gif,image/jpeg,image/webp',
      size: 1024 * 1024 * 10,
      extensions: 'gif,jpg,jpeg,png,webp',
      // extensions: ['gif', 'jpg', 'jpeg','png', 'webp'],
      // extensions: /\.(gif|jpe?g|png|webp)$/i,


      multiple: true,
      directory: false,
      drop: true,
      dropDirectory: true,
      thread: 3,
      name: 'file',

      postAction: './post.php',
      putAction: './put.php',

      headers: {
        "X-Csrf-Token": "xxxx",
      },

      data: {
        "_csrf_token": "xxxxxx",
      },


      auto: false,


      // headers: {},
      // putAction: '',
      // postAction: 'http://upload.qiniu.com/',
      // accept: 'image/png,image/gif,image/jpeg,image/webp',
      // extensions: '',
      // size: 0,
    }
  },

  watch: {
    auto(auto) {
      if (auto && !this.$refs.upload.uploaded && !this.$refs.upload.active) {
        this.$refs.upload.active = true
      }
    }
  },

  methods: {

    // add Directory
    addDirectory() {
      if (!this.$refs.upload.features.directory) {
        return
      }

      let input = this.$refs.upload.$el.querySelector('input')
      input.directory = true
      input.webkitdirectory = true
      this.directory = true

      input.onclick = null
      input.click()
      input.onclick = (e) => {
        this.directory = false
        input.directory = false
        input.webkitdirectory = false
      }
    },


    //  event Filter
    inputFilter(newFile, oldFile, prevent) {
      if (newFile && !oldFile) {
        // 过滤系疼文件 or 隐藏文件
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
          return prevent()
        }

        // 过滤 php html js 文件
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
          return prevent()
        }

        // 创建 blob 字段
        newFile.blob = ''
        var URL = window.URL || window.webkitURL
        if (URL && URL.createObjectURL) {
          newFile.blob = URL.createObjectURL(newFile.file)
        }
      }
    },


    // add, update, remove File Event
    inputFile(newFile, oldFile) {
      if (newFile && oldFile) {
        // console.log('update', newFile, oldFile)

        if (newFile.active && !oldFile.active) {
          // this.beforeSend(newFile)

          // min size
          if (newFile.size >= 0 && newFile.size < 100 * 1024) {
            // newFile = this.$refs.upload.update(newFile, {error: 'size'})
          }
        }

        if (newFile.progress != oldFile.progress) {
          // this.progress(newFile)
          // console.log('progress', newFile.progress)
        }

        if (newFile.error && !oldFile.error) {
          // this.error(newFile)
          // console.log('error', newFile.error, newFile.response)
        }

        if (newFile.success && !oldFile.success) {
          // this.success(newFile)
          // console.log('success', newFile.response)
        }
      }


      if (!newFile && oldFile) {
        // this.remove(oldFile)
        // console.log('remove', oldFile)
      }


      // 自动开始
      if (newFile && !oldFile && this.auto && !this.$refs.upload.active) {
        this.$refs.upload.active = true
      }
    },

    abort(file) {
      this.$refs.upload.update(file, {active: false})
      // or
      // this.$refs.upload.update(file, {error: 'abort'})
    },

    customError(file) {
      this.$refs.upload.update(file, {error: 'custom'})
    },
    remove(file) {
      this.$refs.upload.remove(file)
    },

  },
}



</script>
