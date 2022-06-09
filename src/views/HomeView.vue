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

export default {
  name: 'HomeView',
  data() {
    return {
      foo: left,
      bar: right
    }
  }
}
</script>

<template>
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
            0 <i-icon name="ink-warning" class="_color:warning" />
            0 <i-icon name="ink-danger" class="_color:danger" />
          </div>
        </div>
        <div class="_background:white _flex-grow:1 textarea-full-height">
          <i-textarea
            v-model="foo"
            placeholder="Type something.."
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
            0 <i-icon name="ink-warning" class="_color:warning" />
            0 <i-icon name="ink-danger" class="_color:danger" />
          </div>
        </div>
        <div class="_background:white _flex-grow:1 textarea-full-height">
          <i-textarea
            v-model="bar"
            placeholder="Type something.."
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
</template>

<style>
.textarea-full-height div {
  height:100%;
}

.textarea-full-height div div {
  height:100%;
}

.textarea-full-height div div textarea{
  height:100%;
  resize: none;
}
</style>