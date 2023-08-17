<script>
import YAML from 'yaml'
import { validateFormat, payloadToObject } from '@byteplug/payload'
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

      format: {
        text: "",
        value: undefined,
        errors: [],
        warnings: [],
        timer: null
      },
      payload: {
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
      this.format.text = this.metadata.types[type_].example.format
      this.payload.text = this.metadata.types[type_].example.validPayload
    },
    validateFormat() {
      try {
        var object = YAML.parse(this.format.text)
      }
      catch (err) {
        console.log("Failed to parse the YAML string.")

        this.format.value = undefined
        this.format.errors.length = 0
        this.format.errors.push("Syntax is invalid.")
        this.format.errors.warnings = 0
        this.format.timer = null

        return
      }

      this.format.value = object

      var errors = []
      var warnings = []
      validateFormat(object, errors, warnings)

      this.format.errors = formatPaths(errors)
      this.format.warnings = formatPaths(warnings)

      this.format.timer = null
    },
    validateDocument() {
      if (this.format.value === undefined)
        return

      try {
        var payload = JSON.parse(this.payload.text)
      }
      catch (err) {
        console.log("Failed to parse the JSON string.")

        this.payload.value = undefined
        this.payload.errors.length = 0
        this.payload.errors.push("Syntax is invalid.")
        this.payload.errors.warnings = 0
        this.payload.timer = null

        return
      }

      this.payload.value = payload

      var errors = []
      var warnings = []
      // payloadToObject(payload, this.format.value, errors, warnings)
      payloadToObject(this.payload.text, this.format.value, errors, warnings)

      this.payload.errors = formatPaths(errors)
      this.payload.warnings = formatPaths(warnings)

      this.payload.timer = null
    }
  },
  computed: {
    status() {
      if (this.format.timer !== null) {
        return "pending"
      }
      else if (this.format.errors.length > 0 || this.format.warnings.length > 0) {
        return "invalid"
      }
      else {
        return "valid"
      }
    }
  },
  watch: {
    'format.text'(value) {
      clearTimeout(this.format.timer)
      this.format.timer = setTimeout(this.validateFormat, 50)

      this.payload.timer = 42
    },
    'payload.text'(value) {
      this.payload.timer = 42

      if (this.dynamicValidation) {
        clearTimeout(this.payload.timer)
        this.payload.timer = setTimeout(this.validatePayload, 50)
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
        href="https://www.byteplug.io/standards/payload"
        class="_margin-right:1"
      >Official Website</i-nav-item>
    </i-nav>

    </i-container>
    <div class="_flex-grow:1">
      <i-container class="_height:100% _display:flex _flex-direction:column">
        <div class="_width:75%">
          <h4>Payload Validator</h4>
          <div class="_margin-bottom:3">
Toy around with the Payload standard. On the left you enter the YAML-based
format (or specification) of your JSON payload, and on the right see how it
validates (or invalidates) them.
          </div>
        </div>
        <i-row class="_flex-grow:1">
          <i-column
            xs="6"
            class="_display:flex _flex-direction:column"
          >
            <div class="_display:flex _justify-content:space-between">
              <div class="_font-weight:bold _font-size:lg _margin-bottom:1">
                Format (YAML)
              </div>
              <div v-if="status === 'invalid'">
                <warnings :warnings="format.warnings"/>
                <errors :errors="format.errors"/>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="format.text"
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
                Payload (
                  JSON)
              </div>
              <div v-if="status === 'valid'">
                <div v-if="payload.timer != null"></div>
                <div v-else-if="payload.errors.length > 0 || payload.warnings.length > 0">
                  <warnings :warnings="payload.warnings"/>
                  <errors :errors="payload.errors"/>
                </div>
                <div v-else>
                  <i-icon name="ink-check" class="_color:success" />
                </div>
              </div>
            </div>
            <div class="_background:white _flex-grow:1">
              <codemirror
                v-model="payload.text"
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
              @click="validatePayload"
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
