<script>
import YAML from 'yaml'
import { validateSpecs, documentToObject } from '@byteplug/document'
import { StreamLanguage } from "@codemirror/language"
import { json } from '@codemirror/lang-json'
import { yaml } from "@codemirror/legacy-modes/mode/yaml"
import { Codemirror } from 'vue-codemirror'

import metadata from '@/metadata.yml?raw'
import Warnings from '@/Warnings.vue'
import Errors from '@/Errors.vue'

function formatPaths(values) {
  return values.map(value => {
    var text = "root"
    if (value.path.length > 0)
      text = value.path.join('.')

    return text + ": " + value.message
  })
}

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
      metadata: YAML.parse(metadata),

      specs: {
        text: "",
        value: undefined,
        errors: [],
        warnings: [],
        timer: null
      },
      document: {
        text: "",
        value: undefined,
        errors: [],
        warnings: [],
        timer: 42
      },
      dynamicValidation: false
    }
  },
  methods: {
    setExample(type_) {
      this.specs.text = this.metadata.types[type_].example.specs
      this.document.text = this.metadata.types[type_].example.validDocument
    },
    validateSpecs() {
      try {
        var object = YAML.parse(this.specs.text)
      }
      catch (err) {
        console.log("Failed to parse the YAML string.")

        this.specs.value = undefined
        this.specs.errors.length = 0
        this.specs.errors.push("Syntax is invalid.")
        this.specs.errors.warnings = 0
        this.specs.timer = null

        return
      }

      this.specs.value = object

      var errors = []
      var warnings = []
      validateSpecs(object, errors, warnings)

      this.specs.errors = formatPaths(errors)
      this.specs.warnings = formatPaths(warnings)

      this.specs.timer = null
    },
    validateDocument() {
      if (this.specs.value === undefined)
        return

      try {
        var document = JSON.parse(this.document.text)
      }
      catch (err) {
        console.log("Failed to parse the JSON string.")

        this.document.value = undefined
        this.document.errors.length = 0
        this.document.errors.push("Syntax is invalid.")
        this.document.errors.warnings = 0
        this.document.timer = null

        return
      }

      this.document.value = document

      var errors = []
      var warnings = []
      // documentToObject(document, this.specs.value, errors, warnings)
      documentToObject(this.document.text, this.specs.value, errors, warnings)

      this.document.errors = formatPaths(errors)
      this.document.warnings = formatPaths(warnings)

      this.document.timer = null
    }
  },
  computed: {
    status() {
      if (this.specs.timer !== null) {
        return "pending"
      }
      else if (this.specs.errors.length > 0 || this.specs.warnings.length > 0) {
        return "invalid"
      }
      else {
        return "valid"
      }
    }
  },
  watch: {
    'specs.text'(value) {
      clearTimeout(this.specs.timer)
      this.specs.timer = setTimeout(this.validateSpecs, 50)

      this.document.timer = 42
    },
    'document.text'(value) {
      if (this.dynamicValidation) {
        clearTimeout(this.document.timer)
        this.document.timer = setTimeout(this.validateDocument, 50)
      }
    }
  },
  mounted() {
    this.setExample('number')
  }
}
</script>

<template>
  <div class="_height:100% _display:flex _flex-direction:column">
    <i-container>
    <i-nav class="_display:flex _justify-content:end">
      <i-nav-item
        href="https://www.byteplug.io/standards/document-validator"
        class="_margin-right:1"
      >Official Website</i-nav-item>
    </i-nav>

    </i-container>
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
              <div v-if="status === 'invalid'">
                <warnings :warnings="specs.warnings"/>
                <errors :errors="specs.errors"/>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="specs.text"
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
              <div v-if="status === 'valid'">
                <div v-if="document.timer != null"></div>
                <div v-else-if="document.errors.length > 0 || document.warnings.length > 0">
                  <warnings :warnings="document.warnings"/>
                  <errors :errors="document.errors"/>
                </div>
                <div v-else>
                  <i-icon name="ink-check" class="_color:success" />
                </div>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="document.text"
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
                v-for="type_ in metadata.categories.fundamental.types"
                :key="type_"
                @click="setExample(type_)"
              >{{ metadata.types[type_].name }}
              </i-dropdown-item>
              <i-dropdown-divider />
              <i-dropdown-item
                v-for="type_ in metadata.categories.composite.types"
                :key="type_"
                @click="setExample(type_)"
              >{{ metadata.types[type_].name }}
              </i-dropdown-item>
              <i-dropdown-divider />
              <i-dropdown-item
                v-for="type_ in metadata.categories.extra.types"
                :key="type_"
                @click="setExample(type_)"
              >{{ metadata.types[type_].name }}
              </i-dropdown-item>
            </template>
          </i-dropdown>
          <div class="_display:flex">
            <i-toggle v-model="dynamicValidation">Enable dynamic validation</i-toggle>
            <i-button
              class="_margin-left:1"
              color="primary"
              @click="validateDocument"
              :disabled="status !== 'valid'">Validate</i-button>
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
