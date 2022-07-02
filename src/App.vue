<script>
const left = `\
type: map
fields:
  email:
    type: email
  password:
    type: string
    regex: foobar
  status:
    type: enum
    values: ['pending', 'active', 'deleted']
    default: 'pending'
  name:
    type: string
    length: [2, 40]
    option: true
  hobbies:
    type: array
    value:
      type: string
`

const right = `\
{
  "email": "john.doe@gmail.com",
  "password": "1234abcd",
  "status": "active",
  "hobbies": ["Soccer", "Badminton", "Movies"]
}
`

import { StreamLanguage } from "@codemirror/language"
import { json } from '@codemirror/lang-json'
import { yaml } from "@codemirror/legacy-modes/mode/yaml"
import { Codemirror } from 'vue-codemirror'
import Warnings from '@/Warnings.vue'
import Errors from '@/Errors.vue'

export default {
  name: 'App',
  components: {
    Codemirror,
    Warnings,
    Errors
  },
  setup() {
    const yamlExtensions = [StreamLanguage.define(yaml)]
    const jsonExtensions = [json()]

    return {
      yamlExtensions,
      jsonExtensions
    }
  },
  data() {
    return {
      specs: {
        value: left,
        warnings: [],
        errors: [],
        timer: null
      },
      document: {
        value: right,
        warnings: [],
        errors: [],
        timer: null
      }
    }
  },
  methods: {
    parseSpecs() {
      console.log("parse specs...")

      const body = {
        specs: this.specs.value
      }

      this.axios.post('/validator/parse-specs', body)
        .then((response) => {
          this.specs.warnings = response.data.warnings
          this.specs.errors = response.data.errors

          console.log("[POST] /validator/parse-specs -- OK")
        })
        .catch((error) => {
          console.log("[POST] /validator/parse-specs -- Error")
        })
    },
    parseDocument() {
      console.log("parse document...")

      const body = {
        document: this.document.value
      }

      this.axios.post('/validator/parse-document', body)
        .then((response) => {
          this.document.warnings = response.data.warnings
          this.document.errors = response.data.errors

          console.log("[POST] /validator/parse-document -- OK")
        })
        .catch((error) => {
          console.log("[POST] /validator/parse-document -- Error")
        })
    },
    validateDocument() {
      console.log("validate document...")
    }
  },
  watch: {
    'specs.value'(value) {
      clearTimeout(this.specs.timer)
      this.specs.timer = setTimeout(this.parseSpecs, 3000);
    },
    'document.value'(value) {
      clearTimeout(this.document.timer)
      this.document.timer = setTimeout(this.parseDocument, 3000);
    }
  }
}
</script>

<template>
  <div class="_height:100% _display:flex _flex-direction:column">
    <i-nav class="_display:flex _justify-content:space-between">
      <i-nav-item href="/">
        <img height="40" src="@/logo.svg">
        <span
          class="_margin-left:1 _font-size:lg"
          style="font-family: Xolonium;"
        >Document Validation Playground</span>
      </i-nav-item>
      <i-nav>
        <i-nav-item
          href="https://www.byteplug.io/standards/document-validator"
          class="_margin-right:1"
        >Website</i-nav-item>
      </i-nav>
    </i-nav>
    <div class="_flex-grow:1">
      <i-container class="_height:100% _display:flex _flex-direction:column">
        <div class="_width:75%">
          <h3>JSON Validation Playground</h3>
          <div class="_margin-bottom:3">
  Toy around with Byteplug JSON Validator Standard. On the left, enter the YAML
  specifications of a JSON document, and on the right see how it validates (or
  invalidates) them.
          </div>
        </div>
        <i-row class="_flex-grow:1">
          <i-column
            xs="6"
            class="_display:flex _flex-direction:column"
          >
            <div class="_display:flex _justify-content:space-between">
              <div class="_font-weight:bold _font-size:lg _margin-bottom:1">
                YAML Specs
              </div>
              <div>
                <warnings :warnings="specs.warnings"/>
                <errors :errors="specs.errors"/>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="specs.value"
                :style="{ height: '100%' }"
                :autofocus="true"
                :indent-with-tab="true"
                :tab-size="2"
                :extensions="yamlExtensions"
              />
            </div>
          </i-column>
          <i-column
            xs="6"
            class="_display:flex _flex-direction:column"
          >
            <div class="_display:flex _justify-content:space-between">
              <div class="_font-weight:bold _font-size:lg _margin-bottom:1">
                JSON Document
              </div>
              <div>
                <warnings :warnings="document.warnings"/>
                <errors :errors="document.errors"/>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="document.value"
                :style="{ height: '100%' }"
                :autofocus="true"
                :indent-with-tab="true"
                :tab-size="2"
                :extensions="jsonExtensions"
              />
            </div>
          </i-column>
        </i-row>
        <div class="_display:flex _justify-content:space-between _margin-y:1">
          <i-dropdown size="sm">
            <i-button>Examples</i-button>
            <template #body>
              <i-dropdown-item>Action</i-dropdown-item>
              <i-dropdown-item>Another action</i-dropdown-item>
              <i-dropdown-item disabled>Disabled action</i-dropdown-item>
              <i-dropdown-divider />
              <i-dropdown-item>Separated item</i-dropdown-item>
            </template>
          </i-dropdown>
          <div class="_display:flex">
            <i-toggle v-model="toggled">Enable dynamic validation</i-toggle>
            <i-button class="_margin-left:1" color="primary">Validate</i-button>
          </div>
        </div>
      </i-container>
    </div>
    <div class="_display:flex _justify-content:center">
      <div class="_flex-grow:1">
        <svg height="60" width="100%">
          <line x1="6%" y1="48px" x2="94%" y2="48px" style="stroke:#d60000;stroke-width:1" />
        </svg>
      </div>
      <div class="_margin-bottom:2">
        <img src="@/logo.svg" height="60" alt="">
      </div>
      <div class="_flex-grow:1">
        <svg height="60" width="100%">
          <line x1="6%" y1="48px" x2="94%" y2="48px" style="stroke:#d60000;stroke-width:1" />
        </svg>
      </div>
    </div>
  </div>
</template>

<style>
html, body, #app {
  height: 100%;
}

@font-face {
  font-family: "Xolonium";
  src: local("Xolonium"), url(./fonts/Xolonium/Xolonium-Regular.ttf) format("truetype");
}
</style>