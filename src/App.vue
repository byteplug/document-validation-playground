<script>
import flagSpecs       from '@/examples/flag/specs.yml?raw'
import flagDocument    from '@/examples/flag/document.json?raw'
import integerSpecs    from '@/examples/integer/specs.yml?raw'
import integerDocument from '@/examples/integer/document.json?raw'
import decimalSpecs    from '@/examples/decimal/specs.yml?raw'
import decimalDocument from '@/examples/decimal/document.json?raw'
import stringSpecs     from '@/examples/string/specs.yml?raw'
import stringDocument  from '@/examples/string/document.json?raw'
import enumSpecs       from '@/examples/enum/specs.yml?raw'
import enumDocument    from '@/examples/enum/document.json?raw'

import listSpecs     from '@/examples/list/specs.yml?raw'
import listDocument  from '@/examples/list/document.json?raw'
import tupleSpecs    from '@/examples/tuple/specs.yml?raw'
import tupleDocument from '@/examples/tuple/document.json?raw'
import mapSpecs      from '@/examples/map/specs.yml?raw'
import mapDocument   from '@/examples/map/document.json?raw'

import userSpecs         from '@/examples/user/specs.yml?raw'
import userDocument      from '@/examples/user/document.json?raw'
import fancyByteSpecs    from '@/examples/fancy-byte/specs.yml?raw'
import fancyByteDocument from '@/examples/fancy-byte/document.json?raw'

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
        value: userSpecs,
        warnings: [],
        errors: [],
        timer: null
      },
      document: {
        value: userDocument,
        warnings: [],
        errors: [],
        timer: null
      },
      dynamicValidation: false,
      examples: {
        'flag': {
          name: "Flag",
          specs: flagSpecs,
          document: flagDocument
        },
        'integer': {
          name: "Integer",
          specs: integerSpecs,
          document: integerDocument
        },
        'decimal': {
          name: "Decimal",
          specs: decimalSpecs,
          document: decimalDocument
        },
        'string': {
          name: "String",
          specs: stringSpecs,
          document: stringDocument
        },
        'enum': {
          name: "Enum",
          specs: enumSpecs,
          document: enumDocument
        },
        'list': {
          name: "List",
          specs: listSpecs,
          document: listDocument
        },
        'tuple': {
          name: "Tuple",
          specs: tupleSpecs,
          document: tupleDocument
        },
        'map': {
          name: "Map",
          specs: mapSpecs,
          document: mapDocument
        },
        'user': {
          name: "User",
          specs: userSpecs,
          document: userDocument
        },
        'fancy-byte': {
          name: "Fancy Byte",
          specs: fancyByteSpecs,
          document: fancyByteDocument
        }
      },
      fundamentalExamples: ['flag', 'integer', 'decimal', 'string', 'enum'],
      compositeExamples: ['list', 'tuple', 'map'],
      concreteExamples: ['user', 'fancy-byte'],
    }
  },
  methods: {
    setExample(example) {
      this.specs.value = this.examples[example].specs
      this.document.value = this.examples[example].document
    },
    validateSpecs() {
      console.log("validate specs...")

      const body = {
        specs: this.specs.value
      }

      this.axios.post('/validator/validate-specs', body)
        .then((response) => {
          // TODO; Handle when response is 'invalid-yaml-specs'
          // if (response == "invalid-yaml-specs")

          this.specs.warnings = response.data.warnings
          this.specs.errors = response.data.errors

          console.log("[POST] /validator/validate-specs -- OK")
        })
        .catch((error) => {
          console.log("[POST] /validator/validate-specs -- Error")
        })

      this.specs.timer = null
    },
    validateDocument() {
      console.log("validate document...")

      if (!this.isSpecsValid) {
        console.log("specs is not valid; abort early")
        return
      }

      const body = {
        specs: this.specs.value,
        document: this.document.value
      }

      this.axios.post('/validator/validate-document', body)
        .then((response) => {
          // TODO; Handle when response is 'invalid-yaml-specs' and 'invalid-json-document'
          // if (response == "invalid-yaml-specs")
          // if (response == "invalid-json-document")

          this.document.warnings = response.data.warnings
          this.document.errors = response.data.errors

          console.log("[POST] /validator/validate-document -- OK")
        })
        .catch((error) => {
          console.log("[POST] /validator/validate-document -- Error")
        })

      this.document.timer = null
    }
  },
  computed: {
    isSpecsValid: function() {
      // We want to disable the validate button when the specs is not valid (we
      // cannot validate a document with an invalid specs).
      return this.specs.errors.length == 0 && this.specs.timer == null
    }
  },
  watch: {
    'specs.value'(value) {
      clearTimeout(this.specs.timer)
      this.specs.timer = setTimeout(this.validateSpecs, 3000);
    },
    'document.value'(value) {
      if (this.dynamicValidation) {
        clearTimeout(this.document.timer)
        this.document.timer = setTimeout(this.validateDocument, 3000);
      }
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
          <h4>JSON Document Validator</h4>
          <div class="_margin-bottom:3">
Toy around with the Document Validator standard. On the left you enter the YAML
specifications of your JSON document, and on the right see how it validates (or
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
              <i-dropdown-item
                v-for="example in fundamentalExamples"
                :key="example"
                @click="setExample(example)"
              >{{ examples[example].name }}
              </i-dropdown-item>
              <i-dropdown-divider />
              <i-dropdown-item
                v-for="example in compositeExamples"
                :key="example"
                @click="setExample(example)"
              >{{ examples[example].name }}
              </i-dropdown-item>
              <i-dropdown-divider />
              <i-dropdown-item
                v-for="example in concreteExamples"
                :key="example"
                @click="setExample(example)"
              >{{ examples[example].name }}
              </i-dropdown-item>
            </template>
          </i-dropdown>
          <div class="_display:flex">
            <i-toggle v-model="dynamicValidation">Enable dynamic validation</i-toggle>
            <i-button class="_margin-left:1" color="primary" @click="validateDocument">Validate</i-button>
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
